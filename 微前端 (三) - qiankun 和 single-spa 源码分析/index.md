## 微前端 (三) - qiankun 和 single-spa 源码分析 
### 概览

 `qiankun` 是一个基于[single-spa](https://github.com/single-spa/single-spa)的微前端实现库。接下来我们将通过源码透析其实现方式及技巧。相信通过前面两篇文章，你也能更容易明白源码实现。

qiannkun 针对于`主应用`暴露了方法 `registerMicroApps` 和 `start` 以及 `loadMicroApp` 方法。这里的`主应用`就是上两篇里的`容器应用`，虽然个人更愿意使用`容器`术语，但是为了和文档统一，下面都将使用`主应用`指代`容器应用`。

对于`微应用`，需要其暴露约定的勾子方法：`bootstrap`、`mount`、`unmount` 以及 `update` 。前面三个接口方法是 `single-spa` 提供的，并且要求是必须实现的，在下面的源码中`微应用`将由变量 `app` 指代。

在主应用和微应用的交互模式上采用了`发布/订阅`的方式，`微应用`实现了统一的接口(mount、unmount...)，并订阅了主应用。所以我们将主要从主应用提供的接口来追溯其实现。

### 源码准备

因为 `qiankun` 是基于 `single-spa` 实现的，所以你需要同时从[qiankun 仓库](https://github.com/umijs/qiankun)和[single-spa 仓库](https://github.com/single-spa/single-spa)克隆两份 master 分支的源码。

### 注册微应用

在[官方文档](https://qiankun.umijs.org/zh/guide/getting-started#2-在主应用中注册微应用)中，我们可以看到第一个步骤便是注册微应用，它调用方法 `registerMicroApps` ：

``` jsx
import { registerMicroApps, start } from 'qiankun';

registerMicroApps([
  {
    name: 'reactApp',
    entry: '//localhost:3000',
    container: '#container',
    activeRule: '/app-react',
  },
  {
    name: 'vueApp',
    entry: '//localhost:8080',
    container: '#container',
    activeRule: '/app-vue',
  },
  {
    name: 'angularApp',
    entry: '//localhost:4200',
    container: '#container',
    activeRule: '/app-angular',
  },
]);
// 启动 qiankun
start();
```

注册列表项需要提供一下信息：

* 微应用名称；
* 微应用入口文件地址；
* 微应用在主应用的挂载点；
* 初始化的路由规则。

这其实就是我们在[第二篇](/blog/micro-frontends-2-implementation/#microapplication)中 `MicroApplication` 做的事情，这里通过一个数组来管理。

在实际场景下，这个数组可以作为一个`动态配置列表`，在 `CI 环境` 产生并更新，这样在发布一个新的微应用，我们只需要关注该配置是否正确即可，而主应用也不需要重新发布。

在调用 `start` 之前，各个微应用会被`下载`，但不会被`初始化`、`挂载`或`卸载`。

#### registerMicroApps

主应用的第一步便是`注册微应用`，它通过 `registerMicroApps` 方法来实现，代码如下：

``` ts
export function registerMicroApps<T extends object = {}>(
  apps: Array<RegistrableApp<T>>,
  lifeCycles?: FrameworkLifeCycles<T>,
) {
  // 防止重复注册
  const unregisteredApps = apps.filter(app => !microApps.some(registeredApp => registeredApp.name === app.name));

  microApps = [...microApps, ...unregisteredApps];

  unregisteredApps.forEach(app => {
    const { name, activeRule, loader = noop, props, ...appConfig } = app;

  // 调用【single-spa】的 registerApplication 方法
    registerApplication({
      name,
      app: async () => {
        loader(true);
        await frameworkStartedDefer.promise;

        const { mount, ...otherMicroAppConfigs } = await loadApp(
          { name, props, ...appConfig },
          frameworkConfiguration,
          lifeCycles,
        );

        return {
          mount: [async () => loader(true), ...toArray(mount), async () => loader(false)],
          ...otherMicroAppConfigs,
        };
      },
      activeWhen: activeRule,
      customProps: props,
    });
  });
}
```

上面的方法做了一次剔除`重复注册的app`后，就直接调用了 `single-spa` 的 `registerApplication` 方法。

这里的防重复注册其实在 `registerApplication` 里也有处理。只是源码中以直接`抛出异常`方式处理。

#### registerApplication

registerApplication 方法定义在**src/applications/apps.js**文件中，我们来看看它做了哪些事:

``` jsx
export function registerApplication(
  appNameOrConfig,
  appOrLoadApp,
  activeWhen,
  customProps
) {

  
  // 统一规整参数项
  const registration = sanitizeArguments(
    appNameOrConfig,
    appOrLoadApp,
    activeWhen,
    customProps
  );

  // 这里做了防止微应用重复注册处理

  // 添加到 apps 的列表里。
  apps.push(
    assign(
      {
        loadErrorTime: null,
        status: NOT_LOADED, // 这里标记为 未加载 状态
        parcels: {},
        devtools: {
          overlays: {
            options: {},
            selectors: [],
          },
        },
      },
      registration
    )
  );

  if (isInBrowser) {
    // 使用 jQuery 提供的事件总线
    ensureJQuerySupport();
    // 加载并路由页面
    reroute();
  }
}
```

上面的代码做了几个主要工作：

1. 把`微应用配置`，标记为`未加载 (NOT_LOADED)`状态，然后添加到 `apps` 列表里去；
2. 在浏览器环境下，如果存在 `jQuery`，那么 jQuery 的`事件总线`也会`捕获路由事件消息`。在下面的代码可以看到，源码使用 `window.dispatchEvent` 来派发消息。；
3. 在浏览器环境下，如果主应用已经启动，那么只需路由改变操作。否则，加载

目前为止，我们还没看到没有任何执行`微应用加载的操作`，所以接下来 reroute 方法里做的事将会是我们关注的焦点：

``` javascript
export function reroute(pendingPromises = [], eventArguments) {
  if (appChangeUnderway) {
    return new Promise((resolve, reject) => {
      peopleWaitingOnAppChange.push({
        resolve,
        reject,
        eventArguments,
      });
    });
  }

  // 获取各自未激活下各个状态的微应用列表
  const {
    appsToUnload,
    appsToUnmount,
    appsToLoad,
    appsToMount,
  } = getAppChanges();

  let appsThatChanged,
    navigationIsCanceled = false,
    oldUrl = currentUrl,
    newUrl = (currentUrl = window.location.href);

  if (isStarted()) {
    appChangeUnderway = true;
    appsThatChanged = appsToUnload.concat(
      appsToLoad,
      appsToUnmount,
      appsToMount
    );
    return performAppChanges();
  } else {
    appsThatChanged = appsToLoad;
    return loadApps();
  }

  // 其它内部方法...
}
```

我们先关注主应用第一次未启动下，执行的 `loadApps` 方法。

#### 未启动时

##### loadApps

``` javascript
function loadApps() {
  return Promise.resolve().then(() => {
    const loadPromises = appsToLoad.map(toLoadPromise);

    return (
      Promise.all(loadPromises)
        .then(callAllEventListeners)
        // there are no mounted apps, before start() is called, so we always return []
        .then(() => [])
        .catch((err) => {
          callAllEventListeners();
          throw err;
        })
    );
  });
} 
```

可以从源码注释中了解到，在调用 `start` 方法之前，不会有`已挂载`的 `apps` 。

如果你查阅 `getAppChanges` 方法，会发现 `appsToLoad` 保存着我们即将加载的 app。

在上面的的代码中，核心的函数是 `toLoadPromise`，直观上来看，是把 app 转为 Promise，来执行异步操作：

##### toLoadPromise

该函数存在 `src/lifecycles/load.js` 中，并内容仅此一个函数，除去不相关代码我们可以看到：

``` javascript
export function toLoadPromise(app) {
  return Promise.resolve().then(() => {
    // 其它代码...
    app.status = LOADING_SOURCE_CODE;

    let appOpts, isUserErr;

    return (app.loadPromise = Promise.resolve()
      .then(() => {
        const loadPromise = app.loadApp(getProps(app));
        // 其它代码...
        return loadPromise.then((val) => {
          app.loadErrorTime = null;

          appOpts = val;

          /**
           * 此处省去的代码处理以下内容:
           * 1. 校验 bootstrap、mount、unmount 函数的有效性，否者抛出异常
           * 2. devtools overlays 处理
           */
          
          app.status = NOT_BOOTSTRAPPED;
          app.bootstrap = flattenFnArray(appOpts, "bootstrap");
          app.mount = flattenFnArray(appOpts, "mount");
          app.unmount = flattenFnArray(appOpts, "unmount");
          app.unload = flattenFnArray(appOpts, "unload");
          app.timeouts = ensureValidAppTimeouts(appOpts.timeouts);

          delete app.loadPromise;

          return app;
        });
      })
      .catch((err) => {
        delete app.loadPromise;

        let newStatus;
        if (isUserErr) {
          newStatus = SKIP_BECAUSE_BROKEN;
        } else {
          newStatus = LOAD_ERROR;
          app.loadErrorTime = new Date().getTime();
        }
        handleAppError(err, app, newStatus);

        return app;
      }));
  });
}
```

在经过上面的代码处理后，我们的 `app` 的状态将将变成`未启动` (NOT_BOOTSTRAPPED) ，并且 app 的勾子函数都已准备就绪。

至此，我们可以知道在主应用`未启动`的情况下，只做了`加载 app`前的准备工作，并未真正开始执行`初始化`操作。

接下来，我们将跟踪[注册微应用](/blog/micro-frontends-3-qiankun/#注册微应用)这小节中的 `start` 方法，看它是如何启动。

#### 启动 (bootstrap & mount) 工作

##### start

它的代码如下：

``` typescript

export function start(opts: FrameworkConfiguration = {}) {
  frameworkConfiguration = { prefetch: true, singular: true, sandbox: true, ...opts };
  const { prefetch, sandbox, singular, urlRerouteOnly, ...importEntryOpts } = frameworkConfiguration;

  if (prefetch) {
    doPrefetchStrategy(microApps, prefetch, importEntryOpts);
  }

  if (sandbox) {
    if (!window.Proxy) {
      console.warn('[qiankun] Miss window.Proxy, proxySandbox will degenerate into snapshotSandbox');
      // 快照沙箱不支持非 singular 模式
      if (!singular) {
        console.error('[qiankun] singular is forced to be true when sandbox enable but proxySandbox unavailable');
        frameworkConfiguration.singular = true;
      }
    }
  }

  startSingleSpa({ urlRerouteOnly });

  frameworkStartedDefer.resolve();
}
```

我们这边先不关注`预加载`和`沙箱`，而把重心放在它是如何启动的。所以重心落在了 `single-spa` 提供的 `start` 方法上，引入的时候被重名为 `startSingleSpa` 。

``` typescript
import { start as startSingleSpa } from 'single-spa';
```

 `single-spa` 的 `start` 代码如下：

``` javascript
export function start(opts) {
  started = true;
  if (opts && opts.urlRerouteOnly) {
    setUrlRerouteOnly(opts.urlRerouteOnly);
  }
  if (isInBrowser) {
    reroute();
  }
}
```

该方法很简单，可以发现它在主应用标记`已启动`状态后，就执行了 `reroute` 方法。

上面我们已经分析了在未启动状态下，`reroute` 对 app 进行了准备工作。而接下来 我们来看看已启动后的，`reroute` 里执行的另一个执行方法 `performAppChanges` 。

##### performAppChanges

它的代码如下：

``` javascript
function performAppChanges() {
    return Promise.resolve().then(() => {

      // https://github.com/single-spa/single-spa/issues/545

      // 其它代码...
      // 上面的代码为处理派发事件消息

      // 完成卸载
      const unloadPromises = appsToUnload.map(toUnloadPromise);

      const unmountUnloadPromises = appsToUnmount
        .map(toUnmountPromise)
        .map((unmountPromise) => unmountPromise.then(toUnloadPromise));

      const allUnmountPromises = unmountUnloadPromises.concat(unloadPromises);

      const unmountAllPromise = Promise.all(allUnmountPromises);

      unmountAllPromise.then(() => {
        window.dispatchEvent(
          new CustomEvent(
            "single-spa:before-mount-routing-event",
            getCustomEventDetail(true)
          )
        );
      });

      /* We load and bootstrap apps while other apps are unmounting, but we
       * wait to mount the app until all apps are finishing unmounting
       */
      const loadThenMountPromises = appsToLoad.map((app) => {
        return toLoadPromise(app).then((app) =>
          tryToBootstrapAndMount(app, unmountAllPromise)
        );
      });

      /* These are the apps that are already bootstrapped and just need
       * to be mounted. They each wait for all unmounting apps to finish up
       * before they mount.
       */
      const mountPromises = appsToMount
        .filter((appToMount) => appsToLoad.indexOf(appToMount) < 0)
        .map((appToMount) => {
          return tryToBootstrapAndMount(appToMount, unmountAllPromise);
        });
      return unmountAllPromise
        .catch((err) => {
          callAllEventListeners();
          throw err;
        })
        .then(() => {
          /* Now that the apps that needed to be unmounted are unmounted, their DOM navigation
           * events (like hashchange or popstate) should have been cleaned up. So it's safe
           * to let the remaining captured event listeners to handle about the DOM event.
           */
          callAllEventListeners();

          return Promise.all(loadThenMountPromises.concat(mountPromises))
            .catch((err) => {
              pendingPromises.forEach((promise) => promise.reject(err));
              throw err;
            })
            .then(finishUpAndReturn);
        });
    });
  }
```

在执行 app 状态发生改变时，会通过 `window.dispatchEvent` 来派 `app 状态改变事件` 消息。

我们从[官网文档](https://zh-hans.single-spa.js.org/docs/api#events)对于其触发事件时序有明确描述：

| 事件排序 | 事件名称                                                            | 消费条件                  |
| :------: | ------------------------------------------------------------------- | ------------------------- |
|    1     | `single-spa:before-app-change` 或 `single-spa:before-no-app-change` | 任意 app 将发生状态改变   |
|    2     | `single-spa:before-routing-event`                                  | -                         |
|    3     | `single-spa:before-mount-routing-event`                            | -                         |
|    4     | `single-spa:before-first-mount`                                    | 第一次任意 app 正在挂载中 |
|    5     | `single-spa:first-mount`                                           | 第一次任意 app 已挂载     |
|    6     | `single-spa:app-change` 或 `single-spa:no-app-change`              | 任意 app 发生状态改变     |
|    7     | `single-spa:routing-event`                                         | -                         |

在上面的代码中，首先是把 `appsToUnmount` 和 `appsToUnload` 里的 app 都标记为`卸载中` (UNMOUNTING) 状态，完成后发送 `single-spa:before-mount-routing-event` 事件。

也就是说，在`启动 app` (bootstrap) 和 `mount` 操作之前，必须先把要卸载的 app 处理完成。

最后，当卸载操作完成后，才真正开始 `bootstrap` 和 `mount`，它由 `tryToBootstrapAndMount` 方法完成。

##### tryToBootstrapAndMount

然我们来看看它的代码：

``` javascript
function tryToBootstrapAndMount(app, unmountAllPromise) {
  if (shouldBeActive(app)) {
    return toBootstrapPromise(app).then((app) =>
      unmountAllPromise.then(() =>
        shouldBeActive(app) ? toMountPromise(app) : app
      )
    );
  } else {
    return unmountAllPromise.then(() => app);
  }
}
```

可以看到上面执行 bootstrap 的方法为 `toBootstrapPromise`，它也是 `lifecycles` 中方法之一。

##### toBootstrapPromise

它的代码如下：

``` javascript
export function toBootstrapPromise(appOrParcel, hardFail) {
  return Promise.resolve().then(() => {
    if (appOrParcel.status !== NOT_BOOTSTRAPPED) {
      return appOrParcel;
    }

    appOrParcel.status = BOOTSTRAPPING;

    if (!appOrParcel.bootstrap) {
      // Default implementation of bootstrap
      return Promise.resolve().then(successfulBootstrap);
    }

    return reasonableTime(appOrParcel, "bootstrap")
      .then(successfulBootstrap)
      .catch((err) => {
        if (hardFail) {
          throw transformErr(err, appOrParcel, SKIP_BECAUSE_BROKEN);
        } else {
          handleAppError(err, appOrParcel, SKIP_BECAUSE_BROKEN);
          return appOrParcel;
        }
      });
  });

  function successfulBootstrap() {
    appOrParcel.status = NOT_MOUNTED;
    return appOrParcel;
  }
}
```

 `toBootstrapPromise` 方法里，把 app 状态标记为`启动中` (BOOTSTRAPPING) ，而后调用了 `reasonableTime` 方法。

##### reasonableTime

``` javascript
export function reasonableTime(appOrParcel, lifecycle) {
  const timeoutConfig = appOrParcel.timeouts[lifecycle];
  const warningPeriod = timeoutConfig.warningMillis;
  const type = objectType(appOrParcel);

  return new Promise((resolve, reject) => {
    let finished = false;
    let errored = false;

    appOrParcel [lifecycle](getProps(appOrParcel) )
      .then((val) => {
        finished = true;
        resolve(val);
      })
      .catch((val) => {
        finished = true;
        reject(val);
      });

    setTimeout(() => maybeTimingOut(1), warningPeriod);
    setTimeout(() => maybeTimingOut(true), timeoutConfig.millis);

    // 其它代码...

    function maybeTimingOut(shouldError) {
      // 这里代码处理超时
    }
  });
} 
```

 `reasonableTime` 方法里做了生命周期勾子执行过慢的提醒，直到执行时长大于设定的超时时间，那么判定超时并抛出异常。而在这里就表示 `bootstrap` 超时。

你可以在[这里](https://github.com/single-spa/single-spa/blob/a79e58ba2e9c7d41216b856b1c6b2edc80e392e3/src/applications/timeouts.js#L108)看到该方法的实现，并了解到其中定义了常量 `globalTimeoutConfig` 来表示各个生命周期中的默认超时设定。

同时，我们可以下面这里为真正执行生命周期勾子的地方：

``` javascript
appOrParcel [lifecycle](getProps(appOrParcel) )
```
