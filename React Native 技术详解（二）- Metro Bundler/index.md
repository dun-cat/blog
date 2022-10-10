## React Native 技术详解（二）- Metro Bundler 
### Metro 介绍

在[React Native 技术详解（一）- 认识它](../react-native-1-introduction/#metro-bundler)中，简单介绍过 Metro Bundler。[Metro](https://github.com/facebook/metro) 是`构建 jsbundle 包`及提供`开发服务`的工具，默认被集成在`react-native`命令行工具内，你可以在[这里](https://github.com/react-native-community/cli/blob/main/packages/cli-plugin-metro/src/commands/start/runServer.ts#L9)找到其开发服务集成源码。

`react-native`命令行工具源码是由[Lerna](https://lerna.js.org/docs/introduction)管理的 monorepo 仓库，每个子命令在单独的子包里。而 React Native 的打包由其`cli-plugin-metro`子包管理。

在`@react-native-community/cli-plugin-metro`的 9.1.1 版本中，有两个命令：`start`和`bundle`，分别在以下[目录](https://github.com/react-native-community/cli/tree/main/packages/cli-plugin-metro/src/commands)里：

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

通常你可以直接用`react-native bundle`命令打出一个 jsbundle 文件及其资源目录：

``` bash
react-native bundle --platform android --dev false --entry-file index.js --bundle-output dist/index.bundle --assets-dest dist/
# 为 Android 平台打一个 jsbundle 包
```

该命令最终调用会调用 metro 包的`Server.js`文件的[build()](https://github.com/facebook/metro/blob/main/packages/metro/src/Server.js#L179)方法以及[getAssets()](https://github.com/facebook/metro/blob/main/packages/metro/src/Server.js#L316)方法。

`build()`会返回两个值`code`和`map`，然后完成 jsbundle 文件的存储：

* code： 表示已经打包完成的目标代码；
* map：表示 sourcemap。

`getAssets()`会获取到资源文件列表`AssetData[]`，然后根据对应平台（android|ios）把资源文件复制到指定目标目录。

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

Metro 通过项目根目录`metro.config.js`文件来对打包进行配置，metro.config.js 的配置结构如下：

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

你可以在[这里](https://facebook.github.io/metro/docs/configuration)获取全部配置项详情。其中，`resolver`、`transformer`、`serializer`三个配置项是值得我们去关注的，下面将详细介绍。

Metro 项目的子包`metro-config`可以找到其[默认配置](https://github.com/facebook/metro/blob/main/packages/metro-config/src/defaults/index.js)选项，有几个默认配置需要关注的：

* **projectRoot**
  
  指定 React Native 项目的根目录。如果未指定，默认通过`node_modules/metro-config`的位置解析。若指定的 projectRoot 不正确，那么在 Metro 的解析阶段将直接报错。

  ``` js
  projectRoot: projectRoot || path.resolve(__dirname, '../../..'),
  ```

* **cacheStores**
  
  提供转换后的缓存文件的存储目录，默认存储至`系统临时目录`。

  ```js
  cacheStores: [
    new FileStore({
      root: path.join(os.tmpdir(), 'metro-cache'),
    }),
  ],
  ```

* **resetCache**

  每次编译模块是否忽略缓存重新执行转换，默认值为`false`，即使用缓存。
  > 有时候缓存文件未必是正确的可用文件，此时可以在 react-native 命令后面指定`--reset-cache`参数或设置改选项为 true 来修复问题。

### Metro 打包的三个阶段

Metro 的打包过程有三个阶段：**Resolution（解析）**、**Transformation（转换）**、**Serialization（序列化）**。

![react-native-metro](react-native-metro.svg)

#### Resolution（解析）

该阶段用于解析模块文件及垫片文件的路径。

Resolution 阶段会去从`入口文件`开始，寻找模块的文件路径，构建一张所有模块的图，它的具体执行位置在`IncrementalBundler.js`文件的[buildGraph()](https://github.com/facebook/metro/blob/main/packages/metro/src/IncrementalBundler.js#L186)方法，该函数有两个返回值：`prepend`和`graph`。

1.当打印`graph`时，它的结构如下：

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

`entryPoints`是应用的入口文件路径，而`dependencies`是由模块文件`绝对路径`为 key 组成的所有模块 map 结构。

入口文件 **index.js** 的源码如下：

``` ts
/**
 * @format
 */

import {AppRegistry} from 'react-native';
import App from './App';
import {name as appName} from './app.json';

AppRegistry.registerComponent(appName, () => App);
```

我们可以从`dependencies`的第一项获取解析之后结构：

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

`metro.config.js`暴露了一个实验性的勾子`experimentalSerializerHook`选项，你可以对`graph`进行调整。

``` js
  serializer: {
    experimentalSerializerHook: (graph) => {
      return graph
    }
  }
```

通过`graph`，我使用 D3.js 构建了一张依赖图，如下面展示那样：

<iframe width="100%" height="1084" frameborder="0"
  src="https://observablehq.com/embed/@dun-cat/mobile-patent-suits?cells=chart"></iframe>


2.`prepend`的值为一些`垫片（Polyfill）`文件路径组成的数组：

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

#### Transformation（转换）

该阶段用于转义文件至目标平台能够理解的代码。

##### Babel 代码转义

Metro 使用 Babel 作为转义工具，如果你使用`react-native init`初始化项目的话，可以在项目根目录看到`babel.config.js`配置文件，内容如下：

``` js
module.exports = {
  presets: ['module:metro-react-native-babel-preset'],
};
```

Metro 提供了预设`metro-react-native-babel-preset`，该预设是 Metro 项目的一个子包，在[这里](https://github.com/facebook/metro/blob/main/packages/metro-react-native-babel-preset/src/configs/main.js)可以看到其具体预设内容。

其内置的一些默认插件：

* `@babel/plugin-syntax-flow`：用于支持 Facebook 自家的[Flow](https://flow.org/en/docs/getting-started/)静态类型文件（.flow.js）。 Flow 和 Typescript 类似，却并未推广开；
* `@babel/plugin-transform-block-scoping`：支持作用块（let）；
* `@babel/plugin-transform-typescript`：支持 typescript 的转义；
* `@babel/plugin-transform-arrow-functions`：对箭头函数的支持；
* `@babel/plugin-syntax-dynamic-import`：模块的`异步加载`支持；
* ...

预设提供了实验性设置选项`unstable_transformProfile`。如果`unstable_transformProfile`的值为`hermes-stable`或`hermes-canary`其中之一，并且项目中采用`Hermes`作为 Javascript 引擎，那么下面的一些插件会被忽略：

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

Metro 在转换文件时，使用 Worker 来并行执行。Worker 的执行文件为：`Worker.flow.js`，并暴露的执行的函数为[transform(...)](https://github.com/facebook/metro/blob/main/packages/metro/src/DeltaBundler/Worker.flow.js#L72)。

Metro 使用 Facebook 自家 Jest 测试框架的[jest-worker](https://github.com/facebook/jest/blob/main/packages/jest-worker/README.md)来创建多个 worker，你可以[这里](https://github.com/facebook/metro/blob/main/packages/metro/src/DeltaBundler/WorkerFarm.js#L120)看到 jest-worker 的实例化。

通过[maxworkers](https://facebook.github.io/metro/docs/configuration#maxworkers)选项，可以指定多少个 worker 一起并行执行模块转换工作。不过通常不需要进行设置，[默认配置](https://github.com/facebook/metro/blob/main/packages/metro/src/lib/getMaxWorkers.js)会根据 cpu 核数来进行合理配置。

#### Serialization（序列化）
