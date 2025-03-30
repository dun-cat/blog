## Javascript 中的函数式编程 
从一开始，函数就是 JavaScript 世界中的`一等公民` (first-class citizens) ，即`函数`可以作为别的函数的`参数`、函数的`返回值`，`赋值给变量`或`存储在数据结构`中，他们可以当做函数类型的普通变量来对待。

> 1960年代中期，[克里斯托弗·斯特雷奇](https://zh.m.wikipedia.org/zh-hans/克里斯托弗·斯特雷奇)在 **functions as first-class citizens** 中提出这一概念。

函数不是头等公民的程序设计语言可以使用`函数指针`或`委托` (delegate) ，实现函数作为参数。

### 纯函数

纯函数 (pure function) 是函数式编程中的原子构建块。它们因其`简单性`和`可测试性`而备受推崇。

在计算机编程中，纯函数有需要同时满足以下两种属性：

1. 对于`恒等` (identical) 的参数 (不发生变化的局部静态变量、非局部变量、可变引用参数或 input 流) ，函数返回值也是恒等的，并且；
2. 函数应用`不会`产生`副作用` (side effects) 。不会产生副作用指：局部静态变量、非局部变量、可变引用参数或 I/O 流没有`异变` (mutation) 。

在 JavaScript 中通俗的副作用可以有以下一些操作：

* 全局更改变量、属性或数据结构；
* 更改函数参数的原始值；
* 处理用户输入；
* 抛出异常，除非它在同一个函数中被捕获；
* 打印到屏幕或记录；
* 查询 HTML 文档、浏览器 cookie 或数据库。

以上是来自 wiki 的定义，我们也可以使用以下的一些容易理解的特性来定义纯函数：

1. 纯函数不会改变外部状态；
2. 无论函数应用了多少次，每次输入 (input) 相同时，它的输出 (output) 也一定是相同的；
3. 纯函数都是[引用透明](https://en.wikipedia.org/wiki/Referential_transparency) (referential transparency) 的，函数是可以被它的输出所取代，而不影响程序的行为。

> 函数式编程的一个定义特征是它只允许`引用透明`的函数，引用透明性是定义纯函数的一种更正式的方式。

一个简单的纯函数如下：

``` ts
function add(a, b) {
  return a + b
}
```

#### 非纯函数示例

下面这些函数是不纯 (impure) 的。

1\. `console.log()` 改变了函数外部的应用状态：

``` ts
function add(a, b) {
  const total = a + b
  console.log(total)
}
```

2\. 相同的输入，每次输出是不同的：

``` ts
function add(a, b) {
  const total = a + b + Math.random()
  return total
}
```

3\. 以下是引用非透明的函数，它的输出并非完全取决于它的输入：

``` js
let g = 0;
function foo(x) {
  g++;
  return x + g;
}
```

4\. 可变引用参数的改变，改变了函数外部程序状态：

``` ts
const impureAssoc = (key, value, object) => {
  object[key] = value;
};

const person = {
  name: 'Bobo'
};

const result = impureAssoc('shoeSize', 400, person);

console.log({
  person,
  result
});
```

不过我们可以净化它，使它成为一个纯函数：

``` ts
const pureAssoc = (key, value, object) => {
  const newObject = JSON.parse(JSON.stringify(object));

  newObject[key] = value;

  return newObject;
};

const person = {
  name: 'Bobo'
};

const result = pureAssoc('shoeSize', 400, person);

console.log({
  person,
  result
});
```

改变你的输入可能很危险，但改变它的副本是没有问题的。我们的最终结果仍然是一个可测试、可预测的函数，无论何时何地调用它都可以工作。

注意这里是深度克隆，这可能在复杂的引用类型结构引发性能问题。所以有些库会通过`结构共享`来优化它。

### 记忆化

`记忆化` (memoization) 是一种通用的函数式编程技术，它适用于任何纯函数，如果使用相同的参数调用它总是会产生相同的结果。所以只有`引用透明`的函数才能被记忆化

记忆一个函数会产生一个`新函数`，该函数将在内部检查先前计算值的缓存，如果在那里找到所需的结果，它将被返回而无需任何进一步的工作。

如果在缓存中没有找到该值，则所有需要的工作都将完成，但在返回给调用者之前，结果将存储在缓存中，以便将来使用来电。

可以通过下面函数来实现一个记忆化的函数：

``` ts
const memoize = (fn) => {
  const cache = new Map();
  return (...args) => {
    const strX = JSON.stringify(args);
    if (!cache.has(strX)) {
      cache.set(strX, fn(...args));
    }
    return cache.get(strX);
  };
};
```

可以再优化下，用于接收上下文：

``` ts
const memoize = function(fn, context) {
  const cache = new Map();
  return function(...args) {
    const strX = JSON.stringify(args);
    if (!cache.has(strX)) {
      cache.set(strX, fn.call(context || this, ...args));
    }
    return cache.get(strX);
  };
};
```

### 高阶函数

在数学和计算机科学中，高阶函数(Higher-order function) 是至少满足以下`条件之一`的函数：

* 将`一个`或`多个`函数作为参数；
* 返回一个函数作为其结果。

* 不可变性
* 记忆化
* 纯函数
* 高阶函数
* 管道和过滤器
* 闭包
* λ 演算

参考资料：

\> [https://en.wikipedia.org/wiki/Pure_function](https://en.wikipedia.org/wiki/Pure_function)

\> [https://github.com/getify/Functional-Light-JS](https://github.com/getify/Functional-Light-JS)
