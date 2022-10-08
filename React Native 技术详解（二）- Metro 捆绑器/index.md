## React Native 技术详解（二）- Metro 捆绑器 
### Metro 介绍

在[React Native 技术详解（一）- 认识它](../react-native-1-introduction/#metro-bundler)中，我们简单介绍过 Metro 捆绑器。Metro 是构建 jsbundle 的工具，并默认被集成在`react-native`命令行工具内，你可以在[这里](https://github.com/react-native-community/cli/blob/main/packages/cli-plugin-metro/src/commands/start/runServer.ts#L9)找到其开发服务集成源码。

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

`build()`会返回两个值`code`和`map`，并完成 jsbundle 的存储：

* code： 表示已经打包完成的目标代码；
* map：表示 sourcemap。

`getAssets()`会获取到资源文件列表`AssetData[]`，并根据平台执行资源到指定目标目录的复制。

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

### Metro 打包的三个阶段

Metro 打包有三个阶段：**Resolution（解析）**、**Transformation（转化）**、**Serialization（生成）**。

![react-native-metro](react-native-metro.svg)

#### Resolution

该阶段用于解决文件路径的。

Resolution 阶段会去从`入口文件`开始，寻找模块的文件路径，构建一张所有模块的图，它的具体执行位置在`IncrementalBundler.js`文件的[buildGraph()](https://github.com/facebook/metro/blob/main/packages/metro/src/IncrementalBundler.js#L186)方法，该函数有两个返回值：`prepend`和`graph`。

#### Transformation

该阶段用于转义文件至目标平台能够理解的代码。

#### Serialization
