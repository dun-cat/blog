## React Native 1 Introduction 
### React Native

React Native 是一种实现跨端技术框架。与[cordova](https://cordova.apache.org/)（前身：PhoneGap）这种在`Webview` 中嵌套`网页 App`的跨端不同。React Native 最终提供给用户的视图是`原生视图`，这让用户能体验到原生应用的感觉。

### 开发成本

React Native 遵循 Write once, run anywhere 的宗旨，让一套代码同时运行在多个端（android、ios、windows），这极大的提高了研发效率。同时，由于代码可以动态下发至 App 运行，所以可以通过热更新来提高迭代频率。但这种技术的应用也会带来额外的前期成本。

#### 对原生端技术的了解

React Native 虽然是一种跨端技术，随着应用的深入开发，它依然需要前端开发人员了解移动端研发技术，通常是`android`和`ios`两端。

> 虽然 React Native 也在支持 Windows 平台，但目前国内市场不常见。

很多时候单纯前端研发人员并不能完成原生组件或功能开发，Native 端同学的介入是**必须的**，以往经历告诉我前端和 Native 端同学需要紧密合作。

一个包含 Native 功能的 Node 包至少包含了`android`和`ios`的原生功能实现源码，遇到问题时，前端研发同学通常需要优先去解决问题。问题的定位有时候极为困难，因为报错信息会毫无头绪。

#### 初期研发环境的搭建

React Native 的初期环境搭建体验并不是很好，有几方面的原因。
