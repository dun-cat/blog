## React 源码分析（一）：调度器（Scheduler） 
### 数据结构

调度器根据`优先级`做任务调度，`优先队列`符合这一特征，通常采用`堆`数据结构来实现。

React 中保存着两个优先队列：`任务队列` (taskQueue) 和 `计时器队列`(timerQueue)，并都采用了`最小堆`存储。

### 方法

#### advanceTimers

此函数的任务是把小于当前时间的任务加入到任务队列里去。

``` javascript
function advanceTimers(currentTime) {}
```

1. 循环遍历`timerQueue`；
2. 获取`timerQueue`第 1 个元素，比较`timer`的`开始时间` 和 `当前时间`：
   1. 若 `timer.startTime` $<$ `currentTime`，则把计时器索引的 task 移交到 `taskQueue`;
3. 直到`timerQueue`为空，完成。

#### handleTimeout


#### flushWork

扩展阅读：

\> [https://wicg.github.io/scheduling-apis/](https://wicg.github.io/scheduling-apis/)