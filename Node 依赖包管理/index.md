## Node 依赖包管理 
### yarn

#### 配置文件

yarn 的配置文件是`.yarnrc`。

你可以如下配置多个仓库源：

``` text
registry: https://registry.npm.taobao.org
'@lumin:registry': https://npm.lumin.tech
```

#### 列出配置

``` shell
yarn config list
```

#### 修改配置

``` shell
yarn config set registry https://registry.npmjs.org/
```

#### 安装依赖

yarn 通过 `yarn install` 来安装依赖：

``` bash
yarn install

# 或简写

yarn
```

默认安装的是 `dependencies` 依赖，你可以通过`-D`或者`--dev`来添加 `devDependencies` 依赖：

``` bash
yarn add -D tailwindcss
# 或
yarn add --dev tailwindcss
```

全局安装：

``` bash
yarn global add nodemon
```

对于锁版本文件 `yarn.lock` 的生成规则如下：

* 若 `yarn.lock` 文件已提供，并能够满足在 `package.json` 下的所有依赖，则 `yarn.lock` 记录的精确版本号会被安装，并且 `yarn.lock` 文件不发生改变，Yarn 并不会去检查新的版本号；
* 若 `yarn.lock` 文件未提供或不能满足在 `package.json` 下的所有依赖 (例如：你手动添加了一个依赖至 `package.json` ) ，则 Yarn 会寻找`最新的可用版本`，并更新 `yarn.lock` 文件。

### pnpm

全局按照

``` bash
pnpm add -g mathjax-node
```

### 切换 node 版本 nvm

``` shell

```

### 切换依赖源 nrm

使用 [nrm](https://github.com/Pana/nrm) 可以切换多个依赖源。

#### 安装

``` shell
yarn global add nrm
```

#### 列出源

 `nrm ls` 命令可以列出所有可以切换的依赖源：

``` shell
$ nrm ls

* npm ---------- https://registry.npmjs.org/
  yarn --------- https://registry.yarnpkg.com/
  tencent ------ https://mirrors.cloud.tencent.com/npm/
  cnpm --------- https://r.cnpmjs.org/
  taobao ------- https://registry.npmmirror.com/
  npmMirror ---- https://skimdb.npmjs.com/registry/
```

#### 切换源

 `nrm use` 命令切换到指定依赖源：

``` shell
$ nrm use cnpm  //switch registry to cnpm

    Registry has been set to: http://r.cnpmjs.org/
```
