## 响应式编程之 ReactiveX 
### 简介

[ReactiveX](https://reactivex.io/)，通过`流`（streams）、`可观察对象`（observables）和`运算符`（operators）完成对`响应式编程`的一种实现，具有多种语言实现，包括 RxJs、RxJava、.NET、RxPy 和 RxSwift。下面会以 RxJs 为蓝本进行讲解。

ReactiveX 是一种使用`可观察流`（observable streams）进行[异步编程](https://en.wikipedia.org/wiki/Asynchronous_programming)的 API 。`异步编程`允许程序员调用函数，然后在完成时让函数“回调”，通常是通过给函数提供另一个函数的地址以在完成时执行。

ReactiveX 上下文中的`可观察流`（即可以观察到的流）就像`事件发射器`（event emitters），可以发出三个事件：`下一个`（next）、`错误`（error）和`完成`（complete）。

一个`observable`直到它发出 error 事件或 complete 事件，才会发出 next 事件。但是，此时它不会再发出任何事件，除非它再次被订阅（subscribed）。

### 动作

动作（Motivation）

ReactiveX 结合了`观察者`和`迭代器`模式以及`函数式编程`的思想。

#### 可观察者和观察者

观察者（Observer）订阅一个可观察（Observable）序列，该序列通常通过调用提供的`回调函数`发送项目（Item）给 observer。

如果许多事件异步进入，它们`必须`存储在队列中或丢弃。在 ReactiveX 中，observer 永远不会被乱序的项目调用或（在多线程上下文中）在回调返回前一个项目之前调用。异步调用保持异步，可以通过返回一个 observable 来处理。

它类似于`迭代器模式`，如果发生致命错误，它会单独通知 observer（通过调用第二个函数）。当所有项目都发送完毕后，它就完成了（并通过调用第三个函数通知 observer）。Reactive Extensions API 还从其他编程语言的迭代器运算符中借用了许多运算符。

在 RxJs 中创建 observable：

``` js
let observable = Rx.Observable.create(function (observer) {
   observer.next();
});
```

#### 操作符

操作符（operator）本质上是一个`纯函数` (pure function)，它接收一个 Observable 作为输入，并生成一个新的 Observable 作为输出。它将一个 observable（`源`）作为其`第一个参数`并返回另一个 observable（`目标`或`外部`可观察对象）。

``` js
function myOperator(observable) {
  // ...
  return newObservable
}
```

在 RxJs 中，操作符是 Observable 类型上的方法，比如 `.map(...)`、`.filter(...)`、`.merge(...)`等等。

> 操作符是函数，它基于当前的 Observable 创建一个新的 Observable。这是一个无副作用的操作：前面的 Observable 保持不变。

你可以自定义一个操作符：

``` js
function counter(input) {
  let output = Rx.Observable.create(function subscribe(observer) {
    input.subscribe({
      next: (number) => observer.next(number + 1),
      error: (err) => observer.error(err),
      complete: () => observer.complete()
    });
  });
  return output;
}
```

使用该操作符：

``` js
// input 是个 observable
let input = Rx.Observable.from([1, 2, 3, 4]);
// output 是个新的 observable
let output = counter(input);

output.subscribe(x => console.log(x));
```

输出：

``` bash
2
3
4
5
```

然后对于源 observable 发出的每个项目，它会对该项目应用一个函数，然后在目标 Observable 上发出它。它甚至可以在目标 observable 上发出另一个 observable。这称为`内部可观察对象`。

一个发出内部 observable 的操作符后面可以跟着另一个运算符，它以某种方式组合所有内部 observable 发出的项目，并在其外部 observable 上发出项目。示例包括：

switchAll– 订阅每个新的内部可观察对象，一旦它发出并取消订阅前一个。
mergeAll– 在所有内部可观察对象发出时订阅它们，并以接收它们的任何顺序输出它们的值。
concatAll– 按顺序订阅每个内部 observable 并等待它完成，然后再订阅下一个 observable。
运算符可以链接在一起以创建复杂的数据流，这些数据流根据特定标准过滤事件。多个运算符可以应用于同一个 observable。
