## 调度算法 
### 调度算法类型

* **先来先服务** (FIFO - First-Come, First-Served)：最简单的调度算法，按照任务到达的顺序执行。
* **最短作业优先** (SJF - Shortest-Job-First)：优先执行预计执行时间最短的任务。
* **优先级调度** (Priority Scheduling)：根据任务的优先级决定执行顺序。高优先级的任务先执行。
* **轮转调度** (RR - Round-Robin Scheduling)：每个任务轮流获得一定时间片，适用于时间共享系统。
* **多级队列** (Multilevel Queue)：将任务分入不同的队列，每个队列有自己的调度算法。
* **多级反馈队列** (Multilevel Feedback Queue)：结合多种方法，根据任务的行为动态调整其优先级。
* **实时调度**：用于实时操作系统，确保关键任务能够在规定时间内完成。

### First-Come, First-Served Scheduling

参考资料：

\> [https://zh.wikipedia.org/wiki/调度_(计算机)](https://zh.wikipedia.org/wiki/调度_(计算机))
