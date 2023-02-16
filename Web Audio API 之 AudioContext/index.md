## Web Audio API 之 AudioContext 
### 简介

引用 [mdn](https://developer.mozilla.org/zh-CN/docs/Web/API/AudioContext) 的简介：

AudioContext 接口表示由链接在一起的音频模块构建的音频处理图，每个模块由一个 AudioNode 表示。音频上下文控制它包含的节点的创建和音频处理或解码的执行。

在做任何其他操作之前，您需要创建一个 AudioContext 对象，因为所有事情都是在上下文中发生的。建议创建一个 AudioContext 对象并复用它，而不是每次初始化一个新的 AudioContext 对象，并且可以对多个不同的音频源和管道同时使用一个 AudioContext 对象。

参考资料：

\> [https://developer.mozilla.org/zh-CN/docs/Web/API/AudioContext](https://developer.mozilla.org/zh-CN/docs/Web/API/AudioContext)
