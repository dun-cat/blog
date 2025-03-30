## Webpack 5 之 模块联合 (Module Federation)  
### 微前端

在了解 webpack 中 `模块联合`的之前，我们先了解一下`微前端`，如果你对微前端不了解可以查阅这里两篇文章[《微前端 (一) - 理念篇》](/blog/micro-frontends-1-concept/)和[《微前端 (二) - 实现篇》](/blog/micro-frontends-2-implementation/)来熟悉它的基本概念。

通常我们的微前端的模型如下：

<img src="micro-fe.svg" />

在微前端中，会存在一个`容器应用`，它的任务就是加载各个`微应用`。

微应用需要做一些事：

* 提供两个方法：一个是`挂载方法`，容器将调用它来渲染微应用。另一个是`卸载方法`，用于卸载微应用，并且他们都将以`接口方法`的形式提供给容器调用；
* 提供`远程入口文件`的地址，容器应用选择合适的时机`动态加载`该文件，在获得`挂载方法`后，执行微应用渲染；
* 提供`微应用 ID`，用于标识自己，对微应用的操作需要该标识；
* 首个`路由地址`，在挂载微应用后，决定微应用的视图展示。

容器应用要做的事是：

* 加载远程的微应用 (下载远程 js 入口文件) ，并执行渲染；
* 在合理的契机，卸载微应用。

以上便是微前端的基本功能。接下来，我们再来看看 webpack 的模块联合 (module federation) 。

### 模块联合

通常在使用 webpack 构建产生的`模块`都存储在`本地`，直接被当前应用所使用。在 webpack 5 中提出了`远程模块`的概念，允许`运行时`把当前构建的应用作为`容器应用`，异步加载`远程模块`。下面将用简称 `MF` 指代模块联合。

