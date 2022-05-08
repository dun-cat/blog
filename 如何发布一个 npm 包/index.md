## 如何发布一个 npm 包 
### 发布前

`npm` 包的发布需要提供两个关键信息：`registry` 和 `access token`。

`registry` 指定发布的仓库的地址；`access token` 规定访问权限，是一串16进制的字符串。

#### 登录

CLI 里对 token 的操作以及包发布（automation token 除外）都必须预先登录。

使用`npm login`，然后输入密码邮箱即可。可以用过 `npm whoami` 来验证登录状态。

#### Access Token

`access token` 的可以通过 `Web` 来生成，也可以通过 `CLI` 生成。每次调用 `npm login` 都会生成一条新的 token。

##### 类型

Token 分为三种类型：`read only`、`publish`、`automation`；

* `read only`：只允许安装和分发；
* `publish`：允许安装、分发、修改、发布以及对帐户所有权；
* `automation`：主要用于 `CI` 环境的包发布并且`不能在 CLI 里创建 automation token`，可以避免双因子认证（2FA）的密码输入；

##### 查看

生成的 token 可以通过 `npm token` 或 `npm token list` 命令来查看：

``` shell
➜  lerna-learning git:(main) ✗ npm token
┌────────┬─────────┬────────────┬──────────┬────────────────┐
│ id     │ token   │ created    │ readonly │ CIDR whitelist │
├────────┼─────────┼────────────┼──────────┼────────────────┤
│ f09eeb │ d60c64… │ 2021-02-19 │ no       │                │
├────────┼─────────┼────────────┼──────────┼────────────────┤
│ 788d8e │ 7516cd… │ 2021-02-19 │ no       │                │
├────────┼─────────┼────────────┼──────────┼────────────────┤
│ 77cda0 │ 39fe9d… │ 2021-02-19 │ no       │                │
└────────┴─────────┴────────────┴──────────┴────────────────┘
```

##### 生成

通过 `npm token create`，然后输入密码即可创建一个`可发布`的 token。

``` shell
 npm token create [--read-only] [--cidr=<cidr-ranges>]
 ```

``` shell
➜  lerna-learning git:(main) ✗ npm token create
npm password: 
┌────────────────┬──────────────────────────────────────┐
│ token          │ d554bd1e-3ec2-49d9-a0c7-36e967efcd87 │
├────────────────┼──────────────────────────────────────┤
│ cidr_whitelist │                                      │
├────────────────┼──────────────────────────────────────┤
│ readonly       │ false                                │
├────────────────┼──────────────────────────────────────┤
│ automation     │ false                                │
├────────────────┼──────────────────────────────────────┤
│ created        │ 2021-02-19T06:12:45.787Z             │
└────────────────┴──────────────────────────────────────┘
```

Token 有几个属性：

* `cidr_whitelist`：可以指定IP范围内的地址可访问；
* `readonly`：只读权限；
* `automation`：是否是自动化 token；
* `create`：token 创建时间；

> token 生成后只展示一次，后面将永远不再展示；通过 CLI 方式`不能`生成 `automation token`，即 automation 永远是 false；

##### 移除

使用 `npm token revoke <token|id>`，即可移除 token：

``` shell
➜  lerna-learning git:(main) ✗ npm token revoke f09eeb
Removed 1 token
```

#### Registry

指定一个目标仓库，用于存储发布包，默认`npm registry`的值为：<https://registry.npmjs.org>。

npm 的 `registry` 使用的是 [couch database](https://couchdb.apache.org/) 数据库，可以直接使用 RESTful API 方式访问数据。

##### 修改 registry

在 `package.json` 添加 `publishConfig.registry`，把发布包指向一个内部仓库：

``` json
{
  "publishConfig":{"registry":"http://my-internal-registry.local"}
}
```

如果没有指定 publishConfig，默认使用 npm registry 属性值来发布。

### 发布（Publish）

直接执行 `npm publish` 即可：

``` shell
npm publish [<tarball>|<folder>] [--tag <tag>] [--access <public|restricted>] [--otp otpcode] [--dry-run]
```

需要注意的是：

* 如果发布的包是一个 scope 包，那么必须指定 `--access=public`。
* 如果远程仓库开启了双因子认证（2FA），那么必须指定 `--otp optcode`。

``` shell
➜  lerna-learning git:(main) ✗ npm publish --access=public
npm notice 
npm notice 📦  @dun-cat/lerna-learning@1.0.1
npm notice === Tarball Contents === 
npm notice 616B package.json               
npm notice 28B  index.js                   
npm notice === Tarball Details === 
npm notice name:          @dun-cat/lerna-learning                 
npm notice version:       1.0.1                                   
npm notice package size:  990 B                                   
npm notice unpacked size: 2.4 kB                                  
npm notice shasum:        3097ce97e0aa99f9d1001bc8697b529775a304cc
npm notice integrity:     sha512-zVB5Cl0/HP3zM[...]LP7bQKnBGZAOQ==
npm notice total files:   9                                       
npm notice 
+ @dun-cat/lerna-learning@1.0.1
```

Tip：每次发布 `package.json` 里的 `version` 版本号必须进行`累加`，否则发布失败。

#### 累加版本号

通过 `npm version` 命令可以实现修改版本号，

``` shell
 npm version [<newversion> | major | minor | patch | premajor | preminor | prepatch | prerelease [--preid=<prerelease-id>] | from-git]
```

每次 `npm verion` 执行，会提交一条新的 `git commit` 记录，并打上版本号 `tag`，可以通过 `--no-git-tag-version` 来取消此操作：

``` shell
npm --no-git-tag-version version patch
```

### CI/CD 环境下的自动发布

#### 使用 automation token

在 CI/CD 环境直接使用 `automation token`来避免双因子认证的密码输入。

可以将 token 配置到 `.npmrc` 文件里。如果项目根目录包含 .npmrc 则优先使用它。对于已安装 npm 的用户，用户目录会包含一份作用全局的 `~/.npmrc` 文件，通常使用它作为 CD 会更为方便。

``` xml
//registry.npmjs.org/:_authToken=${NPM_TOKEN}
```

> 请使用`环境变量`赋值`_authToken`，切记不要直接把 token 直接写到 .npmrc 文件里去。

#### 配置 package.json

我们把`版本号累加`及`包发布`配置到 `package.json` 的 `scripts` 来执行：

``` json
{
  "scripts": {
    "pub": "npm run updateVersion && npm publish --access=public",
    "updateVersion": "npm --no-git-tag-version version patch"
  },
}
```
