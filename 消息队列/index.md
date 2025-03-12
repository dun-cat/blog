## 消息队列 
### 简介

在计算机科学中，`消息队列` (message queues) 和`邮箱` (mailboxes) 是软件工程的组件，通常用于`进程间通信` (IPC) 或同一进程内的`线程间通信`。他们使用`队列`进行消息传递。 `群组通信系统` (GCS) 提供类似的功能。

消息队列范式是`发布/订阅模式`的兄弟，更典型的是`面向消息中间件系统`的一部分。大多数消息系统在他们的 API 中同时支持`发布者/订阅者`和`消息队列模型`，例如：`Java 消息服务` (JMS)。

### 应用范围

消息队列实现有在操作系统或应用程序内部运行，此类只在当前系统使用。

也有其他实现允许在不同的操作系统之间传递消息，可能连接多个应用程序和多个操作系统。这些消息队列系统通常提供弹性功能，以确保在系统发生故障时消息不会“丢失”。这种消息队列软件 (也称为面向消息的中间件) 的商业实现示例包括 `IBM MQ` (以前称为 MQ 系列) 和 `Oracle Advanced Queuing` (AQ) 。有一个称为 `Java Message Service` 的 Java 标准，它有几个专有的和免费软件实现。

目前，有很多消息队列有很多开源的实现，包括 JBoss Messaging、JORAM、Apache ActiveMQ、Sun Open Message Queue、RabbitMQ、IBM MQ、Apache Qpid、Apache RocketMQ和HTTPSQS。

参考资料：

\> [https://www.geeksforgeeks.org/difference-between-message-queues-and-mailboxes/](https://www.geeksforgeeks.org/difference-between-message-queues-and-mailboxes/)

\> [https://en.wikipedia.org/wiki/Message_queue](https://en.wikipedia.org/wiki/Message_queue)
