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
* 模块通常有两个必要接口：一个是`定义模块接口`，另一个是`模块导入导出接口`。导入导出接口一般成对出现。导入的模块，通常也只能使用模块导出接口指定的值；
* 模块的颗粒度可以是任意大小，这非常适合组建健壮的大型应用。

### RequireJS

在前端制定模块系统标准之前，已经有一些优秀模块库实现了模块化的功能。例如在浏览器环境下的 [RequireJS](https://requirejs.org/)，通常也被叫做`模块加载器`，它是一款非常经典的 [AMD 模块规范](https://github.com/amdjs/amdjs-api/blob/master/AMD.md)的实现库。

> RequireJS 也支持在其它环境，例如：Rhino 和 Node。不过，浏览器端是它的最为广泛的应用。

#### 模块加载

模块化的加载与传统的 `<script>` 标签加载 JavaScript 文件不同，模块化加载鼓励使用`模块 ID` 来加载 JavaScript 文件而不是使用 script 标记的 URL。

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

RequireJS 为了加载速度对依赖的`加载都是无序的`，正式的来说，他们都是`异步加载`的。但回调函数一定是在他们加载完成之后再执行的，RequireJS 保证我们使用依赖时一定是正确的。

##### 如何加载模块？

RequireJS 把依赖模块文件处理成 script 标签，因此你可以在 header 里看到它：

``` html
<script type="text/javascript" charset="utf-8" async data-requirecontext="_" data-requiremodule="entry" src="./entry.js"></script>
<script type="text/javascript" charset="utf-8" async data-requirecontext="_" data-requiremodule="app/sub" src="lib/../app/sub.js"></script>
```

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

但是你**不能**以下面的方式去加载，它会抛出异常：

``` js
// ⚠️ 不要这么去做，它会抛出异常。
require(['https://lf6-cdn-tos.bytecdntp.com/cdn/expire-1-M/dayjs/1.10.8/dayjs.min'], function (dayjs) {});
```

#### 模块定义和导出

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

从 RequireJS 支持 [CommmonJS 模块](#commonjs-模块)方式，因此，无需提供依赖数组，可以直接在包裹的 `define` 里，通过 `require` 函数引入依赖：

``` js
define(function (require, exports, module) {
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

有一些 CommonJS 系统，主要是 Node，允许通过将导出值分配为 `module.exports` ，来设置模块对外暴露值。RequireJS 支持该习惯用法：

``` js
define(function (require, exports, module) {
  const myCart = require('app/my/cart');
  const myInventory = require('app/my/inventory');
  module.exports = {
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

这和直接 return 该对象是一样的效果。

#### 模块导入

在上面已经看到了导入模块接口的实现，RequireJS 通过 `require` 函数来导入模块：

``` js
require(["app/goods"], function(goods) {
  console.log(goods)
  // output: {id: 0, addToCart: f }
});
```

需要注意的是如果我们没有指定 `paths: { app: '../app'}` 配置，那么依赖模块 `app/goods` 会被解析到 `lib/app/goods.js` 的错误文件路径。

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

[CommonJS 模块](https://wiki.commonjs.org/wiki/Modules/1.1.1#Require)是的一种规范，它定义了一种模块格式。在前端领域使用最广的是 Node.js 环境下的对它的实现。

在 Node 环境中，我们**无需**对文件使用类似 RequireJS 的 `define` 函数包裹模块，每个模块在执行之前，都会被一个[函数包裹器](https://nodejs.org/api/modules.html#the-module-wrapper)包裹，类似下面代码：

``` js
(function(exports, require, module, __filename, __dirname) {
// Module code actually lives in here
});
```

为此，Node 会做一些事情：

* 保持顶级变量 (var、const、let 定义的) 作用于模块范围，而不是 global 对象；
* 帮助提供一些全局搜索变量，这些变量可以指向到当前模块。例如：
  * `module` 和 `export` 对象，可以用于导出模块值；
  * `__filename` 和 `__dirname` 变量包含模块的绝对路径文件名称和目录路径。

如果打印 module，你会看到它的全部属性：

``` js
Module {
  id: '/Users/lumin/Desktop/tmp/commonjs/a.js',
  path: '/Users/lumin/Desktop/tmp/commonjs',
  exports: {},
  filename: '/Users/lumin/Desktop/tmp/commonjs/a.js',
  loaded: false,
  children: [],
  paths: [
    '/Users/lumin/Desktop/tmp/commonjs/node_modules',
    '/Users/lumin/Desktop/tmp/node_modules',
    '/Users/lumin/Desktop/node_modules',
    '/Users/lumin/node_modules',
    '/Users/node_modules',
    '/node_modules'
  ]
}
```

其中 `exports` 为 `module` 对象的一个属性，所以下面的代码是一样：

``` js
exports = { greeting: 'hello' }
// same with
module.exports = { greeting: 'hello' }
```

如果在 Node 环境编写重写 RequireJS 下的 `app/goods.js` 代码会是下面的样子：

``` js
const myCart = require('./app/my/cart');
const myInventory = require('./app/my/inventory');

module.exports = {
  id: 0,
  color: "black",
  size: "unisize",
  addToCart: function () {
    myInventory.decrement(this);
    myCart.add(this);
  }
}
```

由于 Node 的[路径解析规则](https://nodejs.org/api/modules.html#all-together)，为了正确引入模块需要添加`./`来解析正确的路径。

#### 模块导出

上面了解到 CommonJS 无需开发者`手动定义模块`，即不需要模块包裹函数，只需要通过 `exports` 来导出模块。

``` js
module.exports = {}
// or
exports = {}
```

这里需要注意的是 `module.exports 默认是个空对象`，这意味着如果你给 `exports` 赋值，将`替换该空对象`，因此一个模块 `exports =` 只会出现一次。

#### 模块导入

Node 环境下，CommonJS 的模块导入语法如下：

``` js
const a = require("a-module");
const b = require("b-module");
a.doSomething();
b.doSomething();
```

可以看到相比 RequireJS 的模块导入，CommonJS 模块导入**没有回调**，这是 CommonJS 模块和 AMD 模块很大的一个区别，AMD 支持异步模块加载。

AMD 规范**强烈要求**模块是基于需要回调的需求，这是因为`动态计算`的`依赖项`可能会`异步加载`，所以上面可以看到 `define` 定义的模块都是通过回调方式来执行的。

#### 循环依赖

Node 允许循环依赖的存在，即便它是反模式的。

因为 exports 有默认值，因此如果存在循环依赖，也能立即返回一个空对象，不至于引发程序异常。下面提供了一个示例：

`a.js`：

``` js
console.log('a starting');
exports.done = false;
const b = require('./b.js');
console.log('in a, b.done = %j', b.done);
exports.done = true;
console.log('a done');
```

`b.js`：

``` js
console.log('b starting');
exports.done = false;
const a = require('./a.js');
console.log('in b, a.done = %j', a.done);
exports.done = true;
console.log('b done');
```

`main.js`：

``` js
console.log('main starting');
const a = require('./a.js');
const b = require('./b.js');
console.log('in main, a.done = %j, b.done = %j', a.done, b.done);
```

它的执行顺序如下：

1. `main.js` 加载时 `a.js`，则 `a.js` 依次加载 `b.js`；
2. `b.js` 尝试加载 `a.js`。 为了防止无限循环，将导出对象的**未完成副本** `a.js` 返回给 `b.js` 模块。
3. `b.js` 然后完成加载，并将其 `exports` 对象提供给 `a.js` 模块。

到 `main.js` 加载两个模块时，它们都已完成。因此，该程序的输出将是：

``` shell
$ node main.js
main starting
a starting
b starting
in b, a.done = false
b done
in a, b.done = true
a done
in main, a.done = true, b.done = true
```

我们假设一个 `require()` 方法实现，它的实际执行和 Node require 方法非常相似，你可以在[这里](https://github.com/nodejs/node/blob/f08655532b5bf06549000f5f7dd3c833c06b62c4/lib/internal/modules/cjs/loader.js#L948)看到其开源的执行源码。

在当调用 `require('./b.js')` 时，`b.js` 会如下执行：

``` js
function require(/* ... */) {
  const module = { exports: {}, loaded: false, /* ... */ };
 // 1. 如果当前模块没有加载完成，那么立即返回该对象
  if(!module.loaded) {
    return module.exports;
  }

 // 模块函数包裹器   
  ((module, exports) => {
    // 这里是 b.js 编写模块代码
    
    console.log('b starting');
    exports.done = false;
    // 由于 1 步骤，a.js 模块未加载完成，因此 a.js 直接返回了默认 module.exports 的值，此时它被绑定了 exports.done = false;
    
    const a = require('./a.js');
    // 所以，a.done 为 false
    console.log('in b, a.done = %j', a.done);
    exports.done = true;
    console.log('b done');
    
  })(module, module.exports);

  module.loaded = true;

  return module.exports;
}
```

Node 在实现 CommonJS 标准解决循环依赖的方式和 RequireJS 类似，通过直接返回一个空对象来避免执行异常而终止程序运行。这也侧面说明了即便循环依赖是反模式的，但 Node 依然允许循环依赖的存在。不过站在开发者角度来说，必须清晰认识到它的执行流程，否则容易出现难以发现的意外逻辑执行。

### AMD 模块

参考资料：

\> [https://zh.wikipedia.org/wiki/模块化编程](https://zh.wikipedia.org/wiki/模块化编程)

\> [https://vibaike.com/109683/?ivk_sa=1024320u](https://vibaike.com/109683/?ivk_sa=1024320u)

\> [www.adequatelygood.com/JavaScript-Module-Pattern-In-Depth.html](www.adequatelygood.com/JavaScript-Module-Pattern-In-Depth.html)

\> [https://reflectoring.io/nodejs-modules-imports/](https://reflectoring.io/nodejs-modules-imports/)
