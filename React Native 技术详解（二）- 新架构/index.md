## React Native 技术详解（二）- 新架构 
在[React Native 技术详解（一）- 认识它](../react-native-1-introduction/#react-native-架构及核心生态)中，我们知道 React Native 以前的架构图如下：

![react-native-architecture](react-native-architecture.svg)

React Native 以 React 技术为开发基础，通过 Metro 捆绑器打包成最终目标代码文件 JSBundle。JSBundle 运行在 JavaScriptCore 执行引擎，通过 Bridge 传递布局及相关渲染数据。最后，由 Yoga 进行与 Native UI 模块管理布局和渲染的工作。

新的架构图如下：

新的架构有以下几个新的概念：`JSI`、`Fabric`、`Turbo Modules`、`CodeGen`，这些都将至我们必须了解的，下面都将逐一解释。

### 概念

#### JSI

JSI（JavaScript Interface）将取代老架构中的 Bridge，它为 JS 执行引擎提供 API，使 JS 直接感知原生函数和对象。

JSI 带来的一个优势是 `JS 线程`和`Native Modules`的`完全同步`。在 JSI 的帮助下，JS 将能够保存对热对象的引用并调用它们的方法。它还会附带共享所有权的概念，允许**原生端**直接与**JS 线程**通信。

#### Fabric

Fabric 是负责 Native 端的`UIManager`的新名称。现在最大的不同是它不再通过桥与 JS 端通信，而是使用 JSI 暴露一个 Native 函数，因此 JS 端可以直接通过 ref 函数进行通信，反之亦然。更好、更高效的性能以及在双方之间传递数据。

#### Turbo Modules

`Turbo Modules`的目的与老架构的`Native Modules`相同，但实现和行为不同。首先，它们是`懒加载`（lazy-loaded）的，这意味着它只在应用程序`需要它们时加载`，而不是在启动时加载所有它们。此外，它们还使用 JSI API，JS 持有一个引用以在 React Native JS 库端使用它们，从而获得更好的性能，尤其是在启动时间上。

#### CodeGen

为了确保 React Native 和 Native 部分之间的无缝通信，Facebook 的团队目前正在开发名为 CodeGen 的工具。期望能够自动实现两个线程的兼容性并`使它们同步`。该生成器将定义`TurboModules`和`Fabric`所需的接口元素，并自信地将消息发送到领域。此升级将消除为两个线程复制代码的需要，并有望确保顺利同步。

参考文献：

\> [https://litslink.com/blog/new-react-native-architecture](https://litslink.com/blog/new-react-native-architecture)

\> [https://medium.com/mindful-engineering/fabric-architecture-react-native-a4f5fd96b6d2](https://medium.com/mindful-engineering/fabric-architecture-react-native-a4f5fd96b6d2)