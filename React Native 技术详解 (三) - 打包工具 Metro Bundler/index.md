## React Native 技术详解 (三) - 打包工具 Metro Bundler 
### Metro 介绍

在[React Native 技术详解 (一) - 认识它](../react-native-1-introduction/#metro-bundler)中，简单介绍过 Metro Bundler。 [Metro](https://github.com/facebook/metro) 是`构建 jsbundle 包`及提供`开发服务`的工具，默认被集成在 `react-native` 命令行工具内，你可以在[这里](https://github.com/react-native-community/cli/blob/e89f296b1f1b27da23ffb77e3c8fc5bc2f4942ee/packages/cli-plugin-metro/src/commands/start/runServer.ts#L9) 找到其开发服务集成源码。

 `react-native` 命令行工具源码是由 [Lerna](https://lerna.js.org/docs/introduction) 管理的 monorepo 仓库，每个子命令在单独的子包里。而 React Native 的打包由其 `cli-plugin-metro` 子包管理。

在 `@react-native-community/cli-plugin-metro` 的 9.1.1 版本中，有两个命令：`start` 和 `bundle`，分别在以下[目录](https://github.com/react-native-community/cli/tree/main/packages/cli-plugin-metro/src/commands)里：

``` bash
├── CHANGELOG.md
├── package.json
├── src
│   ├── commands
│   │   ├── bundle # 打包
│   │   ├── index.ts
│   │   └── start # 开发服务
│   ├── index.ts
│   └── tools
│       ├── __tests__
│       ├── loadMetroConfig.ts
│       └── metroPlatformResolver.ts
└── tsconfig.json
```

通常你可以直接用 `react-native bundle` 命令打出一个平台的 jsbundle 文件及其资源目录：

``` bash
react-native bundle --platform android --dev false --entry-file index.js --bundle-output dist/index.bundle --assets-dest dist/
```

该命令最终会调用 metro 包的 `Server.js` 文件的 [build()](https://github.com/facebook/metro/blob/fa103665c9cd555e3f78e6ed3ef6c54df92687fa/packages/metro/src/Server.js#L179) 方法以及 [getAssets()](https://github.com/facebook/metro/blob/fa103665c9cd555e3f78e6ed3ef6c54df92687fa/packages/metro/src/Server.js#L316) 方法来完成打包工作。

 `build()` 会返回两个值 `code` 和 `map`，然后完成 jsbundle 文件的存储：

* code： 表示已经打包完成的目标代码；
* map：表示 sourcemap。

 `getAssets()` 会获取到资源文件列表 `AssetData[]`，然后根据对应平台 (android | ios) 把资源文件复制到指定目标目录。

``` ts
export interface AssetData {
  __packager_asset: boolean;
  fileSystemLocation: string;
  hash: string;
  height: number | null;
  httpServerLocation: string;
  name: string;
  scales: number[];
  type: string;
  width: number | null;
  files: string[];
}
```

> 虽然 Metro 提供了 API，但是 react-native 并没有直接使用。

### 配置

Metro 通过项目根目录 `metro.config.js` 文件来对打包进行配置，metro.config.js 的配置结构如下：

``` js
module.exports = {
  /* general options */

  resolver: {
    /* resolver options */
  },
  transformer: {
    /* transformer options */
  },
  serializer: {
    /* serializer options */
  },
  server: {
    /* server options */
  },
  watcher: {
    /* watcher options */
    watchman: {
      /* Watchman-specific options */
    }
  }
};
```

从[源码](https://github.com/facebook/metro/blob/fa103665c9cd555e3f78e6ed3ef6c54df92687fa/packages/metro-config/src/loadConfig.js#L84)中得知，Metro 使用 [cosmiconfig](https://github.com/davidtheclark/cosmiconfig) 来加载配置文件的，`metro.config.js` 配置也允许是一个函数，并且接收一个默认配置的实例，因此你也可以直接对已有配置进行修改。

``` js
module.exports = (defaultConfig) => {
  return defaultConfig;
}
```

你可以在[这里](https://facebook.github.io/metro/docs/configuration)获取全部配置项详情。其中，`resolver`、`transformer`、`serializer` 三个配置项下面将详细介绍。

#### metro-config

 `metro-config` 是 Metro 项目的子包，用于配置 metro 打包。它暴露了以下的一些方法，这些方法方便我们做 React Native 的深度开发。

``` js
module.exports = {
  loadConfig,
  resolveConfig,
  mergeConfig,
  getDefaultConfig,
};
```

在[源码](https://github.com/facebook/metro/blob/fa103665c9cd555e3f78e6ed3ef6c54df92687fa/packages/metro-config/src/defaults/index.js)中，你可以找到其默认配置选项，有几个默认配置需要关注的：

* **projectRoot**
  
  指定 React Native 项目的根目录。如果未指定，默认通过 `node_modules/metro-config` 的位置解析。若指定的 projectRoot 不正确，那么在 Metro 的解析阶段将直接报错。

  ``` js
  projectRoot: projectRoot || path.resolve(__dirname, '../../..'),
  ```

* **cacheStores**
  
  提供转换后的缓存文件的存储位置，默认存储至`系统临时目录`。

  ```js
  cacheStores: [
    new FileStore({
      root: path.join(os.tmpdir(), 'metro-cache'),
    }),
  ],
  ```

  除了默认存储至本地文件系统，你还添加一个`服务器存储`：

  ``` js
  cacheStores: [
    new FileStore({/*opts*/}),
    new HttpStore({/*opts*/})
  ]
  ```

  这种缓存设计是分层的 (multi-layered cache) ，让构建`缓存共享`变成可能。

* **resetCache**

  每次编译模块是否忽略缓存重新执行转换，默认值为 `false`，即使用缓存。
  > 有时候缓存文件未必是正确的可用文件，此时可以在 react-native 命令后面指定 `--reset-cache` 参数或设置改选项为 true 来修复问题。

#### 加载逻辑

若你使用 `react-native` 作为执行 jsbundle 的打包命令工具，那么它将有三个主要配置主体：`react-native cli`、`metro-config`、`metro.config.js` 。

他们的执行时序图如下：

![react-native-metro-config](react-native-metro-config.svg)

1. 用户执行 `react-native bundle` 打包命令，携带命令行参数 `args` ；
2. `react-native cli` 内置一份默认部分配置 `defaultConfigOverrides`，将其与 `metro-config` 包内的 `defaults` 进行合并生成新的一份 `defaultConfig` ；
3. 读取用户的 `metro.config.js` 文件获取配置 `configModule` 与 `defaultConfig` 进行合并生成 `config`，返回给 `react-native cli` ；
4. 命令行的参数优先级要高于配置。因此，若 `args` 与 `config` 配置重复，`args` 会覆盖它。

### Metro 打包的三个阶段

Metro 的打包过程有三个阶段：**Resolution (解析)**、**Transformation (转换)**、**Serialization (序列化)**。

![react-native-metro](react-native-metro.svg)

#### Resolution (解析)

该阶段用于解析`模块文件`的路径。Metro 实现了 [Node 模块解析](https://nodejs.org/api/modules.html#loading-from-node_modules-folders)的一个版本。

从`入口文件`开始，寻找依赖模块的文件路径，构建一张所有模块的图，它的具体顶层执行位置在 `IncrementalBundler.js` 文件的 [buildGraph()](https://github.com/facebook/metro/blob/fa103665c9cd555e3f78e6ed3ef6c54df92687fa/packages/metro/src/IncrementalBundler.js#L186) 方法，该函数有两个返回值：`prepend` 和 `graph` 。

1.当打印 `graph` 时，它的结构如下：

``` js
{
  entryPoints: Set(1) { '/Users/lumin/Desktop/AwesomeTSProject/index.js' },
  transformOptions: {
    customTransformOptions: [Object: null prototype] {},
    dev: false,
    hot: false,
    minify: true,
    platform: 'android',
    runtimeBytecodeVersion: null,
    type: 'module',
    unstable_transformProfile: undefined
  },
  dependencies: Map(388) {
    '/Users/lumin/Desktop/AwesomeTSProject/index.js' => {
      inverseDependencies: CountingSet {},
      path: '/Users/lumin/Desktop/AwesomeTSProject/index.js',
      dependencies: Map(4) {
        'XEo4Z+Aarw9Y7I7ZLBt66vGLAVQ=' => [Object],
        '7kvm5yrOpz4NYiDi6sn4qxa8DVQ=' => [Object],
        'THJ70LbOuESuoZ1vCvaV/6cOUg0=' => [Object],
        '1udmcfu0G1JlZthWUoMNlgdDqG0=' => [Object]
      },
      getSource: [Function: getSource],
      output: [Array]
    },
    ...,
    '/Users/lumin/Desktop/AwesomeTSProject/App.tsx' => {
      inverseDependencies: CountingSet {},
      path: '/Users/lumin/Desktop/AwesomeTSProject/App.tsx',
      dependencies: [Map],
      getSource: [Function: getSource],
      output: [Array]
    },
    ...,
    '/Users/lumin/Desktop/AwesomeTSProject/app.json' => {
      inverseDependencies: CountingSet {},
      path: '/Users/lumin/Desktop/AwesomeTSProject/app.json',
      dependencies: Map(0) {},
      getSource: [Function: getSource],
      output: [Array]
    }
  },
  importBundleNames: Set(0) {},
  privateState: {
    resolvedContexts: Map(0) {},
    gc: {
      color: [Map],
      possibleCycleRoots: Set(0) {},
      importBundleRefs: Map(0) {}
    }
  }
}
```

 `entryPoints` 是应用的入口文件路径，而 `dependencies` 是由模块文件`绝对路径`为 key 组成的所有模块 map 结构。

目前的入口文件 **index.js** 的源码如下：

``` ts
/**
 * @format
 */

import {AppRegistry} from 'react-native';
import App from './App';
import {name as appName} from './app.json';

AppRegistry.registerComponent(appName, () => App);
```

我们可以从 `dependencies` 的第一项获取 index.js 模块解析之后数据结构：

``` ts
{
  inverseDependencies: CountingSet {},
  path: '/Users/lumin/Desktop/AwesomeTSProject/index.js',
  dependencies: Map(4) {
    'XEo4Z+Aarw9Y7I7ZLBt66vGLAVQ=' => {
      absolutePath: '/Users/lumin/Desktop/AwesomeTSProject/node_modules/react-native/index.js',
      data: [Object]
    },
    '7kvm5yrOpz4NYiDi6sn4qxa8DVQ=' => {
      absolutePath: '/Users/lumin/Desktop/AwesomeTSProject/node_modules/@babel/runtime/helpers/interopRequireDefault.js',
      data: [Object]
    },
    'THJ70LbOuESuoZ1vCvaV/6cOUg0=' => {
      absolutePath: '/Users/lumin/Desktop/AwesomeTSProject/App.tsx',
      data: [Object]
    },
    '1udmcfu0G1JlZthWUoMNlgdDqG0=' => {
      absolutePath: '/Users/lumin/Desktop/AwesomeTSProject/app.json',
      data: [Object]
    }
  },
  getSource: [Function: getSource],
  output: [ { data: [Object], type: 'js/module' } ]
}
```

 `metro.config.js` 暴露了一个实验性的勾子 `experimentalSerializerHook` 选项，你可以对 `graph` 进行调整。

``` js
  serializer: {
    experimentalSerializerHook: (graph) => {}
  }
```

通过 [graph](https://github.com/dun-cat/code-snippets/blob/main/metro.config.js)，我使用 D3.js 构建了一张 Android 平台下的[模块依赖图](https://observablehq.com/embed/@dun-cat/mobile-patent-suits?cells=chart)

2. `prepend` 的值为一些`垫片 (Polyfill)`文件路径组成的数组：

``` ts
[
  {
    dependencies: Map(0) {},
    getSource: [Function: getSource],
    inverseDependencies: CountingSet {},
    path: '__prelude__',
    output: [ [Object], [Object] ]
  },
  {
    inverseDependencies: CountingSet {},
    path: '/Users/lumin/Desktop/AwesomeTSProject/node_modules/metro-runtime/src/polyfills/require.js',
    dependencies: Map(0) {},
    getSource: [Function: getSource],
    output: [ [Object] ]
  },
  {
    inverseDependencies: CountingSet {},
    path: '/Users/lumin/Desktop/AwesomeTSProject/node_modules/@react-native/polyfills/console.js',
    dependencies: Map(0) {},
    getSource: [Function: getSource],
    output: [ [Object] ]
  },
  {
    inverseDependencies: CountingSet {},
    path: '/Users/lumin/Desktop/AwesomeTSProject/node_modules/@react-native/polyfills/error-guard.js',
    dependencies: Map(0) {},
    getSource: [Function: getSource],
    output: [ [Object] ]
  },
  {
    inverseDependencies: CountingSet {},
    path: '/Users/lumin/Desktop/AwesomeTSProject/node_modules/@react-native/polyfills/Object.es8.js',
    dependencies: Map(0) {},
    getSource: [Function: getSource],
    output: [ [Object] ]
  }
]
```

在最终的代码合并之后，垫片代码会处于 jsbundle 的顶部，在加载 jsbundle 时，预先执行。

##### 可视化分析

你可以使用 [react-native-bundle-visualizer](https://github.com/IjzerenHein/react-native-bundle-visualizer) 来分析你的模块依赖。通过它你可以很方便的看到那些占用体积较大的模块，并优化它们。

![source-map-v](source-map-v.png)

该工具原理是通过 `react-native bundle` 命令打包，在系统临时目录生成 jsbundle 及对应的 source map 文件，通过分析 source map 输出可视化的依赖 html 报告。

因此，你需要提前确认 `react-native bundle` 命令能够正确的打出 jsbundle 包。

#### Transformation (转换)

该阶段用于转义文件至目标平台能够理解的代码。

Metro 使用 Babel 作为转义工具，如果你使用 `react-native init` 初始化项目的话，可以在项目根目录看到 `babel.config.js` 配置文件，内容如下：

``` js
module.exports = {
  presets: ['module:metro-react-native-babel-preset'],
};
```

##### metro-react-native-babel-preset

Metro 提供了预设 `metro-react-native-babel-preset`，该预设是 Metro 项目的一个子包，在[这里](https://github.com/facebook/metro/blob/fa103665c9cd555e3f78e6ed3ef6c54df92687fa/packages/metro-react-native-babel-preset/src/configs/main.js)可以看到其具体预设内容。

其内置的一些默认插件：

* `@babel/plugin-syntax-flow`：用于支持 Facebook 自家的 [Flow](https://flow.org/en/docs/getting-started/) 静态类型文件 (.flow.js) 。 Flow 和 Typescript 类似，却并未推广开；
* `@babel/plugin-transform-block-scoping`：支持作用块 (let) ；
* `@babel/plugin-transform-typescript`：支持 typescript 的转义；
* `@babel/plugin-transform-arrow-functions`：对箭头函数的支持；
* `@babel/plugin-syntax-dynamic-import`：模块的`异步加载`支持；
* ...

预设提供了实验性设置选项 `unstable_transformProfile` 。

如果 `unstable_transformProfile` 的值为 `hermes-stable` 或 `hermes-canary` 其中之一，并且项目中采用 `Hermes` 作为 Javascript 引擎，那么下面的一些插件会被忽略：

* @babel/plugin-transform-computed-properties
* @babel/plugin-transform-parameters
* @babel/plugin-transform-shorthand-properties
* @babel/plugin-proposal-optional-catch-binding
* @babel/plugin-transform-function-name
* @babel/plugin-transform-literals
* @babel/plugin-transform-sticky-regex

在 React Native 0.70 之后， Hermes 作为默认内置引擎，实现了很多标准，这带来了些许好处：

1. 无需 Babel 额外的转义代码，最终打包的 jsbundle 体积变小；
2. 由于原生语法支持，对这些语法代码执行性能得到一定的提升。

虽然目前为止该设置还处于实验性设置，相信后续 Hermes 会实现更多的语法标准，进一步提升 Javascript 引擎执行效率和用户的开发体验。

##### 多个 Worker 的并行转换

Metro 在转换文件时，使用 Worker 来并行执行。Worker 的执行入口文件为：`Worker.flow.js`，并暴露的执行的函数为 [transform(...)](https://github.com/facebook/metro/blob/fa103665c9cd555e3f78e6ed3ef6c54df92687fa/packages/metro/src/DeltaBundler/Worker.flow.js#L72)。

Metro 使用 Facebook 自家 Jest 测试框架的 [jest-worker](https://github.com/facebook/jest/blob/main/packages/jest-worker/README.md) 来创建多个 worker，你可以[这里](https://github.com/facebook/metro/blob/fa103665c9cd555e3f78e6ed3ef6c54df92687fa/packages/metro/src/DeltaBundler/WorkerFarm.js#L120)看到 jest-worker 的实例化。

通过 [maxworkers](https://facebook.github.io/metro/docs/configuration#maxworkers) 选项，可以指定多少个 worker 一起并行执行模块转换工作。不过通常不需要进行设置，[默认配置](https://github.com/facebook/metro/blob/fa103665c9cd555e3f78e6ed3ef6c54df92687fa/packages/metro/src/lib/getMaxWorkers.js)会根据 cpu 核数来进行合理配置。

![react-native-metro-worker-pool](react-native-metro-worker-pool.svg)

##### metro-transform-worker

做模块转换前，Metro 会通过 `metro-transform-worker` 子包来`指定对应模块类型的转换器`，它暴露一个 `transform` 方法。

``` ts
module.exports = {
  transform: async (
    config: JsTransformerConfig,
    projectRoot: string,
    filename: string,
    data: Buffer,
    options: JsTransformOptions,
  ): Promise<TransformResponse>{
    ...
  }
}
```

Metro 提供了 [transformerpath](https://facebook.github.io/metro/docs/configuration#transformerpath) 选项允许让开发者替换 `metro-transform-worker` 。如果需要做模块转义的完全的开发，该配置项将必不可少。不过，一旦你选择自定义，则官方提供的 [transformer](https://facebook.github.io/metro/docs/configuration#transformer-options) 选项都将失效。

那么，默认是否可以自定义转换呢？

可以的，在 tranformer 选项中提供了 [babelTransformerPath](https://facebook.github.io/metro/docs/configuration#babeltransformerpath) 配置，你可以指定一个实现 `transform` 接口函数的模块绝对路径。

``` js
module.exports.transform = ({ filename, options, plugins, src }) => {
  // transform file...
  return { ast: AST };
}
```

Metro 会指定其子包 `metro-react-native-babel-transformer` 解析后的绝对路径作为 `babelTransformerpath` 的默认值。

目前 Metro 有三种模块类型的转换器：`JS 文件转换器`、`JSON 文件转换器`、`资源文件转换器`。其中，`JS 文件转换器` 是对外部开放的，也就是 `babelTransformerPath` 选项配置。

##### metro-react-native-babel-transformer

上小节介绍到 `metro-react-native-babel-transformer` 作为 JS 模块的默认转换器，来执行真正转换任务。可以看到其暴露了两个函数：

``` js
...
module.exports = {
  transform,
  getCacheKey,
};
```

其中，`transform()` 为`必须`提供的接口函数，用于执行模块转换。 `getCacheKey()` 作为配置缓存函数，后续再介绍。

#### Serialization (序列化)

序列化阶段会把各个模块按照一定顺序组合到单个或者多个 jsbundle。通常它是单一的 jsbundle 文件输出。相对解析和转换，序列化的任务则少很多。

##### 自定义模块 ID

默认情况模块 id 会自动被分配为整数类型。Metro 允许你对每一个模块分配一个自定义 id 标识，你可以通过 [createModuleIdFactory](https://facebook.github.io/metro/docs/configuration#createmoduleidfactory) 选项来进行修改。

``` js
{
  serializer: {
    createModuleIdFactory: function () {
        return function (absoluteModulePath: string) {
          const customModuleId = crypto.randomUUID()
          return customModuleId;
        }
      }
    }
}
```

##### 忽略指定模块的打包

如果你要对 jsbundle 进行`代码分割` (Code Splitting) ，那 [processModuleFilter](https://facebook.github.io/metro/docs/configuration#processmodulefilter) 可以忽略对指定模块的打包。

``` js
{
  serializer: {
    processModuleFilter: function (modules) {
        return true;
    }
  }
}
```

### 缓存

Metro 为构建提供了缓存 (cache) 功能。在上面我们提到过通过设置 `cacheStores` 可以设置多个缓存存储位置，它的缓存功能实现由子包 `metro-cache` 和 `metro-cache-key` 提供。

目前为止，`metro-cache-key` 还是只是个很简单的工具库。

#### metro-cache

 `metro-cache` 为实现多层级缓存系统的一个子包，目前包括两种存储类型：`FileStore` 和 `HttpStore` 。

* FileStore：文件系统的持久化缓存；
* HttpStore：基于 Http 协议的网络存储缓存。目前没有提供鉴权相关的复杂实现，只需要提供一个简单的静态资源服务器。

Metro 开放了 `cacheStores` 配置项，那就说明你可以自定义一个缓存存储实现的 store。

### Source Map

Metro 提供了 `metro-source-map` 子包用于 source map 的生成，source map 生成遵循[version 3 版本](https://docs.google.com/document/d/1U1RGAehQwRypUTovF1KRlpiOFze0b-_2gc6fAH0KY0k/edit#heading=h.qz3o9nc69um5)提议。

### 代码压缩

Metro 提供了 `metro-minify-uglify` 和 `metro-minify-terser` 两个子包用于代码的压缩。在 Metro `0.73` 版本之前，默认使用 metro-minify-uglify 实现代码压缩。最新版本已经采用 metro-minify-terser 来实现。

> [这里](https://github.com/facebook/metro/pull/871)解释了为什么更换默认压缩混淆包的原因。主要 metro-minify-uglify 基于 `uglify-es` 仓库，而该仓库已经废弃，并暴露了一些遗留问题。

同时，Metro 允许自定义压缩功能。你可以指定配置选项 [minifierPath](https://facebook.github.io/metro/docs/configuration#minifierpath) 来自定义。

在 [Expo](https://docs.expo.dev/guides/customizing-metro/#minification-esbuild) 文档中，介绍使用 [metro-minify-esbuild](https://github.com/EvanBacon/metro-minify-esbuild) 三方包来加快压缩速度。

你也可以`忽略代码压缩`的步骤。使用 `react-native bundle` 时，添加`--minify`参数可以忽略代码压缩的执行，默认在开发服务状态下默认不进行压缩。

### 总结

Metro 作为构建工具可以看到常见的功能点：`构建缓存`、`构建任务的并行执行`、`构建定制的配置化`， Metro 分包实现这些功能也是一种不错的[模块化编程](https://zh.wikipedia.org/wiki/模块化编程)实践。

在`构建缓存`上，Metro 的多层级缓存是个不错的设计。虽然实用性有待考量，但`缓存共享`的概念令人耳目一新。

不过，Metro 并不完全像现代构建工具那样 (例如：webpack) ，拥有完善且丰富的功能。缺少`代码分割`、~~`符号链接的支持`~~。

### 补充

在 [React Native 0.72](https://reactnative.dev/blog/2023/06/21/0.72-metro-package-exports-symlinks#new-metro-features) 版本开始支持`符号连接` (symbol link)，并在 [React Native 0.73](https://reactnative.dev/blog/2023/12/06/0.73-debugging-improvements-stable-symlinks) 版本默认开启。

在 Metro 中，通过设置 [watchFolders](https://metrobundler.dev/docs/configuration/#watchfolders) 来依赖 `projectRoot` 以外的目录，使得开发服务和构建能够正确得解决模块引用路径。

