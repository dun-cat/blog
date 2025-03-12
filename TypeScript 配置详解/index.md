## TypeScript 配置详解 
### 简介

当目录中出现了 `tsconfig.json` 文件，则说明该目录是 TypeScript 项目的`根目录`。tsconfig.json 文件指定了编译项目所需的根目录下的文件以及编译选项。

JavaScript 项目可以使用 `jsconfig.json` 文件，它的作用与 `tsconfig.json` 基本相同，只是默认启用了一些 JavaScript 相关的编译选项。

### 使用

* 在调用 tsc 命令并且没有其它输入文件参数时，编译器将由当前目录开始向父级目录寻找包含 tsconfig 文件的目录。
* 调用 tsc 命令并且没有其他输入文件参数，可以使用 --project （或者只是 -p）的命令行选项来指定包含了 tsconfig.json 的目录，或者包含有效配置的 .json 文件路径。

当命令行中指定了`输入文件参数`， tsconfig.json 文件会被忽略。

### 示例

tsconfig.json 文件示例：

* 使用 `files` 属性

``` json
{
  "compilerOptions": {
    "module": "commonjs",
    "noImplicitAny": true,
    "removeComments": true,
    "preserveConstEnums": true,
    "sourceMap": true
  },
  "files": [
    "core.ts",
    "sys.ts",
    "types.ts",
    "scanner.ts",
    "parser.ts",
    "utilities.ts",
    "binder.ts",
    "checker.ts",
    "emitter.ts",
    "program.ts",
    "commandLineParser.ts",
    "tsc.ts",
    "diagnosticInformationMap.generated.ts"
  ]
}
```

* 使用 `include` 和 `exclude` 属性

``` json
{
  "compilerOptions": {
    "module": "system",
    "noImplicitAny": true,
    "removeComments": true,
    "preserveConstEnums": true,
    "outFile": "../../built/local/tsc.js",
    "sourceMap": true
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "**/*.spec.ts"]
}
```

### 基本的 TSConfig

