## 前端性能指标及常见优化技术 
### 简介

这里的指标面向的前端是`广义的`，并非专指 Web 领域，例如：浏览器、android 应用、IOS 应用、以及 React Native 混合技术的应用。

下面的指标阐述将以 Google 的[lighthouse](https://developers.google.com/web/tools/lighthouse) web 开发工具为基础进行阐述，我们可以把它作为端侧指标衡量的依据。

### 指标

#### FCP

全称：`First Contentful Paint`，第一次内容绘制时长。

##### 分数表

| FCP time (以秒为单位) |颜色编码|
|-|-|
|0 - 1.8| 🟢 &nbsp; <span style="color:green">绿色 (快速) </span>|
|1.8 - 3| 🟠 &nbsp; <span style="color:orange">橙色 (中等) </span>|
| > 3| 🔴 &nbsp; <span style="color:red">红色 (慢) </span>|

#### TTI

全称：`Time to Interactive`，交互时长。

TTI 很重要，因为一些网站以牺牲交互性为代价来优化内容可见性，而产生不好的用户体验

##### 分数表

| TTI time (以秒为单位) |颜色编码|
|-|-|
|0 - 3.8| 🟢 &nbsp; <span style="color:green">绿色 (快速) </span>|
|3.9 – 7.3| 🟠 &nbsp; <span style="color:orange">橙色 (中等) </span>|
| > 7.3| 🔴 &nbsp; <span style="color:red">红色 (慢) </span>|

参考资料：

\> [https://web.dev/lighthouse-performance](https://web.dev/lighthouse-performance)
