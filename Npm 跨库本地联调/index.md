## Npm 跨库本地联调 
### link 联调存在的问题

 `npm link` 或者 `yarn link` 都能把需要对外发布为 npm 包的项目，生成一个全局的`符号链接` (Symlink) ，作为引用方的项目可以直接把`依赖包`指向该`符号链接`。

以 `yarn` 作为包管理工具为例，我们有一个 `package.json` 的 `name` 为`@lumin/mobile-ui`的项目，以及一个 `business` 的业务项目，通常会有以下两个命令。

对于`@lumin/mobile-ui`项目：

``` bash
yarn link
```

当然，你可以通过下面命令删除改符号链接：

``` bash
yarn unlink
```

对于引用 `business` 的项目：

``` bash
yarn link @lumin/mobile-ui
```

### 如何跨库联调

通过把 npm 发布到本地，然后

### 工具安装

官方 github 地址：[https://github.com/wclr/yalc](https://github.com/wclr/yalc)

``` bash
npm i yalc -g
# or
yarn global add yalc
```

`@lumin/mobile-ui`的项目：

``` bash
yalc publish
```

之后如果有更新，你需要重新推送到本地：

``` bash
yalc push
```

### `bussiness` 项目

``` bash
yalc push
```
