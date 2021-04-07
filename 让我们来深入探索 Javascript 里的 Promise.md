## 让我们来深入探索 Javascript 里的 Promise 
### 介绍

在计算机科学中，`future`、`promise`、`delay`和`deferred`是指用于在某些并发编程语言中同步程序执行的构造。由于某些计算（或者网络请求）尚未结束，我们需要一个对象来代理这个未知的结果，于是就有了上述这些构造（future、promise等）。

Promise 一词在1976年就已提出，也有称之为 `eventual`，它的应用起源于[函数式编程](https://zh.wikipedia.org/wiki/函数式编程)和相关范例。

无论是函数式编程还是面向对象编程都是编程范式，即如何编程程序的方法论。我们拿`纯函数式`编程语言 haskell 举例，看看它的一些范式：

``` haskell

```

### Javascript 里的 Promise

[ECMAScript 2015 (6th Edition, ECMA-262) Promise](https://262.ecma-international.org/6.0/#sec-promise-objects)的标准定义是：“一个 Promise 是一个对象(object)，用于延时计算或异步计算的最终结果的占位”。

在 javascript 里创建 一个 Promise 如下：

``` javascript
const p = new Promise((resolutionFunc, rejectionFunc) => {
    resolutionFunc(777);
});
```

更多用法可以查看文章[Javascript Promise 的使用](/articles/promise-what-you-need-know)。

任何 Promise 对象都存在三个互斥状态：`fulfilled`、`rejected`、以及`pending`，即同时只存在其中一个状态。

假设有一个 Promise 实例 **p** ：

* 如果立即入列 `p.then(f, r)` 任务并且之后再调用 `f` 函数 ，那么 `p` 的状态是 `fulfilled`；
* 如果立即入列 `p.then(f, r)` 任务并且之后再调用 `r` 函数 ，那么 `p` 的状态是 `rejected`；
* 如果 `p` 的状态不是 `fulfilled` 也不是 `rejected`，那么标记为 `pending` 状态。

扩展阅读：

\> [https://zh.wikipedia.org/wiki/Future与promise](https://zh.wikipedia.org/wiki/Future与promise)

\> [https://262.ecma-international.org/6.0/#sec-promise-objects](https://262.ecma-international.org/6.0/#sec-promise-objects)

\> [https://github.com/zloirock/core-js/blob/master/packages/core-js/modules/es.promise.js](https://github.com/zloirock/core-js/blob/master/packages/core-js/modules/es.promise.js)

\> [https://promisesaplus.com/](https://promisesaplus.com/)

\> [https://zh.wikipedia.org/wiki/函数式编程](https://zh.wikipedia.org/wiki/函数式编程)

\> [https://zh.wikipedia.org/wiki/反面模式](https://zh.wikipedia.org/wiki/反面模式)