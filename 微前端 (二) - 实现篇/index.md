## 微前端 (二) - 实现篇 
该演示全部使用 React.js 构建，React 并没有垄断该架构。你可以使用许多不同的工具或框架来实现微前端。我们选择 React 是因为它很受欢迎，也因为对它很熟悉。

为了方便演示，使用 [lerna](https://github.com/lerna/lerna) 来管理多个项目，真实场景下他们都应该有`单独的版本控制库`来`单独开发`和`单独部署`。

你可以在 Github 上获取所有演示代码：[https://github.com/dun-cat/micro-frontends](https://github.com/dun-cat/micro-frontends)。

### 组成部分

微前端会包含两种应用类型：一种是`容器应用`，另一种便是`微应用`，或者叫做微前端。容器作为壳，用于承载`微应用`。

#### 容器应用

容器部分，主体功能需要实现：

* 根据`微应用 ID`装载对应的微应用，并渲染他们；
* 在离开微应用的时候，卸载它。

#### 微应用

微应用部分，主体功能需要实现：

* 接收一个挂载 的元素ID，并暴露一个渲染函数来实现动态渲染。
* 实现业务逻辑，并能独立开发和独立部署。

### 容器应用

要了解`微应用`以何种方式打开，可以通过容器应用的`路由代码`了解到：

``` jsx
// :packages/container/src/App.js
<Switch>
  <Route exact path="/" component={MicroAppA} />
  <Route exact path="/app-a/:id" component={MicroAppA} />
  <Route exact path="/app-b" render={MicroAppB} />
  <Route exact path="/about" render={About} />
</Switch>
```

#### MicroApplication

我们创建了一个 `MicroApplication` 的 React 组件，让它提供`微应用`的挂载点，并加载我们的微应用。

``` jsx
const MicroAppA = ({ history }) => (
  <MicroApplication history={history} host={appAHost} name="MicroAppA" />
);
const MicroAppB = ({ history }) => (
  <MicroApplication history={history} host={appBHost} name="MicroAppB" />
);
```

MicroApplication 的代码如下：

``` jsx
class MicroFrontend extends React.Component {
  
  componentDidMount() {
    // 下载微应用脚本到页面，并渲染。
  }
  componentWillUnmount() {
    // 从容器卸载
  }

  renderMicroApplication() {
    // 渲染微应用到容器
  }

  render() {
    return <main id={`${this.props.name}-container`} />;
  }
}
```

##### componentDidMount

我们将在 `componentDidMount` 里加载`微应用`，它的代码如下：

``` jsx
componentDidMount() {
  const { name, host } = this.props;
  const scriptId = `micro-frontend-script-${name}` ;

  if (document.getElementById(scriptId)) {
    this.renderMicroApplication();
    return;
  }

  fetch(`${host}/asset-manifest.json`)
    .then(res => res.json())
    .then(manifest => {
      const script = document.createElement('script');
      script.id = scriptId;
      script.src = `${host}${manifest.files['main.js']}`;
      script.onload = this.renderMicroApplication;
      document.head.appendChild(script);
    });
}
```

首先，我们检查是否已经下载了具有`唯一 ID`的相关脚本。在这种情况下，我们可以`立即渲染`它。

如果没有，根据 `asset-manifest.json` 从主机上获取`主脚本下载的 URL`。

在执 `react-scripts build` 命令后，我们将会在输出文件夹 `build 目录` 获得该`资产清单文件`：

``` json
{
  "files": {
    "main.js": "/static/js/main.4b0094e4.chunk.js",
    "main.js.map": "/static/js/main.4b0094e4.chunk.js.map",
    "runtime-main.js": "/static/js/runtime-main.dd88af7e.js",
    "runtime-main.js.map": "/static/js/runtime-main.dd88af7e.js.map",
    "static/js/2.8e621a23.chunk.js": "/static/js/2.8e621a23.chunk.js",
    "static/js/2.8e621a23.chunk.js.map": "/static/js/2.8e621a23.chunk.js.map",
    "index.html": "/index.html",
    "static/js/2.8e621a23.chunk.js.LICENSE.txt": "/static/js/2.8e621a23.chunk.js.LICENSE.txt"
  },
  "entrypoints": [
    "static/js/runtime-main.dd88af7e.js",
    "static/js/2.8e621a23.chunk.js",
    "static/js/main.4b0094e4.chunk.js"
  ]
}
```

> 我们必须从`资产清单文件` (asset-manifest.json) 中获取脚本的 URL，在 `react-scripts` 输出的已编译 JavaScript `文件名`包含`哈希值`，用以方便缓存。

一旦我们设置了脚本的 URL，剩下的就是把它加到 `documet` 里去，使用一个 `onload 处理程序` 来渲染微应用：

``` jsx
renderMicroApplication = () => {
  const { name, window, history } = this.props;
  window[ `render${name}` ](`${name}-container`, history);
  // 例如：window.renderMicroAppA('MicroAppA-container', history);
};
```

在上面的代码中，我们调用了一个名为 `window.renderMicroAppA` 的全局函数，它是由我们刚刚下载的脚本放在那里的。

我们将`<main>`微应用应该呈现的`元素的 ID`和一个 `history 对象` 传递给它，**这个全局函数的签名是容器应用和微应用之间的关键约定** 。

这是`任何通信`或`集成`应该发生的地方，因此保持相当`轻量级`使其`易于维护`，并在未来添加新的微应用。

> 每当我们想要做一些需要更改`此代码`的事情时，我们应该仔细考虑它对我们的`代码库的耦合`以及`约定`的维护意味着什么。

##### componentWillUnmount

还有最后一件，就是`卸载微应用`。当我们的 MicroApplication 组件卸载 (从 DOM 中删除) 时，我们也想卸载相关的微应用。

为此，每个微应用定义了一个相应的`全局函数`，我们从相应的 React 生命周期方法中调用它：

``` jsx
componentWillUnmount() {
  const { name } = this.props;

  window[ `unmount${name}` ](`${name}-container`);
}
```

容器的`顶级标题`和`导航栏`的 CSS 目前小心地被编写，以确保它只会为标题中的元素设置样式，因此它不应与微应用中的任何样式代码冲突。

该容器应用虽然比较初级的，但为我们提供了一个壳，可以`运行时动态下载`我们的微应用，并紧密得组合到单页面 (SPA) 里去。这些微应用可以一直`独立部署`到`生产环境`，而无需对任何其他`微应用`或`容器本身`进行更改。

### 微应用

通过上面，我们知道`容器应用`需要和`微应用`进行接口`约定`，它的`入口文件`是这样的：

``` jsx
import React from 'react';
import ReactDOM from 'react-dom';
import App from './App';

window.renderMicroAppA = (containerId, history) => {
  ReactDOM.render(
    <App history={history} />,
    document.getElementById(containerId),
  );
};

window.unmountMicroAppA = containerId => {
  ReactDOM.unmountComponentAtNode(document.getElementById(containerId));
};
```

通常在 React.js 应用程序中，调用 `ReactDOM.render` 发生在顶级范围内，这意味着一旦此脚本文件被加载，它就会立即渲染到硬编码的 DOM 元素上。

而这个应用，我们需要能够`控制渲染`发生的`时间`和`地点`，因此我们将它包装在一个函数中，该函数接收 `DOM 元素的 ID` 作为参数，并将该函数添加到全局的 `window 对象` 。我们还可以看到相应的用于清理的`卸载函数`。

#### 独立开发及运行

虽然我们已经看到了当微应用集成到整个容器应用时如何调用这个函数，但成功的最大标准之一是我们可以`独立开发`和`独立运行`微应用。

因此，每个微应用也有自己 `index.html` 的`内联脚本`，可以在容器外部以“独立”模式渲染应用。

你需要针对 React 项目的 `public/index.html` 做出修改：

``` html
<html lang="en">
  <head>
    <title>MiroAppA</title>
  </head>
  <body>
    <noscript>You need to enable JavaScript to run this app.</noscript>
    <main id="container"></main>
    <script type="text/javascript">
      window.onload = () => {
        window.renderMicroAppA('container');
      };
    </script>
  </body>
</html>
```

前面小节我们在微应用中`动态得`控制渲染，所以在`独立`模式下，主动调用渲染函数，让它挂载到元素上去。

### 通过路由进行跨应用通信

我们在[上一篇](/blog/micro-frontends-1-concept/#跨应用通信)提到过，跨应用程序通信应该保持在最低限度。对于容器应用，我们创建了一个`<BrowserRouter>`，它将在内部实例化一个 `history 对象` 。我们使用这个对象来操作客户端历史，我们也可以使用它来将多个 `React Router` 链接在一起。

在我们的微应用中，会像这样做`初始化路由`：

``` jsx
<Router history={this.props.history}>
```

在这种情况下，我们不是让 `React Router` 实例化另一个 `history 对象`，而是为它提供容器应用程序传入的实例。

`<Router>`现在所有实例都已连接，因此在其中任何一个实例中触发`路由更改`都将反映在所有实例中。这为我们提供了一种通过 URL 将`参数`从一个微应用传递到另一个微应用的简单方法。

例如在浏览微应用，我们有一个这样的链接：

``` jsx
<Link to={`/app-a/${id}`}>
```

单击此链接时，容器中的路由将被更新。容器将根据新的 URL 并确定应`安装`和`渲染`哪个微应用。然后，该微应用自己的路由逻辑将从 URL 中提取 `ID` 并渲染正确的信息。

我们使用 URL 作为的通信手段，有以下的一些意图：

* 结构定义很好，并且是作为开放标准；
* 它能全局性访问到页面的任何代码；
* 对数据传输的尺寸限制，符合我们设计意图；
* 它是`声明性`的，而不是强制性的，如果你有更好的通信手段并不影响它的存在；
* 它迫使微应用`间接通信`，而不是直接相互依赖。

### 解决依赖重复

虽然我们希望我们的团队和我们的微应用尽可能独立，但有些事情应该是共同的。

上一篇我们介绍过`共享组件库`如何帮助实现跨微应用的一致性。

而我们可以在`微应用之间共享`的另一件事是：`库依赖项`。依赖重复是微前端的一个常见缺点，尽管跨应用共享这些依赖项有其自身的一系列困难，但对于演示来说，值得讨论如何实现。

#### 选出需要共享依赖项

我们对编译代码的包大小分析表明，大约 50% 的 bundles 是由 `react` 和 `react-dom` 贡献的。除此之外，这两个库是我们最“核心”的依赖项，我们所有微应用都可以从提取它们中受益。

最后，这些是`稳定`、`成熟`的库，通常会在两个主要版本之间引入重大更改，因此`跨应用升级`工作应该不会太困难。

#### 提取重复依赖

至于实际的提取，需要做的就是在项目的 `webpack 配置` 中将库标记为[外部库 (Externals)](https://webpack.docschina.org/configuration/externals/)。

我们通过[react-app-rewired](https://github.com/timarney/react-app-rewired/blob/master/README_zh.md)工具来修改 webpack 配置 。扩展的 `webpack 配置文件` 如下：

``` js
module.exports = (config, env) => {
  config.externals = {
    react: 'React',
    'react-dom': 'ReactDOM'
  }
  return config;
};
```

> react-app-rewired 工具可以在不 `eject` 也不创建额外 `react-scripts` 的情况下修改 `create-react-app` 内置的 webpack 配置，然后你将拥有 create-react-app 的一切特性，且可以根据你的需要去配置 webpack 的 plugins, loaders 等。

我们需要同时修改`容器应用`以及`微应用`的 `package.json` 文件：

修改前：

``` json
{
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test",
    "eject": "react-scripts eject"
  }
}
```

修改后：

``` json
{
  "scripts": {
    "start": "react-app-rewired start",
    "build": "react-app-rewired build",
    "test": "react-app-rewired test",
    "eject": "react-scripts eject"
  },
  "config-overrides-path": "node_modules/@dun-cat/react-app-rewire-micro-frontends",
  "devDependencies": {
    "@dun-cat/react-app-rewire-micro-frontends": "0.0.2",
    "react-app-rewired": "^2.1.8"
  }
}
```

我们注意到在 `package.json` 添加了自定义字段 `config-overrides-path` ：

``` json
{
  "config-overrides-path": "node_modules/@dun-cat/react-app-rewire-micro-frontends"
}
```

我们把扩展的 `webpack 配置文件` 封装到一个[npm 包 (@dun-cat/react-app-rewire-micro-frontends)](https://github.com/dun-cat/react-app-rewire-micro-frontends)里去，并通过 `config-overrides-path` 指向到文件所在位置。

> 把 webpack 配置以 npm 包的形式来管理，是对今后项目构建升级的维护考虑。我们希望项目的构建升级对已有的项目无任何入侵，甚至对于项目开发者来说是无感知的。

当做完以上工作后，我们重新执行微应用的`构建命令`时，就会发现资产清单 (asset-manifest.json) 只剩下下面一些文件了：

``` json
{
  "files": {
    "main.js": "/static/js/main.31f38797.js",
    "main.js.map": "/static/js/main.31f38797.js.map",
    "index.html": "/index.html",
    "static/js/main.31f38797.js.LICENSE.txt": "/static/js/main.31f38797.js.LICENSE.txt"
  },
  "entrypoints": [
    "static/js/main.31f38797.js"
  ]
}
```

#### 修改 index.html

我们的`共享依赖项`的 `script` 需要在`容器应用`和`每个微应用`的 `index.html` 中添加。他们将从`共享资源服务器`中获取我们的依赖库 `react` 和 `react-dom` 。

你可以在 `react` 源码的最新稳定版分支，通过下面命令获取这两个依赖库的 `UMD` 版本。

``` bash
yarn build react/index,react-dom/index --type=UMD
```

微应用的 `public/index.html` 的代码如下：

``` html
<body>
  <noscript>You need to enable JavaScript to run this app.</noscript>
  <main id="container"></main>
  <script src="%REACT_APP_CONTENT_HOST%/react.prod-17.0.3.min.js"></script>
  <script src="%REACT_APP_CONTENT_HOST%/react-dom.prod-17.0.3.min.js"></script>

  <script type="text/javascript">
    window.onload = () => {
      window.renderMicroAppA('container');
    };
  </script>
</body>
```

容器应用中 `public/index.html` 代码如下：

``` html
<body>
  <noscript>You need to enable JavaScript to run this app.</noscript>
  <div id="root"></div>    
  <script src="%REACT_APP_CONTENT_HOST%/react.prod-17.0.3.min.js"></script>
  <script src="%REACT_APP_CONTENT_HOST%/react-dom.prod-17.0.3.min.js"></script>
</body>
```

> `REACT_APP_CONTENT_HOST` 环境变量保存项目`根目录`在`.env`文件中，构建时替换变量。

### 共享资源服务器

你可以在`演示代码`里找到 `packages/shared-content` 的项目，该项目保存了微前端`公共静态资源`。例如我们的 `react` 和 `react-dom` 依赖项。

你可以进入该项目，看到它的启动配置：

``` json
{
  "scripts": {
    "start": "serve -p 5000 --cors content"
  }
}
```

### 跨域问题

当我们的容器应用加载微应用的资源文件时 (例如：资产清单) ，我们需要他们符合浏览器的`同源策略`。在本地开发的时候，我们在 `src` 目录里添加了一个 `setupProxy.js` 文件用于解决开发跨域问题。

它的代码如下：

``` javascript
module.exports = app => {
  app.use((req, res, next) => {
    res.header('Access-Control-Allow-Origin', '*');
    next();
  });
};
```

至此，我们的微前端实现，基本已经完成。它演示了一个微前端架构的基本原理。我相信你基于此能够更好的理解一些其它的`微前端架构`实现原理。

### 演示源码

你可以在 Github 上获取所有演示代码：[https://github.com/dun-cat/micro-frontends](https://github.com/dun-cat/micro-frontends)。

通过下面几个步骤很容易运行该项目：

**1.安装所有依赖**

``` bash
npm run bootstrap
```

**2.运行**

``` bash
npm run start
```

在执行上面的应用后，会独立运行`微应用 A`、`微应用 B`、`容器应用`以及`共享资源服务器`。
