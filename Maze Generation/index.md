## Maze Generation 
### 简介

创建迷宫会有几个策略，我们会根据优劣来选择我们想要的一种。

### 二叉树生成

利用二叉树来生成迷宫是一种非常好的方式，它在生成过程不用保存任何状态，这会节省内存空间并且不受迷宫大小限制。

#### 如何构建

就像二叉的名字一样，它在每一个节点步骤时都会有两种选择：对于网格 (grid) 的每个单元格 (cell) ，都将通过抛硬币来决定`往北`还是`往西`开辟通道。

#### 步骤

1. 遍历网格中的单元格：
   1. 若他们存在北边或者西边的相邻单元格
   2. 抛硬币来连接他们中的一个。
2. 完事。

参考资料：

\> [https://hurna.io/academy/algorithms/maze_generator/index.html](https://hurna.io/academy/algorithms/maze_generator/index.html)

\> [https://en.wikipedia.org/wiki/Maze_generation_algorithm](https://en.wikipedia.org/wiki/Maze_generation_algorithm)
