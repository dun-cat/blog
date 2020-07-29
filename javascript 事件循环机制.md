## javascript 事件循环机制 
### 简介

javascript 运行在单线程上。该线程除了用业务逻辑的执行，也用作操作DOM，即UI的渲染。

### 基本执行示意图

[![](/img/event-loop.png)](/img/event-loop.png)

### 阐述

Javascript 中单线程实现异步的设计思想，比较简单。提供一个队列和暴露外部的api，每当遇到异步任务就通过 api 加入队列，直到执行栈里无执行任务时，把队列里的任务取出来执行便可。

这有个在线可视化工具，用于帮助理解事件循环机制以及概念， 工具也是开源的。
[http://latentflip.com/loupe](http://latentflip.com/loupe)

#### (执行栈) Call Stack

用于执行代码，并且是以栈的方式存储。

#### (事件循环)Event Loop

用于提取任务队列里的任务，然后把其加入执行栈执行。

#### (任务队列)Task Queue

Task Queue 也被称为 `Macro Task`，有些地方的示意图还会以`Callback Queue`表示，当代码中包含异步任务的时候，这些任务就会进入队列。所谓的任务其实就是事件的回调函数。

例如：WebApi 里的 onClick，需要绑定一个函数，那么这个函数在用户出发单击事件时，就会进入任务队列。

这类任务 WebApi 提供的接口有以下一些：

*   setTimeout
*   setInterval
*   setImmediate
*   requestAnimationFrame
*   I/O
*   UI rendering

#### (小任务队列)MicroTask Queue

该队列作用和上面一样，但是执行的优先级却不一样。在同一个执行上下文中，Microtask 的优先级要`大于` Macrotask。

这类任务 WebApi 提供的接口有以下一些:
- process.nextTick
- Promises
- Object.observe
- MutationObserver

扩展阅读：

\> [https://developer.mozilla.org/en-US/docs/Web/JavaScript/EventLoop](https://developer.mozilla.org/en-US/docs/Web/JavaScript/EventLoop)

\> [https://www.youtube.com/watch?v=8aGhZQkoFbQ](https://www.youtube.com/watch?v=8aGhZQkoFbQ)

\> [https://www.jianshu.com/p/d3ee32538b53](https://www.jianshu.com/p/d3ee32538b53)