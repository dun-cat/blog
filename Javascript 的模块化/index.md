## Javascript 的模块化 
### 简介

现代软件开发，[模块化编程](https://zh.wikipedia.org/wiki/模块化编程)是一个必然的演进历程。在前端领域模块化编程也是目前研发的主流编程范式。

### 模块化的目的

通过模块化可以有以下一些收益：

* 通过把功能划分到各个模块之后，每个模块都是`独立的`，通常一个模块有自己的功能和职责，能够方便让开发者理解并使用；
* 划分模块后，会定义统一模块接口，来对外暴露功能。这使得所有开发者定义模块和引用模块的方式都是`一致的`；
* 模块化后，通用的功能的模块在开发者直接来回共享，可以在不同项目进行`复用`。

当全世界的人们使用一致的模块标准规范后，模块共享及使用变得非常便利，这也利于开源软件的发展。在 JavasScript 语言中，模块化在低层次同时解决了全局变量被污染的 (意外覆盖) 历史问题。

### 模块化标准之前

模块通常有两个必要实现：一个是`定义模块接口`，另一个是`引用模块接口`。这两个接口成对出现。引用的模块，也必须由模块接口定义的。

在前端制定模块系统标准之前，已经有一些优秀模块库实现了模块化的功能。例如在浏览器环境下的 [RequireJS](https://requirejs.org/)。

> RequireJS 也支持在其它环境，例如：Rhino 和 Node。不过，浏览器是它的初始应用。

以下是它的模块定义：

``` js
  define("foo/title",
      ["my/cart", "my/inventory"],
      function(cart, inventory) {
          return "hello world"
      }
  );
```

RequireJS 通过 `define` 函数来定义模块。`foo/title` 是定义模块的名称，`my/cart` 和 `my/inventory` 是当前模块所需依赖模块，以数组形式传入。最后，提供了一个回调函数，其参数为依赖模块暴露的数据，并且 `foo/title` 模块提供了数据 `hello world`。

模块的回调函数会在两个依赖加载完成之后执行。

从上面我们可以看到，模块的唯一标识其实就是`路径+名称`，对应到本地文件结构如下：

``` text
.
├── foo
│   └── title.js
└── my
    ├── cart.js
    └── inventory.js
```

同样它有一个引用模块接口的实现：

``` js
requirejs(["foo/title"], function(title) {
    // title 值为 hello world
});
```

#### 循环引用

模块一旦很多之后，难保存在模块之间的相互引用。

参考资料：

\> [https://zh.wikipedia.org/wiki/模块化编程](https://zh.wikipedia.org/wiki/模块化编程)

\> [https://vibaike.com/109683/?ivk_sa=1024320u](https://vibaike.com/109683/?ivk_sa=1024320u)
