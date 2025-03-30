## Node 包的语义化版本号控制 
### 介绍

对版本号做个约定性的规范，你知我知大家知。
npm 管理包的版本基于语义化版本控制规范 (SemVer) 。

### 版本号

版本号格式：主版本号.次版本号.修订号

1. 主( `major` )版本号：当你做了不兼容的 API 修改；
2. 次( `minor` )版本号：当你做了向下兼容的功能性新增；
3. 修订( `patch` )版本号：当你做了向下兼容的问题修正；

详情参考[semver规范](https://semver.org/lang/zh-CN/)

主要介绍几点：

1. `v1.2.3 不是语义化版本号`，而是tag名。1.2.3才是。
2. 修订版本号的修正，指的是`针对不正确结果而进行的内部修改`。
3. 先行版软件，可以在版本号后面加窜字符。例如：1.0.0-alpha、1.0.0-alpha.1、1.0.0-0.3.7、1.0.0-x.7.z.92。
4. 可以0.1.0 作为软件初始开发的版本号。

### 安装包的范围版本选择

在 package.json 里可以看到 `( ~ )` 和 `( ^ )`等版本号版本前缀语法，其实还有更多丰富的版本范围语法。

~ ：表示安装的时候，主版本号和此版本号不变，只更新修订号到最后一个 (latsed) 版本。
 ^ ：表示安装的时候，主版本号不变，更新次版本号和修订版本号到最后一个 (latsed) 一个版本号 。

还有可以使用 >= , <= , > , < = , -, x ,*等符号。详情参考[npm/node-semver](https://github.com/npm/node-semver)

``` json
{
    "dependencies": {
        "lodash": "<2.4.1",
        "moment": "=1.0.0",
        "iview": "1 - 3",
        "element-ui": "1.x.x"
    }
}
```

### npm install 安装

在 npm install 安装，可以选择 x 符号匹配任意符号。
也可以使用 npm dist-tag的名称。 latset 和 next 是内置标签。具体参考[npm-dist-tag](https://docs.npmjs.com/cli/dist-tag.html#purpose)

``` bash
npm install lodash@x.x.x # 安装最新包
npm install lodash@1.x.x # 主版本号不变，安装最新包
npm install lodash@2.2.x # 主版本号和次版本号不变，安装最新修订版
npm install lodash@latest # 安装最后一个稳定发行版本
npm install lodash@next # 安装即将到来的发行版
```

可以使用版本查看工具：[https://semver.npmjs.com/](https://semver.npmjs.com/)

参考资料：

\> [https://semver.org/lang/zh-CN/](https://semver.org/lang/zh-CN/)

\> [https://github.com/npm/node-semver](https://github.com/npm/node-semver)

\> [https://github.com/semver/semver/blob/master/semver.md](https://github.com/semver/semver/blob/master/semver.md)

\> [https://docs.npmjs.com/cli/dist-tag.html#purpose](https://docs.npmjs.com/cli/dist-tag.html#purpose)
