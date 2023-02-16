## 模块映射 (Import Maps)  
Web 孵化社区群组 (Web Incubator Community Group, `WICG` ) 在 2021 年 1 月份提交了一份有关`模块映射` (Import Maps) 标准的`草案` (Draft) 。你可以在[这里](https://wicg.github.io/import-maps/)看到草案内容。

该技术在 Chrome 的`89`桌面版本、Android、Webview 都已实现，你可以在[这里](https://www.chromestatus.com/feature/5315286962012160)看到相关描述，目前在较新的版本可以体验它的功能。

### 前置概念

#### 模块说明符 (Module specifiers)

浏览器在 import 声明中，只接受一种的`模块说明符` (module specifiers) ，它必须是一个完全限定的的 `URL` 或者由`/`、`./`或`../`起始的路径 (path) ，对于导入指定元素和模块的工作运行得还不错。

``` js
import './my-app-element.js'; // ./my-app-element.js 就是我们的模块说明符
```

下面都是有效的模块说明符：

* <https://example.com/apples.mjs>
* http:example.com\pears.js (转成 <http://example.com/pears.js>)
* //example.com/bananas
* ./strawberries.mjs.cgi
* ../lychees
* /limes.jsx
* data:text/javascript,export default 'grapes';
* blob:<https://whatwg.org/d0360e2f-caee-469f-9a2f-87d5b0456f6f>

然而，当你编写重用的组件，并且想要通过 npm 包 import 一个 peer 依赖时，路径便会发生变化。所以我们希望使用`命名的 import specifiers`的 node 风格采用`聚合引用` (polymer) 的方式。其实就是`命名化的模块`。

``` js
import {PolymerElement} from '@polymer/polymer/polymer-element.js';
```

> 这里的`@polymer/polymer`指的 npm 包名，这种风格的说明符有时被叫做`裸说明符` (bare specifier) 。

上面这种风格的说明符会通过一些 CLI 工具转换成`路径`引用的标准模式，而服务于浏览器环境。像 `polymer`、`webpack` 等构建工具的都能做到这点。

### 概念

Import maps 允许 web 页面去控制 Javascript 的 import 行为。

它的基本想法是想通过 `import 声明` 或者 `import() 表达式` 来控制引入的资源。允许使用`裸说明符`，让 `import moment from "moment"` 这样的语法能够正常的工作，当然 import map 远不止此。我们可能已经好几年习惯这样的写法，然后它却并未标准化。

### 定义

* 一个`解决结果` (resolution result) 不是一个 URL 就是 null。
* 一个`说明符 map`是一个有序的 map ，它是从 string 到 `解决结果`。
* 一个 `import map` 是一个包含以下两项的结构体：
  * `imports`，一个说明符 map；
  * `scopes`，一个有序 map ，它由 URLs 和 说明符 maps 组成。
* 一个`空 import map`是一个 import map，它的 imports 和 scopes 都是空的maps。

参考资料：

\> [https://github.com/WICG/import-maps](https://github.com/WICG/import-maps)

\> [https://wicg.github.io/import-maps/](https://wicg.github.io/import-maps/)

\> [https://polymer-library.polymer-project.org/3.0/docs/es6](https://polymer-library.polymer-project.org/3.0/docs/es6)

\> [https://html.spec.whatwg.org/multipage/webappapis.html#resolve-a-module-specifier](https://html.spec.whatwg.org/multipage/webappapis.html#resolve-a-module-specifier)
