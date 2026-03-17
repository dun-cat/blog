## Webpack 构建常用配置 
### 加载器 (Loader)

 [Loader](https://webpack.docschina.org/concepts/loaders/) 用于对模块的源代码进行转换。loader 可以使你在 import 或 "load(加载) " 模块时预处理文件。

因此，loader 类似于其他构建工具中“任务(task)”，并提供了处理前端构建步骤的得力方式。

loader 可以将文件从不同的语言（如 TypeScript）转换为 JavaScript 或将`内联图像`转换为 `data URL` 。loader 甚至允许你直接在 `JavaScript` 模块中 `import CSS` 文件！

#### 样式处理

##### css-loader

css-loader 会对`@import`和 `url()` 进行处理，就像 js 解析 `import/require()` 一样。

###### 使用

file.js

``` ts
import css from "file.css";
```

webpack.config.js

``` ts
module.exports = {
  module: {
    rules: [
      {
        test: /\.css$/i,
        use: ["style-loader", "css-loader"],
      },
    ],
  },
};
```

#### TypeScript 处理

##### ts-loader

TypeScript 是 JavaScript 的超集，为其增加了类型系统。通过[ts-loader](https://webpack.docschina.org/guides/typescript/)可以编译把 TypeScript 为普通 JavaScript 代码。

###### 安装

``` shell
npm install --save-dev typescript ts-loader
```

现在，我们将修改目录结构和配置文件：

**project**

``` diff
  webpack-demo
  |- package.json
  |- package-lock.json
+ |- tsconfig.json
  |- webpack.config.js
  |- /dist
    |- bundle.js
    |- index.html
  |- /src
-   |- index.js
+   |- index.ts
  |- /node_modules
```

**tsconfig.json**

这里我们设置一个基本的配置来支持 JSX，并将 TypeScript 编译到 ES5……

``` json
{
  "compilerOptions": {
    "outDir": "./dist/",
    "noImplicitAny": true,
    "module": "es6",
    "target": "es5",
    "jsx": "react",
    "allowJs": true,
    "moduleResolution": "node"
  }
}
```

查看[TypeScript 官方文档](https://www.typescriptlang.org/docs/handbook/tsconfig-json.html)了解更多关于 tsconfig.json 的配置选项。

想要了解 webpack 配置的更多信息，请查看[配置](https://webpack.docschina.org/concepts/configuration/)概念。现在，配置 webpack 处理 TypeScript：

**webpack.config.js**

``` ts
const path = require('path');

module.exports = {
  entry: './src/index.ts',
  module: {
    rules: [
      {
        test: /\.tsx?$/,
        use: 'ts-loader',
        exclude: /node_modules/,
      },
    ],
  },
  resolve: {
    extensions: ['.tsx', '.ts', '.js'],
  },
  output: {
    filename: 'bundle.js',
    path: path.resolve(__dirname, 'dist'),
  },
};
```

这会让 webpack 直接从`./index.ts`进入，然后通过 `ts-loader` 加载所有的`.ts`和`.tsx`文件，并且在当前目录输出一个 `bundle.js` 文件。

{{% notice warning %}}
 `ts-loader` 使用 `tsc`，TypeScript 编译器会依赖 `tsconfig.json` 文件作为编译配置。要避免设置 `module` 为 `CommonJS`，否则 webpack 不会[tree-shake 你的代码](https://webpack.docschina.org/guides/tree-shaking)。
{{% /notice %}}

### 插件

#### HTML 处理

##### html-webpack-plugin

[HtmlWebpackPlugin](https://github.com/jantimon/html-webpack-plugin) 通过该插件可以`生成一个 HTML 文件`。该插件对于浏览器端是必备的。

###### 基本用法

该插件将为你生成一个 HTML5 文件， 在 `body` 中使用 `script 标签` 引入你所有 webpack 生成的 `bundle` 。 只需添加该插件到你的 webpack 配置中，如下所示：

``` ts
const HtmlWebpackPlugin = require('html-webpack-plugin');
const path = require('path');

module.exports = {
  entry: 'index.js',
  output: {
    path: path.resolve(__dirname, './dist'),
    filename: 'index_bundle.js',
  },
  plugins: [new HtmlWebpackPlugin()],
};
```

这将会生成一个包含以下内容的 `dist/index.html` 文件：

``` html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="UTF-8" />
    <title>webpack App</title>
  </head>
  <body>
    <script src="index_bundle.js"></script>
  </body>
</html>
```

如果你有`多个` webpack 入口，他们都会在已生成 HTML 文件中的`<script>`标签内引入。

#### 样式处理

##### mini-css-extract-plugin

本插件会将 CSS `提取`到`单独的`文件中，为`每个`包含 `CSS` 的 `JS` 文件创建一个 `CSS` 文件，并且支持 CSS 和 SourceMaps 的`按需加载`。

本插件基于 `webpack v5` 的新特性构建，并且需要 `webpack 5` 才能正常工作。

与 `extract-text-webpack-plugin` 相比：

* 异步加载
* 没有重复的编译（性能）
* 更容易使用
* 特别针对 CSS 开发

###### 基本用法

建议 `mini-css-extract-plugin` 与[css-loader](https://webpack.docschina.org/loaders/css-loader/)一起使用。

之后将 `loader` 与 `plugin` 添加到你的 webpack 配置文件中。 例如：

style.css

``` css
body {
  background: green;
}
```

component.js

``` ts
import "./style.css";
```

webpack.config.js

``` ts
const MiniCssExtractPlugin = require("mini-css-extract-plugin");

module.exports = {
  plugins: [new MiniCssExtractPlugin()],
  module: {
    rules: [
      {
        test: /\.css$/i,
        use: [MiniCssExtractPlugin.loader, "css-loader"],
      },
    ],
  },
};
```

{{% notice note %}}
如果你从 webpack`入口文件`处导入 CSS 或者在[初始 chunk](https://webpack.docschina.org/concepts/under-the-hood/#chunks)中引入 style， mini-css-extract-plugin 则`不会`将这些 CSS 加载到页面中。

请使用 html-webpack-plugin `自动生成 link 标签`或者在创建 index.html 文件时使用 link 标签。
{{% /notice %}}

参考资料：

\> [https://webpack.docschina.org/plugins/](https://webpack.docschina.org/plugins/)
