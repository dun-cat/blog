## Javascript 的模块化 
### 简介

现代软件开发，[模块化编程](https://zh.wikipedia.org/wiki/模块化编程)是一个必然的演进历程。在前端领域模块化编程也是目前研发的主流编程范式。

### 模块化

通过模块化可以有以下一些收益：

* 通过把功能划分到各个模块之后，每个模块都是`独立的`，通常一个模块有自己的功能和职责，能够方便让开发者理解并使用；
* 划分模块后，会定义统一模块接口，来对外暴露功能。这使得所有开发者定义模块和引用模块的方式都是`一致的`；
* 模块化后，通用的功能的模块在开发者直接来回共享，可以在不同项目进行`复用`。

当所有开发者使用一致的模块标准规范后，模块共享及使用变得非常便利，这也利于开源软件的发展。在 JavasScript 语言中，模块化在低层次同时解决了全局命名空间被污染的历史问题。

JavaScript 的模块有一些特点：

* 模块无需直接引用全局对象，全局对象可以作为模块依赖项引入，这也是标准做法；
* 模块会显式列出其依赖项并获取这些依赖项的`句柄` (Handle)，在标准模块开发时，通常把依赖引入放在文件的开头；
* 模块通常有两个必要实现：一个是`定义模块接口`，另一个是`引用模块接口`。这两个接口成对出现。引用的模块，通常也由定义模块接口定义的；
* 模块的颗粒度可以是任意大小，这非常适合组建健壮的大型应用。

### RequireJS

在前端制定模块系统标准之前，已经有一些优秀模块库实现了模块化的功能。例如在浏览器环境下的 [RequireJS](https://requirejs.org/)，也被叫做`模块加载器`，它是一款非常经典模块系统。

> RequireJS 也支持在其它环境，例如：Rhino 和 Node。不过，浏览器是它的初始应用。

#### 模块加载

模块化的加载与传统的 `<script>` 标签加载 JavaScript 文件不同，模块化记载鼓励使用`模块 ID` 来加载 JavaScript 文件而不是使用 script 标记的 URL。

JavaScript 文件的路径会被提前或动态建立起和自身为模块的 ID 映射。开发者使用模块，只需关注模块 ID 而无需关注该模块在磁盘的具体位置。

当然通过模块 ID 来加载 JavaScript 文件，并不意味着一个模块 ID 对应一个 JavaScript 文件，通常会通过`优化工具`将多个模块打包成单一 JavaScript 文件。

假设我们有一个如下文件结构的项目：

``` text
.
├── app
│   └── sub.js
├── entry.js
├── index.html
└── lib
    └── require.js
```

`app/sub.js` 的内容如下：

``` js
define({
  color: 'red'
});
```

浏览器环境中，我们需要在 `index.html` 中优先引入 `RequireJS` 文件：

``` html
<script data-main="entry.js" src="lib/require.js"></script>
```

`entry.js` 是加载 `lib/require.js` 完成后的`启动脚本`。我们的全局配置可以在 `entry.js` 中完成。

在 entry.js 中：

``` js
requirejs.config({
  // 默认任意模块 ID 都从 lib 目录中加载  
  baseUrl: 'lib',
  // 此外，如果模块 ID 以 "app" 开头，则会从 app 目录加载
  // paths 配置相对于 baseUrl 的路径规则，同时不要包含文件后缀 (.js)，因为 paths 的配置支持指向目录
  paths: {
    app: '../app'
  }
});

// 开始执行应用逻辑
require(['app/sub'], function (sub) {
  // 当 app/sub 模块加载完成之后，当前函数被执行
  console.log(sub)
  // output: {color: 'red'}
});
```

RequireJS 为了加载速度对依赖的`加载都是无序的`，但一定是在他们加载完成之后，再执行回调函数。RequireJS 保证我们使用依赖时一定是正确的。

##### 加载远程模块

除了通过路径引用模块外，可以加载远程模块：

``` js
require.config({
  paths: { 
    dayjs: 'https://lf6-cdn-tos.bytecdntp.com/cdn/expire-1-M/dayjs/1.10.8/dayjs.min' 
  }
});

require(['dayjs'], function(dayjs) {
  console.log(dayjs().format("YYYY-MM-DD HH:mm:ss"));
  // output: 2023-03-21 21:41:30
});
```

但是你**不能**以下面的方式去加载：

``` js
// ⚠️ 不要这么去做
require(['https://lf6-cdn-tos.bytecdntp.com/cdn/expire-1-M/dayjs/1.10.8/dayjs.min'], function (dayjs) {});
```

#### 模块定义

在上面的`app/sub.js`文件，我们已经看到了模块的定义方式。RequireJS 通过 `define` 函数来定义模块，最简单的`值模块`定义如下：

``` js
define({
  id: 0,
  color: "black",
  size: "unisize"
});
```

该模块返回了一个对象值，也可以定义一个`函数模块`:

``` js
define(function() {
  return function(title) {
    window.title = title;
  } 
});
```

也可以加上依赖项：

``` js
define("app/goods", ["app/my/cart", "app/my/inventory"], function(cart, inventory) {
  const goods = {
    id: 0,
    color: "black",
    size: "unisize",
    addToCart: function() {
      inventory.decrement(this);
      cart.add(this);
    }
  }
  return goods;
});
```

我们定义了一个`具名`的模块 `app/goods`，`app/my/cart` 和 `app/my/inventory` 是当前模块所需依赖项数组。最后，提供了一个回调函数，其参数为依赖项提供的对象，并和依赖数组项`顺序一致`。模块的回调函数会在两个依赖加载完成之后执行。

> 通常不会定义具名模块，因为这样会损失`可移植性`。模块加载的时候默认根据路径匹配到模块文件的磁盘位置。

也可以把 `app/goods.js` 文件定义为一个匿名模块：

``` js
define(["app/my/cart", "app/my/inventory"], function(cart, inventory) {
  const goods = {
    id: 0,
    color: "black",
    size: "unisize",
    addToCart: function() {
      inventory.decrement(this);
      cart.add(this);
    }
  }
  return goods;
});
```

现在它的文件结构如下：

``` text
.
├── app
│   ├── goods.js
│   ├── my
│   │   ├── cart.js
│   │   └── inventory.js
│   └── sub.js
├── entry.js
├── index.html
└── lib
    └── require.js
```

从 RequireJS 支持 [CommmonJS 模块](#commonjs-模块)方式，因此，无需提供依赖数组，可以直接在包裹的`define`里，通过 `require` 函数引入依赖：

``` js
define(function (require, exports) {
  const myCart = require('app/my/cart');
  const myInventory = require('app/my/inventory');
  return {
    id: 0,
    color: "black",
    size: "unisize",
    addToCart: function () {
      myInventory.decrement(this);
      myCart.add(this);
    }
  }
});
```

#### 模块引用

在上面已经看到了引用模块接口的实现，RequireJS 通过 `require` 函数来引入模块：

``` js
require(["app/goods"], function(goods) {
  console.log(goods)
  // output: {id: 0, addToCart: f }
});
```

如果我们没有指定 `baseUrl` 和 `paths` 配置，那么依赖模块 `app/goods` 会被解析到 `lib/app/title.js` 的错误文件路径。

#### 循环依赖

在软件设计上来说，`循环依赖` (Circular dependency) 是一种[反模式 (anti-pattern)](https://en.wikipedia.org/wiki/Anti-pattern)。可能导致阻止自动垃圾收集器释放内存。尽管如此，模块一旦很多之后，模块之间的相互引用可能不可避免的发生。通常这种情况，模块系统都有相应的策略来应对。

![javascript-module.svg](javascript-module.svg)

在 RequireJS 中，`A` 模块引用了 `B` 模块，而 `B` 模块也需要依赖 `A` 模块。那么 `B` 模块可以这么去定义：

``` js
// b.js
define(["require", "a"], function(require, a) {
  // 如果 a 被 b 引用。此时，a 的值为 null
  return function(title) {
    return require("a").doSomething();
  }
});
```

让我们再来看看 `A` 模块的定义：

``` js
// a.js
define(["b"], function(bFunc) {
  bFunc("page title");
  return {
    doSomething() {}
  }
});
```

相比非循环依赖的加载，在 `B` 模块中使用 `A` 模块的方法，需要通过 `require` 来主动加载 `A` 模块。虽然解决了循环依赖的问题，但其实只是针对某一模块来解决的，并非一致的。

我们更希望不用确认哪个模块，都使用统一方式来解决循环依赖的问题，同时它也是一种模块定义方式。所以，我们可以使用 CommmonJS 模块方式：

``` js
// b.js
define(function(require, exports, module) {
    // 如果 a 使用 exports ，那么我们在这里有一个真实对象引用。
    // 然后，我们不能使用任何 a 的属性，直到 b 返回一个值
    const a = require("a");
    exports.foo = function () {
        return a.doSomething();
    };
});
```

`exports` 为模块创建一个`空对象`，该对象可`立即`供其他模块引用。同样 `A` 模块也可以使用同样的方式来定义，这种方式和 CommonJS 标准模块已经非常的相似了。

你可以在[这里](https://github.com/dun-cat/requirejs-learning)找到上面的示例。

### CommonJS 模块

CommonJS 是模块的一种规范，它定义了一种模块格式。在前端领域使用最广的是 Node.js 环境下的对它的实现。

参考资料：

\> [https://zh.wikipedia.org/wiki/模块化编程](https://zh.wikipedia.org/wiki/模块化编程)

\> [https://vibaike.com/109683/?ivk_sa=1024320u](https://vibaike.com/109683/?ivk_sa=1024320u)

\> [www.adequatelygood.com/JavaScript-Module-Pattern-In-Depth.html](www.adequatelygood.com/JavaScript-Module-Pattern-In-Depth.html)