根据你要在其中运行代码的不同的 JavaScript 运行时环境，你可以在 [github.com/tsconfig/bases](https://github.com/tsconfig/bases/) 上寻找一个合适的基本配置。你可以通过扩展这些已经处理过不同的 JavaScript 运行时环境的 tsconfig.json 文件来简化你项目中的 tsconfig.json。

举个例子，如果你的项目是基于 `Node.js 12.x` 写的，那么你可以使用 npm 模块：@tsconfig/node12：

``` json
{
  "extends": "@tsconfig/node12/tsconfig.json",
  "compilerOptions": {
    "preserveConstEnums": true
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "**/*.spec.ts"]
}
```

这使你的 tsconfig.json 专注在你的项目的目标环境上，而不是所有可能的运行时环境。现在已经有了一些 tsconfig 基础配置，我们希望社区能够为不同的环境添加更多的内容。

* [推荐配置](https://www.npmjs.com/package/@tsconfig/recommended)
* [Node 10](https://www.npmjs.com/package/@tsconfig/node10)
* [Node 12](https://www.npmjs.com/package/@tsconfig/node12)
* [Node 14](https://www.npmjs.com/package/@tsconfig/node14)
* [Deno](https://www.npmjs.com/package/@tsconfig/deno)
* [React Native](https://www.npmjs.com/package/@tsconfig/react-native)
* [Svelte](https://www.npmjs.com/package/@tsconfig/svelte)

### 细节

当没有指定 `compilerOptions` 时，会使用编译器的`默认配置`。请参考我们支持的[编译器选项](https://www.typescriptlang.org/tsconfig)列表。

### TSConfig 参考

想要了解更多的配置选项的信息，请访问[TSConfig Reference](https://www.typescriptlang.org/tsconfig)。

### 协议

tsconfig.json 的协议可以在这里找到[the JSON Schema Store](http://json.schemastore.org/tsconfig)。

### 选项

#### include

在编程中指定一组文件名或匹配模式来`包含` (include) 它们。这些文件名会以包含 `tsconfig.json` 的目录作为相对路径被解析。

``` json
{
  "include": ["src/**/*", "tests/**/*"]
}
```

包含以下：

``` text
.
├── scripts                ⨯
│   ├── lint.ts            ⨯
│   ├── update_deps.ts     ⨯
│   └── utils.ts           ⨯
├── src                    ✓
│   ├── client             ✓
│   │    ├── index.ts      ✓
│   │    └── utils.ts      ✓
│   ├── server             ✓
│   │    └── index.ts      ✓
├── tests                  ✓
│   ├── app.test.ts        ✓
│   ├── utils.ts           ✓
│   └── tests.d.ts         ✓
├── package.json
├── tsconfig.json
└── yarn.lock
```

 `include` 和 `exclude` 支持通配符形成 `glob` 匹配模式：

* `\*`匹配 0 个或多个字符 (不包括目录分割符)；
* `?` 匹配任意一个字符 (不包括目录分割符)；
* `**/`匹配任意包含嵌套层级的目录。

如果 `glob` 匹配模式不包含文件后缀，那么只有支持的文件会包含进来 (例：`.ts`、`.tsx`以及默认的`.d.ts`，若 `allowJs` 为 true，那么也包含`.js`和`.jsx`)。

#### compilerOptions

##### 模块解析

###### types

默认情况下，所有`可见`的`@types`包都将包含在你的编译过程中。 在 `node_modules/@types` 中的任何包都被认为是`可见`的。 例如，这意味着包含`./node_modules/@types/``../node_modules/@types/`，`../../node_modules/@types/`中所有的包。

当 `types` 被指定，则`只有`列出的包才会被包含在`全局范围`内。例如：

``` json
{
  "compilerOptions": {
    "types": ["node", "jest", "express"]
  }
}
```

这个 tsconfig.json 文件将`只会`包含`./node_modules/@types/node`，`./node_modules/@types/jest`和`./node_modules/@types/express`。其他在 `node_modules/@types/*` 下的包将`不会`被包含。

##### 互用约束

###### esModuleInterop

默认情况下（未设置 `esModuleInterop` 或值为 false），TypeScript 像 `ES6 模块` 一样对待 `CommonJS/AMD/UMD` 。这样的行为有两个被证实的缺陷：

* 形如 `import * as moment from "moment"` 这样的命名空间导入等价于 `const moment = require("moment")`
* 形如 `import moment from "moment"` 这样的默认导入等价于 `const moment = require("moment").default`

这种错误的行为导致了这两个问题：

* ES6 模块规范规定，`命名空间导入`（import * as x）`只能`是一个`对象`。TypeScript 把它处理成`= require("x")`的行为`允许`把导入`当作一个可调用的函数`，这样不符合规范。

* 虽然 TypeScript 准确实现了 ES6 模块规范，但是大多数使用 `CommonJS/AMD/UMD` 模块的库`并没有`像 TypeScript 那样严格遵守。

开启 `esModuleInterop` 选项将会修复 TypeScript 转译中的这两个问题。第一个问题通过改变编译器的行为来修复，第二个问题则由两个新的工具函数来解决，它们提供了确保生成的 JavaScript 兼容性的适配层：

``` ts
import * as fs from "fs";
import _ from "lodash";
fs.readFileSync("file.txt", "utf8");
_.chunk(["a", "b", "c", "d"], 2);
```

当 esModuleInterop `未启用`：

``` ts
"use strict";
Object.defineProperty(exports, "__esModule", { value: true });
const fs = require("fs");
const lodash_1 = require("lodash");
fs.readFileSync("file.txt", "utf8");
lodash_1.default.chunk(["a", "b", "c", "d"], 2);
```

当 esModuleInterop `启用`：

``` ts
"use strict";
var __createBinding = (this && this.__createBinding) || (Object.create ? (function(o, m, k, k2) {
    if (k2 === undefined) k2 = k;
    var desc = Object.getOwnPropertyDescriptor(m, k);
    if (!desc || ("get" in desc ? !m.__esModule : desc.writable || desc.configurable)) {
      desc = { enumerable: true, get: function() { return m[k]; } };
    }
    Object.defineProperty(o, k2, desc);
}) : (function(o, m, k, k2) {
    if (k2 === undefined) k2 = k;
    o[k2] = m[k];
}));
var __setModuleDefault = (this && this.__setModuleDefault) || (Object.create ? (function(o, v) {
    Object.defineProperty(o, "default", { enumerable: true, value: v });
}) : function(o, v) {
    o["default"] = v;
});
var __importStar = (this && this.__importStar) || function (mod) {
    if (mod && mod.__esModule) return mod;
    var result = {};
    if (mod != null) for (var k in mod) if (k !== "default" && Object.prototype.hasOwnProperty.call(mod, k)) __createBinding(result, mod, k);
    __setModuleDefault(result, mod);
    return result;
};
var __importDefault = (this && this.__importDefault) || function (mod) {
    return (mod && mod.__esModule) ? mod : { "default": mod };
};
Object.defineProperty(exports, "__esModule", { value: true });
const fs = __importStar(require("fs"));
const lodash_1 = __importDefault(require("lodash"));
fs.readFileSync("file.txt", "utf8");
lodash_1.default.chunk(["a", "b", "c", "d"], 2);
```

参考资料：

\> [https://www.typescriptlang.org/zh/docs/handbook/tsconfig-json.html](https://www.typescriptlang.org/zh/docs/handbook/tsconfig-json.html)

\> [https://www.typescriptlang.org/zh/tsconfig](https://www.typescriptlang.org/zh/tsconfig)