> webpack 的提供了`动态加载模块`的方式，你可以使用 [import](https://webpack.docschina.org/api/module-methods/#import) 或者较为陈旧的方法[require.ensure](https://webpack.docschina.org/api/module-methods/#requireensure)或 `require([...])` 。

记得上面小节在我们说微前端中，容器应用做的事吗？其实通过 `webpack` 的`动态加载`，就已经实现了容器应用该做的事情。所以我们完全可以认为微应用本身可以具备容器应用的功能。

当我们把`微应用`作为`容器应用时`，那么它的架构模型就发生转变，于是会产生下面的模型：

<img src='mf.svg' />

可以看到，当我们的`微应用`成为`容器应用`后，每个应用在架构里都平等得存在，容器应用之间可以`相互的依赖`和`相互的加载及使用`。

在微前端中，并没有对`微应用`的`复杂度`做任何架构上的约束，也就说它可能只是个按钮，而这个按钮却引爆了地球，当然这是个玩笑。但我们应该从业务上合理划为它，让其成为一个有价值的复用模块并且不受框架束缚。

而如何通过构建工具对应用进行`模块划分`、`模块共享`、`模块加载`，我想这便是 webpack 5 模块联合 (Module Federation) 的`功能意义`所在。

### MF vs 微前端

我们继续思考模块联合和微前端的区别。

在微前端中：

* 加载微应用必须`预定义`接口方法 (mounted、unmount 等) 来实现微应用的动态`挂载`和`卸载`等功能，这意味着每个微应用必须`手动`实现这些`接口方法`；
* 在[《微前端 (二) - 实现篇》](/blog/micro-frontends-2-implementation/)中，我们了解到微应用在`独立开发模式`下，通常也是手动调用接口方法，来动态加载视图；
* 如果我们想要共享某个微应用的模块给其它微应用使用，这并不是轻松地事。这意味着你需要把该模块独立出去，并以合理调用方式被其它微应用`远程加载`。
* 微应用的`切换`通常由`路由状态改变`来触发的。

在模块联合中：

* 上面我们了解了模块联合每个`微应用`可以是一个`容器应用`，所以他们之间可以相互`依赖`及`加载`；
* 每个应用允许`暴露` (exposes) 多个接口，其它应用可以在`动态远程加载`该应用后，直接使用其接口。这解决了上面微前端提到的的`模块共享`问题；
* 在模块使用上非常灵活，当你`引用`一个`远程模块`时，可以像使用普通的 npm 包一样使用它，当然也允许`懒加载`模块；
* 远程模块和路由没有任何关联，加载的契机完全由 host 应用自己灵活决定。

值得注意的`联合模块`作为微前端的技术延展，其依然具备着`微前端`的特性，即每个容器应用应该`独立开发`和`独立部署`，并`团队自治`。

模块联合的架构模型更像下图展示一样，当然不只这一种，因为它非常的灵活。这取决于你如何共享模块和组合他们。

<img src="mf1.svg" />

在上面的架构图中：

* `APP A`、`APP B`、`APP C` 都远程 (remote) 加载并使用 `UI 组件库` 中暴露的 `Button` 和 `Text` 组件，`Table` 组件由于未稳定下来，我们不准备暴露给外部使用；
* `APP B` 和 `APP C` 中的 `List` 模块都共享给 `APP A` 所使用 (例如：业务 B 的订单列表和业务 C 的订单列表都可以直接被集成到业务 A 之中) ；
* `身份验证`应用作为公共模块，被 `APP A`、`APP B` 和 `APP C`，我们不需要单独给新应用添加额外的身份验证模块，它将作为基础服务。

<!-- 作为构建工具，webpack 5 提供的插件能很好的让我们划为模块， -->

### ModuleFederationPlugin

Webpack 5 通过 `ModuleFederationPlugin` 来实现`模块接口暴露`和`远程模块声明`的工作。

 `ModuleFederationPlugin` 插件组合了 `ContainerPlugin` 和 `ContainerReferencePlugin` 。

ContainerPlugin 插件使用`指定`的公开模块来创建一个`额外`的`容器入口`，这意味除了配置的输出文件 (output) ，还会产生额外的`容器入口文件`。

``` js
module.exports = {
  output: {
    filename: 'main.js',
  },
};
```

ContainerReferencePlugin 插件允许我们在使用`远程模块`时，以 import 标准语法方式使用，所以需要我们提前声明远程模块。

 `ModuleFederationPlugin` 允许构建一个作为`提供者`或`消费者`概念的`运行时独立模块`，每个应用都可以成为提供者或消费者。

``` typescript
const { ModuleFederationPlugin } = require('webpack').container;
module.exports = {
  plugins: [
    new ModuleFederationPlugin({/* options */}),
  ],
};
```

你可以在[这里](https://github.com/webpack/webpack/blob/beb42c64f696584ef570d8f57df448ddd0ac7238/types.d.ts#L6427)看到所有 `options` 选项。

#### 容器入口文件

首先，我们需要提供`容器入口文件` (container entry) 来让其它应用能够远程加载该文件：

``` js
new ModuleFederationPlugin({
    name: "ui_lib", // 容器名称
    filename: 'ui.js', // 容器入口文件
})
```

你需要提供一个`唯一的`容器的`名称` (name) 和`文件名` (filename) ，若没有提供 `filename`，那么构建生成的文件名与`容器名称`同名。

构建后，会在 dist 目录里产生 `ui.js` 的额外容器入口文件。

#### 暴露 (expose) 多个模块

你可以暴露任何你想要分享出去的模块，它可以是`网络库`、`公用业务模块`、`UI 组件`、`路由`、`hooks` 以及任何你觉得可以分享出去的任何东西，这听起来很振奋人心，而事实也确实如此。

我们通过 `exposes 选项` 来暴露模块：

``` js
new ModuleFederationPlugin({
  name: "ui_lib", // 容器名称
  filename: 'ui.js', // 容器入口文件
  exposes: {
    "./components": "./src/components/",
  },
})
```

假如我们的 UI 库的入口在项目 `src/components/index.js` 文件里，那么该文件应该是这样的：

``` jsx
export { default as Button } from './button/index.jsx'
export { default as Text } from './text/index.jsx'
```

### 共享模块

容器通常存在基础的重复依赖库 (例如：react、vue 等等) 。和介绍微前端文章的共享库一样，我们也需要将其从我们的容器排除出去，而让他们作为异步模块加载。

不仅如此，MF 对共享模块做了`版本化`管理，你可以在[这个 PR 的交流](https://github.com/webpack/webpack/pull/10960)获取相关信息。

同样我们使用 `ModuleFederationPlugin` 插件中的 `shared 选项` 来指定公共模块异步模块加载使用，它的功能和 webpack 的 [externals](https://webpack.docschina.org/configuration/externals/#root) 类似，允许在运行时加载外部依赖库。

``` js
new ModuleFederationPlugin({
  name: "ui_lib",
  filename: 'ui.js',
  exposes: {
    "./components": "./src/components/",
  },
  shared: {
    react: { singleton: true},
    "react-dom": { singleton: true}
  },
})
```

你可以在[这里](https://github.com/webpack/webpack/blob/beb42c64f696584ef570d8f57df448ddd0ac7238/declarations/plugins/container/ModuleFederationPlugin.d.ts#L265)看到所有 `shared 选项` 。

#### 注意点

1.如果你想要在本地启动项目时使用`共享模块` (shared module) ，需要指定 `eager: true` 的选项，否则将会出现下面的错误。

``` text
Uncaught Error: Shared module is not available for eager consumption
```

该选项允许`共享模块`在初始化的时候直接使用，也就是说不会把它作为一个异步模块来加载。

> 需要注意的是开启 `eager 选项`，它会将模块直接打入容器文件中，作为同步模块加载并使用。

你也可以通过下面的修改，修复上面的问题，即`手动异步加载`共享模块。

首先，我们把原来的 `src/index.js` 文件做一些修改。

以前的你入口文件像下面这样：

``` jsx
import React from 'react';
import ReactDOM from 'react-dom';
import App from './App';
ReactDOM.render(<App />, document.getElementById('root'));
```

修改之后，我们只保留异步加载的功能：

``` jsx
import('./bootstrap');
```

然后，我在同级目录下创建 `bootstrap.js` 的启动文件，把原来的 `index.js` 内容复制进来：

``` jsx
import React from 'react';
import ReactDOM from 'react-dom';
import App from './App';
ReactDOM.render(<App />, document.getElementById('root'));
```

这样就实现了手动异步加载。

---

2.我们通过 `requiredVersion 选项` 来使用指定共享模块的版本。

它有两个值：requiredVersion 为 `string` 类型的值时，表示遵循 [semver](https://semver.org/lang/zh-CN/) 规范的语义化版本号。

你可以直接用 `package.json` 里的 `dependencies` 字段中包名对应版本，这样做是为了共享模块的版本和 `package.json` 中的版本保持一致。如果不一致则会打印警告。

``` js
const deps = require("./package.json").dependencies;
// other code...
new ModuleFederationPlugin({
  name: "ui_lib",
  filename: 'ui.js',
  exposes: {
    "./components": "./src/components/",
  },
  shared: {
    react: {
      requiredVersion: deps.react,
      singleton: true,
    },
    "react-dom": {
      requiredVersion: deps['react-dom'],
      singleton: true,
    }
  }
})
```

requiredVersion 为 `boolean` 类型的值时，表示是否启动版本号`自动推断`。当其为 `true` (默认值) 时，请求的模块自动根据 `package.json` 中的`包名`对应的版本做推断。

### 使用远程模块

首先，同样我们使用 `ModuleFederationPlugin` 插件，提前`声明`哪些是远程模块，这里通过 `remotes 选项` 进行设置：

``` js
new ModuleFederationPlugin({
  name: "app_b",
  remotes: {
    "@lumin-ui": 'ui_lib@http://localhost:3003/ui.js',
  }
})
```

我们在运行时使用时，它和我们平时使用 import 语法没有任何区别：

``` jsx
import { Button, Text } from '@lumin-ui/components';
```

这看起来非常的酷！对于架构升级而言，我们将本地构建模块替换成远程模块并不需要修改任何代码。

### 演示源码

你可以在下面的地址找到以上用例的演示源码，该用例并不完备，但展示了 MF 的基本功能。

[https://github.com/dun-cat/webpack-module-federation](https://github.com/dun-cat/webpack-module-federation)

通过下面的步骤启动项目：

``` bash
# 安装依赖
npm run bootstrap

# 启动项目
npm run start
```

### 其它有趣的用例

上面演示了 MF 中的其中一个 UI 库用例，你可以在[这里](https://github.com/module-federation/module-federation-examples)找到更多用例。

#### shared-routing

我建议你在看看[shared-routing](https://github.com/module-federation/module-federation-examples/tree/master/shared-routing)这个用例，该用例展示了一个完整的应用如何进行`模块划分`，更为重要的是每个模块都获取了`完整的`应用。

你必须知道`模块划分`也意味着`项目划分`和`任务划分`。在独立开发时，通常需要确认如何`保证整个应用的正确性`。所以我们期望自己开发的模块能够运行在整个应用中，而这个例子提供了很好的解决方案。

<img src="mf_routing.svg" />

参考资料：

\> [https://webpack.docschina.org/concepts/module-federation/](https://webpack.docschina.org/concepts/module-federation/)

\> [https://www.bilibili.com/video/BV1z5411K7us](https://www.bilibili.com/video/BV1z5411K7us)

\> [https://www.youtube.com/watch?v=-ei6RqZilYI&ab_channel=Pusher](https://www.youtube.com/watch?v=-ei6RqZilYI&ab_channel=Pusher)

\> [https://github.com/module-federation/module-federation-examples](https://github.com/module-federation/module-federation-examples)

\> [https://www.nicolasdelfino.com/blog/micro-frontends-module-federation-webpack](https://www.nicolasdelfino.com/blog/micro-frontends-module-federation-webpack)
