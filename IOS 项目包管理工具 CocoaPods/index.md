## IOS 项目包管理工具 CocoaPods 
### 配置文件 Podfile

主项目的 pod 的依赖管理配置文件只有 Podfile一个文件，并存放在项目根目录，基础写法如下：

``` shell
platform :ios, '8.0'
use_frameworks!

target 'MyApp' do
  pod 'AFNetworking', '~> 2.6'
  pod 'ORStackView', '~> 3.0'
  pod 'SwiftyJSON', '~> 2.3'
end
```

指定一个 `target` 名字 `MyApp`，它和 XCode 的编译 `target` 保持一致。

版本号管理上遵循[Semantic Versioning](https://semver.org/lang/zh-CN/)和[RubyGems Versioning Policies](https://guides.rubygems.org/patterns/#semantic-versioning)

### 依赖库引入

**1.指定仓库引入**

``` shell
# 默认 master 分支
pod 'Alamofire', :git => 'https://github.com/Alamofire/Alamofire.git'
# 指定 dev 分支
pod 'Alamofire', :git => 'https://github.com/Alamofire/Alamofire.git', :branch => 'dev'
# 指定 tag 标签
pod 'Alamofire', :git => 'https://github.com/Alamofire/Alamofire.git', :tag => '3.1.1'
# 指定 commit
pod 'Alamofire', :git => 'https://github.com/Alamofire/Alamofire.git', :commit => '0f506b1c45'
```

**2.指定本地路径引入**

``` shell
pod 'Alamofire', :path => '~/Documents/Alamofire'
```

### 安装

``` shell
pod install
```

### 更新

``` shell
pod update
```

### pod install VS pod update

pod install 运行会发生的事：

* 每次运行 `pod install` 命令时，都会把新的 pod 版本写入 `Podfile.lock` 。
* `pod install` 运行，会根据 `Podfile.lock` 锁定的版本去下载 pod，不会去校验是否有新的 pod 版本，也就是说对于已安装 pod 不会做任何处理。

pod update 运行会发生的事：

* 根据 `Podfile.lock` 的版本设定规则，尝试把 pod 更新到最新版本。

所以如果不想更新已有的 pod ，而添加新的 pod  情况下，请使用 `pod install` 。

值得注意的是 `pod install` 会锁定 `Podfile.lock` 里的版本，所以仓库应该包含 `Podfile.lock` 文件，提供项目一个稳定的依赖版本配置，让团队其它成员能够确信无误的运行项目。

### 指定安装源 source

可以在全局范围指定安装源，也可以对指定的 pod 指定安装源。

``` shell
# 全局
source 'https://github.com/CocoaPods/Specs.git'
#指定 pod 源
pod 'PonyDebugger', :source => 'https://github.com/CocoaPods/Specs.git'
```
