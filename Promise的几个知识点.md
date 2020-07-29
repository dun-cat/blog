## Promise的几个知识点 
### 使用

让多个异步任务的代码更加优雅简洁；
统一异步任务接口规范，是一种抽象；

#### 以前

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
#### 现在

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

### Promise.all()

接收一个 promise 对象的数组作为参数，当这个数组里的所有 promise 对象全部变为 resolve 或 reject 状态的时候，它才会去调用 .then 方法。如上例所示。


###  Promise.race()

Promise.all 在接收到的所有的对象 promise 都变为 FulFilled 或者 Rejected 状态之后才会继续进行后面的处理， 与之相对的是 Promise.race 只要有一个 promise 对象进入 FulFilled 或者 Rejected 状态的话，就会继续进行后面的处理。

### Promise.resolve() & Promise.reject()

直接返回一个 promise 的实例，和下面的代码功能一致：

``` javascript
new Promise(function(resolve) {
	resolve(42)
})

Promise.resolve(42) // 快捷方式

new Promise(function(resolve, reject){
	reject(new Error("some errors"))
})

Promise.reject(new Error("some errors")) // 快捷方式
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

### 任务的顺序执行

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