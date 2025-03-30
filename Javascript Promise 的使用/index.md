## Javascript Promise 的使用 
### 简介

让多个异步任务的代码更加优雅简洁；
统一异步任务接口规范，是一种抽象；

以前：

```javascript
function foo(callback) {
 // do some async task ...
 callback(error, response)
}
function bar(callback) {
 // do some async task ...
 callback(error, response)
}

foo(function(error, response) {
 if(error) {
  // handle error
 } else {
  bar(function(bar_error, bar_response) {})
 }
})
```

现在：

```javascript
var foo = function() {
 return new Promise(function(resolve, reject) {
  // do some async task ...
  if(error)
   reject(error)
  else
   resolve(response)
 })
}

var bar = function() {
 return new Promise(function(resolve, reject) {
  // do some async task ...
  if(error)
   reject(error)
  else
   resolve(response)
 })
}

foo().then(bar).then(function(result) {}).catch(error) {} // 顺序执行
// or
Promise.all([foo(), bar()]).then(function(result) {}).catch(error) {} // 同时执行，所有任务完成后执行then
```

### 静态方法

#### Promise.all()

接受一个`迭代器对象`作为输入，例如 Array 或 String。

满足以下`任意一个条件`就会继续进行后面的处理：

* `所有`异步任务 resolve，返回`所有兑现值`组成的数组；
  
  ```ts
  const p1 = Promise.resolve(3);
  const p2 = 1337;
  const p3 = new Promise((resolve, reject) => {
    setTimeout(() => {
      resolve("foo");
    }, 100);
  });

  Promise.all([p1, p2, p3]).then((values) => {
    console.log(values); // [3, 1337, "foo"]
  });
  ```
  
* `任意一个` reject，返回`第一个`被拒绝的对象的原因。
  
  ``` ts
  const p = new Promise((resolve, reject) => {
    reject(new Error("拒绝"));
  });
  Promise.all([1, 2, 3, p]).catch((error) => {
    console.error(error.message); // "拒绝"
  });
  ```

#### Promise.allSettled()

接受一个 `Promise 可迭代对象`作为输入。

`所有`异步任务 reject 或 resolve，都通过 `then()` 返回兑现值或原因。Promise.allSettled `不关心` Promise 对象的结果状态。

``` ts
Promise.allSettled([
  Promise.resolve(33),
  new Promise((resolve) => setTimeout(() => resolve(66), 0)),
  99,
  Promise.reject(new Error("一个错误")),
]).then((values) => console.log(values));

// [
//   { status: 'fulfilled', value: 33 },
//   { status: 'fulfilled', value: 66 },
//   { status: 'fulfilled', value: 99 },
//   { status: 'rejected', reason: Error: 一个错误 }
// ]
```

#### Promise.any()

接受一个 `Promise 可迭代对象`作为参数。

满足以下`任意一个条件`就会继续进行后面的处理：

* `任意一个`异步任务 `resolve`，返回`第一个` resolve Promise 对象的兑现值。
  
  ``` ts
  const p1 =new Promise((resolve, reject) => {
    setTimeout(() => {
      resolve('p1');
    }, 100);
  });
  const p2 =new Promise((resolve, reject) => {
    setTimeout(() => {
      resolve('p2');
    }, 200);
  });

  Promise.any([p1,p2]).then(value => console.log(value));
  // p1
  ```

* `所有`异步任务被 `reject`，返回一个包含所有 rejected 原因的 [AggregateError](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/AggregateError) 对象。
  
  ```ts
  const p1 =new Promise((resolve, reject) => {
    setTimeout(() => {
      reject('p1 error');
    }, 100);
  });
  const p2 =new Promise((resolve, reject) => {
    setTimeout(() => {
      reject('p2 error');
    }, 200);
  });

  Promise.any([p1,p2]).catch(e => console.log(e.errors));
  // ['p1 error', 'p2 error']
  ```

#### Promise.race()

接受一个 `Promise 可迭代对象`作为输入。

`任意一个`异步任务 reject 或 resolve，返回`第一个完成`异步任务的兑现值或原因。Promise.race `不关心` Promise 对象的结果状态。

#### Promise.resolve()

直接返回一个 promise 的实例，和下面的代码功能一致：

``` ts
new Promise(function(resolve) {
 resolve(42)
})

Promise.resolve(42) // 快捷方式
```

#### Promise.reject()

``` ts
new Promise(function(resolve, reject){
 reject(new Error("some errors"))
})

Promise.reject(new Error("some errors")) // 快捷方式
```

#### Promise.withResolvers()

等同：

``` ts
let resolve, reject;
const promise = new Promise((res, rej) => {
  resolve = res;
  reject = rej;
});
```



### 异常处理

#### catch

catch 用来截获异常，方式如下：

``` javascript
function foo () {
 return new Promise(function(resolve, reject) {
  reject()
 })
}
function handleError(error) {}
foo().then(function() {}).catch(handleError)
```
#### then

也可以在 then 的第二个回调函数截获异常信息

``` javascript
function foo () {
 return new Promise(function(resolve, reject) {
  reject()
 })
}
function handleError(error) {}
foo().then(function() {}, handleError)
```

#### catch 和 .then 的区别

当 promise 调用 resolve() 后，进入到 .then 的第一个回调函数。此时，若在此函数抛出异常，并不会被 .then 的第二回调函数截获异常。而 .catch 则可以。

``` javascript
function throwError() {
 throw new Error('error info')
}
function handleError(error) {}
Promise.resolve().then(throwError, handleError) // handleError 并不会截获抛出的异常
Promise.resolve().then(throwError).catch(handleError) // 异常信息会被 catch 截获

```

### 异步任务的同步执行

开头的例子使用 then 的方式也是顺序执行，但任务比较多的时候，这种写法就不够优雅，所以我们通过遍历来实现。

``` javascript
var foo = function() {
 return Promise.resolve(10)
}
var bar = function(preResponse) {
 console.log(preResponse)
 preResponse ++
 return Promise.resolve(preResponse)
}
function execTasks(tasks) {
 return tasks.reduce(function(promise, task) {
  return promise.then(task)
 }, Promise.resolve())
}
execTasks([foo, bar]).then(function(response) {
 console.log(response)
})

```

借用数组的 reduce 方法，迭代 promise 的值，传入 execTasks 是一个数组的。

参考资料：

\> [https://cdn.xgqfrms.xyz/promise/understanding-javascript-promises.pdf](https://cdn.xgqfrms.xyz/promise/understanding-javascript-promises.pdf)