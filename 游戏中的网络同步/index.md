## 游戏中的网络同步 
### 简介

网络同步 (network synchronization) 本质上是`数据同步`。当我们再和别人打电话的时候，其实是把我们的声音数据同步到电话的另一端，这也是网络同步的场景。

当我们讨论网络同步的时候，通常关注的焦点是数据同步的`即时性`、`表现一致性`以及在不同场景下的`同步策略`。尤其在电子游戏领域作为`网络游戏`的重要基础技术。

### P2P 同步模型

P2P 可以认为是最简单的同步模型。A 把数据发送给 B，B 收到数据后处理，再把处理后的数据发送给 A。对于两端来说无需 Server 去处理数据，直连互传就行。在 P2P 架构下几乎没有任何反作弊能力。

### 局域网下的同步模型

这里可以从 P2P 上衍生出来，当有两端以上的设备需要进行数据同步的时候，在局域网内把数据发送到一个固定 IP 的主机上，然后由它转发消息到各自的用户端上去。

这样的架构不需要单独都维护一个服务器，任何一个客户端都可以是 Sever，能够比较方便的支持局域网内对战，也能节省服务器的运行与开发成本。

虽说也是CS架构，如果 Host 主机不做任何 server 端的校验逻辑，那么其本质上还是 P2P 模型，只不过所有的客户端可以把消息统一发送到一个 IP，Host 再进行转发，这种方式我们称其为 Packet Server。

### CS 架构下的同步模型

在 CS 架构下的同步模型，把一部分逻辑交给服务器端处理，而渲染和无需服务器处理的逻辑交给客户端处理。

### 同步策略

在弱交互游戏中 (比如回合制游戏) ，所有玩家只需要同步所有指令即可。当所有玩家的状态保持一致，才能执行客户端的指令动作。这种游戏采用的同步方式与计算机网络中的停等协议 (stop-and-wait-type) 非常相似，是一种很自然也很简单的同步模型。

缺点也是很明显，如果有一个玩家网络出现问题，那么所有其它玩家就必须进入等待，或因被迫放弃游戏。这样的游戏在棋牌类的比较常见，因为对同步的即时性的要求并不高。

### CS 同步数据面临的问题

**1.逻辑在服务器执行还是在客户端执行？**

若逻辑都在服务器端执行，那么可以保证客户端的表现是一致的，即状态同步。这样客户端无法作弊，但是对于服务器来说压力就过大了。

若逻辑都在客户端执行，那么可以充分发挥客户端的运算性能，服务器只需要做简单的转发即可，但坏处是很容易在本地进行作弊。

**2.我们要发什么数据进行同步？**

如果发送每个对象当前的状态，那么在一个包含大量的角色游戏里，就会大规模的占用网络带宽，造成数据拥塞、丢包等等问题。

如果发送玩家指令，那这个指令是要服务器执行还是服务器转发？而且对于大型多人在线游戏又没必要处理所有不相关的玩家信息，同样浪费网络资源。

**3.我们应该采用何种协议？**

TCP、UDP 还是 Http？

### 帧同步

帧同步范指`保证每帧 (逻辑帧) 输入一致`的一系列算法。

传统实现有`帧锁定`、`乐观帧锁定`、`lockstep`、`bucket`同步等等。

> LockStep 由军事语境引入，用来表示齐步行军，队伍中的所有人都执行一致的动作步伐。

但凡满足“每帧输入一致”的方法皆可以归纳为帧同步类别，至于要不要回滚？服务端要不要跑一套完整逻辑？操作要不要是键盘鼠标？还是高阶命令？客户端要不要像视频播放器一样保证平滑缓存 1-2 帧？或者要不要保证平滑加一层显示对象的坐标插值？这些都是具体优化手段。

参考资料：

\> [https://ieeexplore.ieee.org/document/1457585](https://ieeexplore.ieee.org/document/1457585)

\> [网络同步在游戏历史中的发展变化](https://mp.weixin.qq.com/s?__biz=MzkzNTIxMjMyNg==&mid=2247491556&idx=1&sn=7101a907cb2d0df3d237ef0752638282&source=41#wechat_redirect)

\> [https://zhuanlan.zhihu.com/p/165293116](https://zhuanlan.zhihu.com/p/165293116)

\> [https://zhuanlan.zhihu.com/p/363690394](https://zhuanlan.zhihu.com/p/363690394)
