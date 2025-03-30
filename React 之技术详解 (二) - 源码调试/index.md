## React 之技术详解 (二) - 源码调试 
### 源码文件结构

React 16 之后的架构分为三层：

* `调度器 (Scheduler)`：调度任务的优先级，高优任务优先进入 `Reconciler` ；
* `协调器 (Reconciler)`：负责找出变化的组件；
* `渲染器 (Renderer)`：负责将变化的组件渲染到页面上。

你可以通过官方文档[源码描述](https://zh-hans.reactjs.org/docs/codebase-overview.html)了解具体目录信息。

### 顶层目录

除去配置文件和隐藏文件夹，根目录的文件夹包括三个：

``` text
根目录
├── fixtures        # 包含一些给贡献者准备的小型 React 测试项目
├── packages        # 包含元数据 (比如 package.json) 和 React 仓库中所有 package 的源码 (子目录 src) 
├── scripts         # 各种工具链的脚本，比如 git、jest、eslint等
```

这里我们关注 packages 目录

### packages 目录

#### react 文件夹

React的核心，包含所有全局 React API，如：

* React.createElement
* React.Component
* React.Children
这些 API 是全平台通用的，它不包含 ReactDOM、ReactNative 等平台特定的代码。在 NPM 上作为单独的一个包发布。

#### scheduler 文件夹

Scheduler (调度器) 的实现。

#### shared 文件夹

源码中其他模块公用的方法和全局变量，比如在 shared/ReactSymbols.js 中保存 React 不同组件类型的定义。

``` javascript
// ...
export let REACT_ELEMENT_TYPE = 0xeac7;
export let REACT_PORTAL_TYPE = 0xeaca;
export let REACT_FRAGMENT_TYPE = 0xeacb;
// ...
```

#### Renderer 相关的文件夹

如下几个文件夹为对应的 Renderer：

``` text
- react-art
- react-dom                 # 注意这同时是 DOM 和 SSR (服务端渲染) 的入口
- react-native-renderer
- react-noop-renderer       # 用于debug fiber (后面会介绍fiber) 
- react-test-renderer
```

#### 试验性包的文件夹

React 将自己流程中的一部分抽离出来，形成可以独立使用的包，由于他们是试验性质的，所以不被建议在生产环境使用。包括如下文件夹：

``` text
- react-server        # 创建自定义 SSR 流
- react-client        # 创建自定义的流
- react-fetch         # 用于数据请求
- react-interactions  # 用于测试交互相关的内部特性，比如 React 的事件模型
- react-reconciler    # Reconciler的实现，你可以用他构建自己的 Renderer
```

#### 辅助包的文件夹

React将一些辅助功能形成单独的包。包括如下文件夹：

``` text
- react-is       # 用于测试组件是否是某类型
- react-client   # 创建自定义的流
- react-fetch    # 用于数据请求
- react-refresh  # “热重载”的 React 官方实现
```

#### react-reconciler 文件夹

我们需要重点关注 react-reconciler，在接下来源码学习中 80% 的代码量都来自这个包。

虽然他是一个实验性的包，内部的很多功能在正式版本中还未开放。但是他一边对接 Scheduler，一边对接不同平台的 Renderer，构成了整个 React16 的架构体系。

### 调试源码

即使版本号相同 (当前最新版为 v17.0.2) ，但是 `facebook/react` 项目 `main 分支` 的代码和我们使用 `create-react-app` 创建的项目 `node_modules` 下的 react 项目代码还是有些区别。

你可以通过官方文档[如何参与](https://zh-hans.reactjs.org/docs/how-to-contribute.html)了解详情。

为了始终使用最新版 React 教学，我们调试源码遵循以下步骤：

1. 从 facebook/react 项目 main 分支拉取最新源码；
2. 基于最新源码构建 react、scheduler、react-dom 三个包；
3. 通过 create-react-app 创建测试项目，并使用步骤 2 创建的包作为项目依赖的包。

#### 拉取源码

拉取 `facebook/react` ：

``` shell
# 拉取代码
git clone https://github.com/facebook/react.git

# 如果拉取速度很慢，可以考虑如下 2 个方案：

# 1. 使用 cnpm 代理
git clone https://github.com.cnpmjs.org/facebook/react

# 2. 使用码云的镜像 (一天会与 react 同步一次) 
git clone https://gitee.com/mirrors/react.git
```

#### 开发

``` bash
# 切入到react源码所在文件夹
cd react

# 安装依赖
yarn
```

执行构建

``` bash
yarn build
```

现在源码目录 build/node_modules 下会生成最新代码的包。我们为 react、react-dom 创建 `yarn link` 。

``` bash
cd build/node_modules/react
# 申明  react 指向
yarn link
cd build/node_modules/react-dom
# 申明 react-dom 指向
yarn link
```

#### 创建项目

接下来我们通过 `create-react-app` 在其他地方创建新项目。这里我们随意起名，比如“react-learning-demo”。

``` bash
npx create-react-app react-learning-demo
```

在新项目中，将 `react` 与 `react-dom` 包指向 facebook/react 下我们刚才生成的包。

``` bash
# 将项目内的 react react-dom 指向之前申明的包
➜  react-learning-demo git:(master) yarn link react react-dom
yarn link v1.22.10
success Using linked package for "react".
success Using linked package for "react-dom".
✨  Done in 0.12s.
```

现在试试在 react/build/node_modules/react-dom/cjs/react-dom.development.js 中随意打印些东西。

在 `react-learning-demo` 项目下执行 `yarn start`，现在浏览器控制台已经可以打印出我们输入的东西了。

通过以上方法，我们的运行时代码就和 React 最新代码一致了。
