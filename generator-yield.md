---
layout: post
title: Generator & Yield
date: 2019-06-19
tags: ["Javascript","javascript","异步编程"]
---

### 简介
Generator 和 Yield 是 ES6 新引入的异步编程概念。
### 术语
生成器函数(Generator Function)：通过`function*` 可以定义一个生成器函数；
生成器对象(Generaor Object)：生成器函数的返回一个生成器对象；
yield 关键字：返回一个 IteratorResult 对象；
#### 简单实例

```javascript
function* foo() {
  let count = 0;
	yield count;
	yield count + 2;
	return count;
}
let iterator = foo();
iterator.next();
// {value: 0, done: false}
iterator.next();
// {value: 2, done: false}
iterator.next();
// {value: 0, done: true}
```
> foo() 返回一个符合[迭代器协议](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Iteration_protocols#iterator)的生成器对象(Generator Object)，也可以说这个对象是一个迭代器。

扩展阅读：

\> [https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/yield](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/yield)

\> [https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/function*](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/function*)

\> [https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Iteration_protocols#iterator](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Iteration_protocols#iterator)