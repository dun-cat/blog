## 迷宫生成: (一) - 二叉树生成 
### 简介

创建迷宫会有几个策略，我们会根据优劣来选择我们想要的一种，而二叉树生成是最简单也是最直接的一种。

### 二叉树生成

利用二叉树来生成迷宫是一种非常好的方式，它在生成过程不用保存任何`状态`，这会节省内存空间并且不受`迷宫大小`限制。

### 如何构建

就像二叉的名字一样，它在每一个节点步骤时都会有两种选择：对于网格 (grid) 的每个单元格 (cell) ，都将通过抛硬币来决定`往北`还是`往西`开辟通道。

### 步骤

1. 遍历网格中的单元格：
   1. 若他们存在`顶部`或`左边`的相邻单元格；
   2. 抛硬币来选择他们中的一个，然后挖去当前的单元格和它之间的墙。
2. 最后我们把入口和出口的墙挖去，就完成了。

### 代码实现

**1.定义一个单元格**

``` javascript
class Cell {
  constructor(x, y, size) {
    this.x = x;
    this.y = y;
    this.len = size
    // top right bottom left
    this.walls = [true, true, true, true];
  }

  removeWalls(other) {
    const x = this.x - other.x;
    const y = this.y - other.y;

    function getDirection() {
      if (y > 0) return 'top'
      if (x > 0) return 'left'
      return null
    }

    switch (getDirection()) {
      case 'left':
        this.walls[3] = false
        other.walls[1] = false
        break
      case 'top':
        this.walls[0] = false
        other.walls[2] = false
        break
    }
  }
}
```

每个单元格有四堵墙，分别上、右、下、左，保存了单元格的大小和位置，并提供了`挖墙`方法。

**2.初始化迷宫**

``` javascript
function createCells(rows, cols) {
  let x;
  let y;
  let cells = []
  for (let i = 0; i < rows; i++) {
    cells[i] = [];
    for (let j = 0; j < cols; j++) {
      x = (j * CELL_SIZE);
      y = (i * CELL_SIZE);
      cells[i].push(new Cell(x, y, CELL_SIZE));
    }
  }
  return cells
}
```

我们通过遍历构建了一个二维数组，用于保存所有单元格。

**3.遍历执行挖墙操作**

``` javascript
 function binaryTreeMaze() {

  // 创建迷宫
  const mazeMatrix = createCells(rows, cols)

// 步骤 1：遍历网格中的单元格
  for (let y = 0; y < cols; y++) {
    for (let x = 0; x < rows; x++) {
      const neighbours = []

      // 步骤 1.1：若他们存在`顶部`或`左边`的相邻单元格；
      if (x > 0) neighbours.push([x - 1, y])
      if (y > 0) neighbours.push([x, y - 1])

      // 如果没有，继续遍历
      if (neighbours.length === 0) continue

      const tossCoin = Math.floor(Math.random() * neighbours.length)

      // 步骤 1.2：抛硬币来选择他们中的一个，然后挖去当前的单元格和它之间的墙。
      const [x1, y1] = neighbours[tossCoin];
      mazeMatrix[x][y].removeWalls(mazeMatrix[x1][y1]);
    }
  }
  return mazeMatrix
}
```

我们挖墙遵循一定的规则，挖墙的方向被要求只选择`上边`和`左边`的其中之一，这样我们能保证每个单元格都至少有一个前进的入口。而最后，我们的墙将形成一个树结构。

当然你可以选择其它相邻两个方向做选择，我们这选择`上左 (西北)`方向。

你不必被`二叉树`这个词所迷惑，从代码实现来，遍历的每个`子操作`都执行`两种`中的其中一种，这最终导致我们形成了一个二叉树结构的墙结构，仅此而已。

**4.开辟入口和出口**

``` javascript
mazeMatrix[0][0].walls[3] = false
mazeMatrix[cols - 1][rows - 1].walls[1] = false
```

这样，我们的二叉树迷宫算就完成了。

尽管这种迷宫有强烈的方向偏向，但依然可以作为一个使人迷惑的迷宫。

### 演示代码

你可以通过以下代码地址：[https://github.com/dun-cat/algorithms-case](https://github.com/dun-cat/algorithms-case)，获取一张通过二叉树生成的 SVG 格式的迷宫。

你可以调节`单元格大小`和迷宫的`行列数`来改变整个迷宫的大小，随着迷宫变大，手动寻路将变得非常困难。

下面的命令将生成一张 SVG 格式的图片：

``` bash
npm run maze-bt-generator
```

<img src='maze-bt.svg' style="max-wdith:100%" />

参考资料：

\> [https://hurna.io/academy/algorithms/maze_generator/index.html](https://hurna.io/academy/algorithms/maze_generator/index.html)

\> [https://en.wikipedia.org/wiki/Maze_generation_algorithm](https://en.wikipedia.org/wiki/Maze_generation_algorithm)

\> [http://weblog.jamisbuck.org/2011/2/1/maze-generation-binary-tree-algorithm#](http://weblog.jamisbuck.org/2011/2/1/maze-generation-binary-tree-algorithm#)
