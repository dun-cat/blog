## React Native 技术详解 (四) - 调试工具 Flipper 
### 介绍

[Flipper](https://fbflipper.com/) 是 Facebook 提供的一个移动端`调试工具`，它支持 iOS、Android、React Native，并提供桌面端 GUI 调试界面。

Flipper 是开箱即用的，提供了包括：`网络`、`布局和属性样式`、`持久化存储展示`、`日志`、`Hermes Debugger` 等非常有用的分析功能，这些功能都以**插件形式**提供。

Flippr 由两部分组成：

* 桌面端应用程序。
* 原生移动端SDK (Android/iOS) ，JavaScript 客户端或甚至是自己实施或在 Web 上找到的第三方客户端。

![react-native-react](react-native-react.png)

### Flipper 的工作原理

快速浏览一下 Flipper 的主要架构：
![flipper](flipper.svg)

具体过程如下：

1. Flipper 不断轮询[ADB](https://developer.android.com/studio/command-line/adb) (Android Debug Bridge：允许开发者与安卓设备通信) 和[IDB](https://fbidb.io/) (iOS Debug Bridge：创建目的与 ADB 相似) ，分别查找可用的 Android 和 iOS 设备；
2. 如果该设备正在运行启用了 Flipper 客户端的程序，则客户端会尝试连接到笔记本电脑上的 Flipper。也就是说，客户端让 Flipper 知晓这里有可以与之通信的应用。然后，Flipper 和该应用建立安全连接。不久之后，他们会就受支持插件列表进行协商。然后，Flipper 向该设备发送一条请求，列出受支持的插件。最后，设备通过 WebSocket 连接回复该列表；
3. 开发者选择其中一个已连接的应用程序/设备；
4. 开发者点击其中一个可用插件；
5. 该插件开始通过 Flipper 公开的消息总线与设备上的应用通信。插件向应用索取必要数据，然后显示在桌面设备的用户界面上。

### 集成 Flipper SDK

React Native 0.62 版本及以上，默认支持 Flipper，并且开箱即用，也就是说你只需设置依赖及是否开启 Flipper 功能，而无需手动实例化他们。下面示例展示如何使用[自动化设置](https://fbflipper.com/docs/getting-started/react-native/)。

> 只有通过 `react-native init` 创建的模板，会内置 Flipper。如果是其他框架的模板，请参考手动设置。

如果低于此版本，请参考[手动 Android 设置](https://fbflipper.com/docs/getting-started/react-native-android/)和[手动 iOS 设置](https://fbflipper.com/docs/getting-started/react-native-ios/)。默认，React Native 可能内置了一个过期的 Flipper SDK，你可以在[这里](https://github.com/facebook/flipper/tags)查看最新发布版本。

最后一个版本的 Flipper 要求 `react-native 0.69+` 。如果你的 `react-native < 0.69.0`，需要降级 `react-native-flipper` 到`0.162.0` (查看该 GitHub [issue](https://github.com/facebook/flipper/issues/4240) 详情)

#### Android

1.在 `android/gradle.properties` 里指定 `FLIPPER_VERSION` 变量，例如：FLIPPER_VERSION = 0.174.0。

并且你可以在 `android/app/build.gradle` 里，看到下面几个 Flipper 原生依赖：

``` groovy
dependencies {
  // other dependencies...

  debugImplementation("com.facebook.flipper:flipper:${FLIPPER_VERSION}") {
      exclude group:'com.facebook.fbjni'
  }

  debugImplementation("com.facebook.flipper:flipper-network-plugin:${FLIPPER_VERSION}") {
      exclude group:'com.facebook.flipper'
      exclude group:'com.squareup.okhttp3', module:'okhttp'
  }

  debugImplementation("com.facebook.flipper:flipper-fresco-plugin:${FLIPPER_VERSION}") {
      exclude group:'com.facebook.flipper'
  }
}
```

2.然后在 android 目录运行：

``` shell
./gradlew clean
```

3.在 `Application` 的 onCreate() 方法中看到[实例化代码](https://github.com/facebook/react-native/blob/a02bd0ded1d193e3fe6f0bfb961138e0f212fccc/template/android/app/src/main/java/com/helloworld/MainApplication.java#L60)。

当然，也可以手动去实例化 Flipper，下面添加了 `Layout`、`Network`、`Shared Preferences Viewer`3 个插件。

``` Java
import com.facebook.flipper.android.AndroidFlipperClient;
import com.facebook.flipper.android.utils.FlipperUtils;
import com.facebook.flipper.core.FlipperClient;
import com.facebook.flipper.plugins.inspector.DescriptorMapping;
import com.facebook.flipper.plugins.inspector.InspectorFlipperPlugin;
import com.facebook.flipper.plugins.network.NetworkFlipperPlugin;
import com.facebook.flipper.plugins.sharedpreferences.SharedPreferencesFlipperPlugin;

public class MyApplication extends Application {

  @Override
  public void onCreate() {
    super.onCreate();
    SoLoader.init(this, false);

    if (BuildConfig.DEBUG && FlipperUtils.shouldEnableFlipper(this)) {
      final FlipperClient client = AndroidFlipperClient.getInstance(this);
      //  Layout Plugin
      client.addPlugin(new InspectorFlipperPlugin(this, DescriptorMapping.withDefaults()));
      // Network Plugin
      NetworkFlipperPlugin networkFlipperPlugin = new NetworkFlipperPlugin();
      client.addPlugin(networkFlipperPlugin);
      // Shared Preferences Viewer Plugin
      client.addPlugin(new SharedPreferencesFlipperPlugin(context, "my_shared_preference_file"));

      client.start();
    }
  }
}
```

同时建议把 `FlipperDiagnosticActivity` 添加到 AndroidManifest.xml 中去，这有助于诊断集成问题和其他问题。

``` xml
<activity android:name="com.facebook.flipper.android.diagnostics.FlipperDiagnosticActivity"
        android:exported="true"/>
```

#### iOS

1.若 **react-native 版本 >= 0.69.0**，在 ios/Podfile 里可以指定一个 Flipper  版本，并通过 `FlipperConfiguration.enabled` 引入 Flipper。

``` ruby
use_react_native!(
  # Enables Flipper.
  #
  # Note that if you have use_frameworks! enabled, Flipper will not work and
  # you should disable the next line.
  :flipper_configuration => FlipperConfiguration.enabled(["Debug"], { 'Flipper' => '0.174.0' })
)
```

2.然后在 ios 项目目录执行安装依赖命令。

``` bash
 pod install --repo-update 
```

---

1.若 **react-native 版本 < 0.69.0**，在 ios/Podfile 里调用 `use_flipper` 引入 Flipper。

``` ruby
use_flipper!({ 'Flipper' => '0.174.0' })
```

在 `node_modules/react-native/scripts/cocoapods/flipper.rb` 文件中可以看到定义的 Flipper 依赖引入方法 `use_flipper_pods` 。

``` ruby
def use_flipper_pods(versions = {}, configurations: ['Debug'])
    versions['Flipper'] ||= $flipper_default_versions['Flipper']
    versions['Flipper-Boost-iOSX'] ||= $flipper_default_versions['Flipper-Boost-iOSX']
    versions['Flipper-DoubleConversion'] ||= $flipper_default_versions['Flipper-DoubleConversion']
    versions['Flipper-Fmt'] ||= $flipper_default_versions['Flipper-Fmt']
    versions['Flipper-Folly'] ||= $flipper_default_versions['Flipper-Folly']
    versions['Flipper-Glog'] ||= $flipper_default_versions['Flipper-Glog']
    versions['Flipper-PeerTalk'] ||= $flipper_default_versions['Flipper-PeerTalk']
    versions['Flipper-RSocket'] ||= $flipper_default_versions['Flipper-RSocket']
    versions['OpenSSL-Universal'] ||= $flipper_default_versions['OpenSSL-Universal']
    pod 'FlipperKit', versions['Flipper'], :configurations => configurations
    pod 'FlipperKit/FlipperKitLayoutPlugin', versions['Flipper'], :configurations => configurations
    pod 'FlipperKit/SKIOSNetworkPlugin', versions['Flipper'], :configurations => configurations
    pod 'FlipperKit/FlipperKitUserDefaultsPlugin', versions['Flipper'], :configurations => configurations
    pod 'FlipperKit/FlipperKitReactPlugin', versions['Flipper'], :configurations => configurations
    # List all transitive dependencies for FlipperKit pods
    # to avoid them being linked in Release builds
    pod 'Flipper', versions['Flipper'], :configurations => configurations
    pod 'Flipper-Boost-iOSX', versions['Flipper-Boost-iOSX'], :configurations => configurations
    pod 'Flipper-DoubleConversion', versions['Flipper-DoubleConversion'], :configurations => configurations
    pod 'Flipper-Fmt', versions['Flipper-Fmt'], :configurations => configurations
    pod 'Flipper-Folly', versions['Flipper-Folly'], :configurations => configurations
    pod 'Flipper-Glog', versions['Flipper-Glog'], :configurations => configurations
    pod 'Flipper-PeerTalk', versions['Flipper-PeerTalk'], :configurations => configurations
    pod 'Flipper-RSocket', versions['Flipper-RSocket'], :configurations => configurations
    pod 'FlipperKit/Core', versions['Flipper'], :configurations => configurations
    pod 'FlipperKit/CppBridge', versions['Flipper'], :configurations => configurations
    pod 'FlipperKit/FBCxxFollyDynamicConvert', versions['Flipper'], :configurations => configurations
    pod 'FlipperKit/FBDefines', versions['Flipper'], :configurations => configurations
    pod 'FlipperKit/FKPortForwarding', versions['Flipper'], :configurations => configurations
    pod 'FlipperKit/FlipperKitHighlightOverlay', versions['Flipper'], :configurations => configurations
    pod 'FlipperKit/FlipperKitLayoutTextSearchable', versions['Flipper'], :configurations => configurations
    pod 'FlipperKit/FlipperKitNetworkPlugin', versions['Flipper'], :configurations => configurations
    pod 'OpenSSL-Universal', versions['OpenSSL-Universal'], :configurations => configurations
end
```

2.然后同样在 ios 目录里执行安装依赖命令：

``` shell
pod install --repo-update 
```

---

3.最后在 ios 的 AppDelegate.m 里加入 Flipper 的`初始化`，让它只在 `DEBUG` 下执行，并添加需要使用的插件。下面添加了 `Layout`、`Network`、`Shared Preferences Viewer`3 个插件。

``` objc
#import <FlipperKit/FlipperClient.h>
#import <FlipperKitLayoutPlugin/FlipperKitLayoutPlugin.h>
#import <FlipperKitLayoutComponentKitSupport/FlipperKitLayoutComponentKitSupport.h>
#import <FlipperKitUserDefaultsPlugin/FKUserDefaultsPlugin.h>
#import <FlipperKitNetworkPlugin/FlipperKitNetworkPlugin.h>
#import <SKIOSNetworkPlugin/SKIOSNetworkAdapter.h>

@implementation AppDelegate

- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions
{
  #if DEBUG
  FlipperClient *client = [FlipperClient sharedClient];
  SKDescriptorMapper *layoutDescriptorMapper = [[SKDescriptorMapper alloc] initWithDefaults];
  [FlipperKitLayoutComponentKitSupport setUpWithDescriptorMapper: layoutDescriptorMapper];
  // Layout Plugin
  [client addPlugin: [[FlipperKitLayoutPlugin alloc] initWithRootNode: application
                                                 withDescriptorMapper: layoutDescriptorMapper]];
  // Shared Preferences Viewer Plugin
  [client addPlugin:[[FKUserDefaultsPlugin alloc] initWithSuiteName:nil]];
  // NetWork Plugin
  [client addPlugin: [[FlipperKitNetworkPlugin alloc] initWithNetworkAdapter:[SKIOSNetworkAdapter new]]];
  [client start];
  #endif
  ...
}
@end
```

参考资料：

\> [https://developers.facebook.com/blog/post/2022/08/25/flipper-and-js-why-we-added-javascript-support-to-a-mobile-debugging-platform/](https://developers.facebook.com/blog/post/2022/08/25/flipper-and-js-why-we-added-javascript-support-to-a-mobile-debugging-platform/)
