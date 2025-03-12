## Lerna 多包管理工具的萌新探索 
### What is Lerna ?

[Lerna](https://lerna.js.org/) 是一个多包管理工具，最初是为了`解决跨库调试`的问题，后面衍生出比较多的执行命令方便调试和管理。

按照 lerna 文件组织结构，所有的 `package` 包都在 `packages` 目录里，外部只保留 `package.json` 配置文件即可。package 包是个完整的 `npm` 项目结构。

### 文件组织结构

``` xml
my-lerna-repo/
  package.json
  packages/
    package-1/
      package.json
    package-2/
      package.json
```

### 常用的关键性命令

#### 初始化或升级项目 (init)

`创建`一个 lerna 项目或把已存在的 git 仓库`升级`为 lerna 项目：

``` shell
lerna init
```

Lerna 会做两件事：

* 在 `package.json` 里的 `devDependency` 加入 lerna。
* 创建 `lerna.json` 配置文件，存储当前 lerna 项目的`版本号`

#### 创建新包 (package)

``` shell
lerna create newPackageName
```

#### 安装依赖 (bootstrap)

所有包在自己的目录里安装依赖，即所有包都有对应其 `package.json` 的 `node_modules` 目录，同时 `symlink` 各个包和二进制包

``` shell
lerna bootstrap
```

该命令会做以下几件事：

1. `npm install` 每个软件包的所有外部依赖项；
2. 将所有 `packages` 相互依赖的Lerna `link` 在一起；
3. 在所有已安装的包里执行 `npm run prepublish` ；
4. 在所有已安装的包里执行 `npm run prepare` ；

##### --ignore

忽略某些依赖的安装，可以指定命令 `--ignore` 项，当然也可以配置在 `lerna.json` 更为通用。

#### 添加依赖

``` bash
lerna add <package>[@version] [--dev] [--exact] [--peer]
```

添加操作可以使用所有过滤标签

``` bash
# Adds the module-1 package to the packages in the 'prefix-' prefixed folders
lerna add module-1 packages/prefix-*

# Install module-1 to module-2 in devDependencies
lerna add module-1 --scope=module-2 --dev

# Install module-1 to module-2 in peerDependencies
lerna add module-1 --scope=module-2 --peer
```

#### 提升安装依赖 Hoisting

每个包都有自己的 `package.json`，安装依赖时都会在每个包里生成 `node_modules` 目录，通常这些包会有`很多重复的依赖`。Hoisting 的解决方法是把`依赖关系提升到最顶层`的 `node_modules` 目录，以此减少开发和构建副本包带来的额外时间和空间。

##### --hoist [glob]

该配置项把依赖安装至`根目录`，可以指定一个 `glob` 来避免在所有 `packages` 里生效。依赖项中的任何二进制文件都将 `link` 到 `node_modules/.bin/` 目录中，如果未指定 glob，则默认为** (安装所有依赖至根目录) 。

##### --nohoist [glob]

``` shell
lerna bootstrap --hoist --nohoist = babel- *
```

可以指定 不进行 hoist 的依赖

``` shell
lerna bootstrap --hoist
```

``` json
{
  "version": "0.0.0",
  "command": {
    "bootstrap": {
      "ignore": "component-*"
    }
  }
}
```

#### 运行各个包的 script

运行 `lerna run` 命令可以执行每个包 `package.json` 里的 `script` 同名脚本：

``` shell
lerna run start # 执行所有包含有 start 的脚本
```

packages/xxx-pk1/package.json

``` json

{
  "scripts":{
    "start": "some start command..."
  }
}
```

packages/xxx-pk2/package.json

``` json
{
  "scripts":{
    "start": "some start command..."
  }
}
```

 `xxx-pk1` 和 `xxx-pk2` 里的 start 都将被执行。

##### 指定过滤标记

通过[过滤标记](https://github.com/lerna/lerna/tree/main/core/filter-options)过滤不需要执行的包。

``` shell
lerna run start --scope xxx-pk1 # 只执行 xxx-pk1 里的 start 脚本
```

> 这里需要注意的是 `xxx-pk1` 需要和 `package.json` 里的 `name` 字段一致，不然在目前的版本会运行错误。

##### 并行执行标记：`--parallel`

如果有些包执行的比较久，可以指定 `--parallel` 使用 `child processes` 来并行处理每个包。

``` shell
lerna run start --scope xxx-pk1 --scope xxx-pk2 --parallel
```

> 建议在使用 --parallel 标志时，同时使用 --scope 标志限定范围，过多的子进程会损耗 shell 性能。

### Lerna 包版本管理

`修改包版本号`及` git push `通过 `version` 命令来实现：

``` shell
lerna version 1.0.1 # explicit
lerna version patch # semver keyword
lerna version       # select from prompt(s)
```

这里 lerna 会做几件事情：

1. 确认所有包都是最新包，这里主要通过 `git` 来确认；
2. 弹出版本选择提示框，来指定版本号；
3. 修改包元数据以反映新版本，并在 `root` 包和每个 `package` 包中运行适当的[生命周期脚本](https://github.com/lerna/lerna/tree/main/commands/version#lifecycle-scripts)；
4. `Commit` 那些已更改的 `package.json` 文件并打上版本 `tag` ；
5. `Push` 到 `git` 远程仓库；

> 在执行 version 命令前，需要执行一次 `git commit` 来记录此次更改。如果没有执行，version 命令将不认为这次有新版本需要发布，`强制发布` (--force-publish) 例外。

#### 强制修改所有版本号: `--force-publish`

``` shell
lerna version --force-publish = package-2，package-4 ＃强制对所有软件包进行版本控制
lerna version --force-publish
```

#### 跳过 Commit/Tag 操作: `--no-git-tag-version`

默认会提交被修改版本号的 package.json 文件并打版本 tag。

#### 跳过 Push 操作: `--no-push`

默认会执行 git push 操作，通过 `--no-push` 可以禁止：

``` shell
lerna version --no-push
```

### 两种模式：锁定 vs 独立

Lerna 项目有两种模式：Fixed/Locked mode (default) | Independent mode，他们对应包版本号管理的两种方式。

#### Fixed/Locked mode (default)

这模式下所有包的版本同步一个主版本号，即 `lerna.json` 里的 `version` 

``` json
{
  "version": "0.0.0"
}
```

#### Independent mode

该模式下，每个包可以独立维护自己的版本号，如果要指定独立运行模式，在 lerna.json 里指定 version 配置如下：

``` json
{
  "version": "independent"
}
```

### 在 CI 环境下的优化配置

#### --amend

正常情况，version 命令会生成一次新的 commit 来记录 package.json 里版本号的改变，通过 --amend 选项将改变合并到当前的 commit，并标记 tag，同时会忽略 push 操作。

``` shell
lerna version --amend
# commit message is retained, and `git push` is skipped.
```

#### --yes

取消弹框确认提示，所有操作默认通过。

### Lerna 的包发布 (Publish)

``` shell
lerna publish              # 发布上次发版发生改变的包
lerna publish from-git     # explicitly publish packages tagged in the current commit
lerna publish from-package # explicitly publish packages where the latest version is not present in the registry
```

> Lerna 永不发布被标记为私有的包 (`"private": true in the package.json`)

#### 每个包的 Publish 配置

每个包通过指定 `package.json` 的 `publishConfig` 字段来改变 `publish` 的一些行为

##### access

默认情况被 scope 的包 (例：@my-scope/app) 都是访问受限的，需要指定公共访问。

``` json
{
    "publishConfig": {
    "access": "public"
  }
}
```

##### registry

指定包的发布仓库。

``` json
{
"publishConfig": {
    "registry": "http://my-awesome-registry.com/"
  }
}
 
```

### lerna.json 配置

``` json
{
  "version": "1.1.3",
  "npmClient": "npm",
  "command": {
    "publish": {
      "ignoreChanges": ["ignored-file", "*.md"],
      "message": "chore(release): publish",
      "registry": "https://npm.pkg.github.com"
    },
    "bootstrap": {
      "ignore": "component-*",
      "npmClientArgs": ["--no-package-lock"]
    }
  },
  "packages": ["packages/*"]
}
```

* `version` : 当前项目的版本；
* `npmClient` ：指定安装客户端，模式采用 `npm install` 安装，可以使用 `yarn` 代理安装，提升安装速度；
  
* `command.publish.ignoreChanges` ：指定一组 `globs` 数组，在发布时，忽略打到包内；
* `command.publish.message` ：发布会`生成`或`覆盖` (假如有--amend 选项) 一条 `commit`，这里可以自定义 `commit` 消息；
* `command.publish.registry` ：指定发布仓库；
  
* `command.bootstrap.ignore` ：指定一组 `globs` 数组，来忽略某些依赖的安装；
* `command.bootstrap.npmClientArgs:` ：可以给 `npm install` 传递变量；
* `packages` ：指定一组 `globs` 数组，来确定包的位置，默认值：\["packages/*"\]；
