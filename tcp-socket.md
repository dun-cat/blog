---
layout: post
title: 基于TCP协议的Socket
date: 2016-12-13
tags: ["socket","tcp","网络编程"]
---

 ### 简介

 Socket是一套用来执行符合TCP/IP协议标准的网络编程接口。有时一个IP地址和一个端口号也可以叫做插口(socket),这个术语最早出现在TCP规范RFC793里面。对于网络I/O来说就是用来建立tcp链接的工具，它是一个统称，不同操作系统原理相同，但接口函数却不太一样。上面是广义，狭义上术语翻译过来就叫套接字，是定义一个具备Internet通信的基本单位(例如：我们可以说建立一个套接字)。

但基本思路按照Unix/Linux系统的哲学"一切皆文件"来实现的。也就是说它的过程类似本地文件读写操作：

"打开open -> 读写write/read -> 关闭close"

通过Socket提供的函数在创建一个连接时，会返回一个短整数类型的描述符(Descriptor)来表示当前这个套接字。在windoes系统叫做句柄。后面的操作也都将把这个套接字的标识作为参数带入函数中。

### 基本流程

基本Socket(既不考虑多线程和有线程池的情况)流程图如下：

[![socket_flowchart](http://www.lumin.tech:12345/wp-content/uploads/2016/12/socket_flowchart.jpg)](socket_flowchart.jpg)

上面的关闭操作是大致流程，具体请看一下内容。

### TCP连接

#### TCP协议详情

 tcp所处位置和协议格式如下：

![tcp_message_segment](tcp_message_segment.jpg)

**几个基本概念**

报文段(message segment)：在网络层逻辑划分上，每一次传输的包含TCP的消息内容，叫做TCP报文段。这一段会被继续往下被封装到IP数据报里面。我们做TCP协议分析都是基于报文段来分析的。

报文段划分为两部分：一个是首部，一个是数据。

16位源/目的端口号：从这可以看出端口号的上限是2<sup>16 </sup>大小，也就是65535个。所以相对此的开放式操作系统(linux, windows and so on)的端口号上限也就是这样的。

32位序列号：表示当前报文段中第一个字节的序列号。TCP数据流中每个字节都有一个序列号。所以它能够对2<sup>32</sup>=4GB数据进行编号。

32位确认序列号：发给对方确认的序列号，用来告诉对方本地已经接收上一次的数据，所以回个信。确认号 = 上一次接收的对方序列号 + 1。

4位首部长度：网络上也有说是数据偏移，但是理解上还是首部长度说法好理解。它表示报文段起始到数据偏移大小，也就是首部的长度。之所以只有4位，因为协议规定它表示的偏移长度是32位，而不是字面上的1位。所以它能够表达的最大首部长度是4x15x32=60字节。正常的长度是20个字节。

保留位：6位的保留位，既没啥用。

6个标志位：

|  标志位    | 说明   |
|  :----  | :---- |
| URG  | 紧急位，为1时，首部中的紧急指针有效 |
| ACK  | (Acknowledgment)确认位，为1时，首部中的确认号有效 |
| PSH  | 推位，为1时，要求把数据尽快交给应用程序 |
| RST  | 复位标志，为1时，复位连接，一般在出错或关闭连接时使用 |
| SYN  | (Synchronization) 同步位，在建立连接时使用，当SYN=1而ACK=0时，表明这是一个连接请求报文段。对方若同意建立连接，在发回的报文段中使SYN=1和ACK=1 |
| FIN  | 结束位，为1时，表示发送方完成了数据发送 |


 #### TCP连接的建立

 客户端和服务器端TCP连接的建立也叫三次握手(three-way handshake)，通过三次数据传输完成。

[![tcp_connect](http://www.lumin.tech:12345/wp-content/uploads/2016/12/TCP_connect.jpg)](TCP_connect.jpg)

 SYN会占用一个序列号

#### TCP的数据传输

[![tcp_translate](http://www.lumin.tech:12345/wp-content/uploads/2016/12/TCP_translate.jpg)](TCP_translate.jpg)

#### TCP连接的关闭

[![tcp_close](http://www.lumin.tech:12345/wp-content/uploads/2016/12/TCP_close.jpg)](TCP_close.jpg)

 FIN也会占用一个序列号
 