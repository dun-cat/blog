## RequestIdleCallback 
### 简介

`一帧渲染页面完成后`，用的时间比较短，有剩余时间，并不会影响下一帧的渲染，此时 requestIdleCallback 内的回调才会被执行。

如果浏览器一直处于繁忙状态，requestIdleCallback 内的回调函数将永远不会被执行。所以需要在第二个参数设置 `timeout` 。如果回调函数执行超时了，那么回调函数会在下一次空闲时期被强制执行。
