## Functional Programing in Js 
### 纯函数

在计算机编程中，纯函数（pure function）有需要同时满足以下两种属性：

1. 对于`恒等`（identical）的参数（不发生变化的局部静态变量、非局部变量、可变引用参数或 input 流），函数返回值也是恒等的，并且。
2. 函数应用`不会`产生`副作用`（side effects）。不会产生副作用指：局部静态变量、非局部变量、可变引用参数或 I/O 流没有`突变`（mutation）。

#### 非纯函数示例

下面这些函数是不纯（impure）的，因为他们不满足纯函数第一种属性。

1.非局部变量的返回值变化。

``` ts
let a = 0;
function foo() {
  return a;
}
```

* 不可变性
* 记忆化
* 纯函数
* 高阶函数
* 管道和过滤器
* 闭包
* λ 演算

参考文献：

\> [https://en.wikipedia.org/wiki/Pure_function](https://en.wikipedia.org/wiki/Pure_function)

\> [https://github.com/getify/Functional-Light-JS](https://github.com/getify/Functional-Light-JS)
