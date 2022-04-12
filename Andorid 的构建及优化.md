## Andorid 的构建及优化 
### 构建任务


构建指定任务：

``` bash
./gradlew task-name
```

通常 android gradle 插件会帮你生成主工程的构建任务，被命名为`assemble`。

所以你可以执行以下命令去构建主工程：

``` bash
./gradlew assemble
```

### Gradle 增量构建（Incrementtal Build）

Gradle 官网叫做`Up-to-date checks`，也被叫做`增量构建`，方便理解使用增量构建作为主称。

任何一个构建工具，都应该能够避免做一些已完成的事情。一旦你的源文件已经编译过，他们就不应该重复编译。跳过重复编译的步骤，可以节省大量时间。

但他们需要在编译的输入输出发生改变时，重新编译。例如：源文件发生修改或输出文件移除。

Gradle 支持开箱即用的`增量构建`功能。每当一个任务（Task）打印`UP-TO-DATE`文本时，表明该任务的构建被略过并使用构建缓存。

构建时，在 android studio 下的 build 日志可以看到下面的一些使用构建缓存的`UP-TO-DATE`标识：

``` bash
> Task :react-native-community_geolocation:preBuild UP-TO-DATE
> Task :react-native-community_geolocation:preDebugBuild UP-TO-DATE
```

下面的图展示一个任务的构建`输入`（inputs）和`输出`（outputs）：

![task_inputs_outputs.png](task_inputs_outputs.png)

一个输入的显著特征是它能够影响一个或多个输出，不同的`字节码`（bytecode）生成依赖于源文件的内容和代码运行所在的 Java 运行时（Java runtime）的最小版本。

> Java 源文件生成的字节码以`.class`为后缀。

作为增量构建的一部分，Gradle 会检测最后一次构建的输入和输出是否改变。若没有改变，则会在下次构建跳过任务动作。要注意的是任务至少有一个输出，否则增量编译不会起作用。

### 构建缓存（build cache）

#### 清除构建缓存

Android 插件的`clean`任务可以清除项目的`build/`目录，与之类似，可以运行 cleanBuildCache 任务来清除项目的构建缓存。

``` bash
./gradlew cleanBuildCache
```

当然你可以使用下面命令删除构建目录：

``` bash
./gradlew clean
```

如果报以下错误：

``` bash
FAILURE: Build failed with an exception.

* What went wrong:
Could not initialize class org.codehaus.groovy.runtime.InvokerHelper
```

请确认 Gradle 版本在`6.3-rc-4`以上，你可以在[这里](https://github.com/gradle/gradle/issues/12599)找到该问题的讨论。

### 构建速度优化

高构建速度的一般过程如下：

1. 采取一些可以使大多数 android studio 项目立即受益的措施，优化 build 配置；
2. 对构建进行性能剖析，确定并诊断一些对您的项目或工作站来说比较棘手的瓶颈问题。

参考文献：

\> [https://developer.android.com/studio/build/build-cache](https://developer.android.com/studio/build/build-cache)

\> [https://docs.gradle.org/current/userguide/more_about_tasks.html#sec:up_to_date_checks](https://docs.gradle.org/current/userguide/more_about_tasks.html#sec:up_to_date_checks)

\> [https://developer.android.com/studio/build/optimize-your-build](https://developer.android.com/studio/build/optimize-your-build)
