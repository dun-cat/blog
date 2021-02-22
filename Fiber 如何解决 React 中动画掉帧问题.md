## Fiber 如何解决 React 中动画掉帧问题 
### 问题

在浏览器环境里，JS运算、页面元素布局及绘制都在一个主线程完成。如果动画占用大量运算时间，那么页面的渲染必然会被阻塞，自然而然帧率(FPS)就会降下来。

> 帧率：每秒能呈现渲染多少次画面，用于衡量动画流畅度的指标。通常 fps < 30 就能明显感知卡顿现象，流畅舒适的动画帧率在60左右。所以渲染一次画面所需的JS运算时间`力求` < 16ms。

### 解决思路

JS运算占用渲染时间，可考虑`JS运算的时机`和`减少运算量`。基本思路是把JS运算切割成多个任务，分批完成各个任务。在完成一部分任务后，把执行的控制权交还给主线程，让其进行渲染工作。渲染完成后，再进行未完成的计算任务。

那么我们要解决的问题便是任务分配的问题，这里有个优化前后的例子：[react-fiber-vs-stack-demo](https://claudiopro.github.io/react-fiber-vs-stack-demo)。

### React 中动画卡顿原因

在 state 或 props 更新时，React 的工作主要包含两个阶段：

`协调阶段（Reconciler）`：React 15 及更早版本被叫做 `Stack Reconciler` ，是自顶向下的递归算法。遍历新数据生成新的Virtual DOM，通过 Diff 算法，`找出需要更新的元素`，放到更新队列中去。关键是这个过程并不能被中断。

`渲染阶段（Renderer）`：遍历更新队列，调用渲染宿主环境的 API, 将对应元素更新渲染。在浏览器中，就是更新对应的DOM元素。除浏览器外，渲染环境还可以是 Native、WebGL 等等。

在协调阶段，为了找出需要更新的元素，花费了大量时间，所以渲染阶段被延后执行，这是导致卡顿原因。

### Fiber 的处理思路

#### 改变数据结构及遍历算法

Fiber 在 React 生成的 Virtual Dom 基础上增加的一层`链表`数据结构，把`递归遍历`转成`循环遍历`。配合 [requestIdleCallback API](https://developer.mozilla.org/zh-CN/docs/Web/API/Window/requestIdleCallback), 实现任务`拆分`、`中断`与`恢复`。

> window.requestIdleCallback()方法将在浏览器的空闲时段内调用的函数排队。这使开发者能够在主事件循环上执行后台和低优先级工作，而不会影响延迟关键事件，如动画和输入响应。函数一般会按先进先调用的顺序执行，然而，如果回调函数指定了执行超时时间timeout，则有可能为了在超时前执行函数而打乱执行顺序。

之所以改变数据结构，是因为`递归遍历不能将工作分解`为增量单位，React一直保持迭代，直到处理完所有组件并且堆栈为空为止。

那么 React 如何实现算法而无需递归遍历树呢？它使用`单链列表树`遍历算法,这样可以`暂停遍历`并`阻止堆栈增长`。链表遍历算法可以`异步运行`，使用指针返回到其暂停工作的节点。

#### Fiber Node
React 的单链表树的节点被叫做 Fiber Node，它包含一些关键信息。以往的 Virtual Dom Node 是包含状态信息的 SateNode，现在会增加节点关系信息。
``` javascript
{
  // 浏览器环境下指 DOM 节点
  stateNode,
  child,
  return,
  sibling
}
```

#### Fiber Node Tree

React 会存在两个 Fiber 树实例：current tree 和 workInProgress tree。

`current tree `建立在第一个渲染器上，并且与 Virtaul DOM 具有一对一的关系，`workInProgress tree` 即将用于渲染的树。

当调用新的渲染器时，React 将 workInProgress 使用协调算法在新实例上开始工作，以遍历组件树并查找必须在何处进行更改。

### Fiber 的渲染流程

Fiber 渲染分成两个阶：`render` 阶段和 `commit` 阶段。

#### Render 阶段

在 React 第一次渲染会生成 Fiber 节点树，并在后续的更新被重用。详细点来说渲染阶段会生成一个部分节点标记了 `side effects` 的 Fiber 节点树，在源码中叫做 `workInProgress tree` 或 `finishedWork`。side effects 描述了在下一个 commit 阶段需要完成的工作。

这个阶段的任务是确定需要插入、更新或删除哪些节点，以及哪些组件需要调用其生命周期方法。

这个阶段的特点是可以`异步执行`，中间的执行可以中断，可以根据`可用时间`来处理一个或多个 Fiber 节点，并且用户不可见。

执行会有几个场景：

1. 完成部分工作后，交出控制权处理其它事情，后面控制权回来再继续处理任务。
2. 超过时，当前的任务会被终止，直到下一次继续。
3. 如果有更高优先级的任务，那当前任务会被终止。什么样的任务具有更高优先级的呢？像用户的交互输入优先级是比较高的。

#### Commit 阶段

这个阶段会用到几个数据结构：

1. render 阶段生成 workInProgress tree，
2. 被叫做 current tree 的 fiber 节点树，它直接用于更新UI。
3. effects list，由 render 阶段生成的列表。

这个阶段的任务是更新UI，并回调一些生命周期方法，包含以下一些操作：

* 在标记了 `Snapshot effect` 的节点上调用 `getSnapshotBeforeUpdate` 生命周期方法；
* 在标记了 `Deletion effect` 的节点上调用 `componentWillUnmount` 生命周期方法；
* 执行所有 DOM 插入，更新和删除；
* 将 workInProgress tree 树设置为 current 树；
* 在标记了 `Placement effect` 的节点上调用 `componentDidMount` 生命周期方法；
* 在标记了 `Update effect` 的节点上调用 `componentDidUpdate` 生命周期方法；

#### 总体的流程

``` shell
--- working asynchronously ---------------------------------------------------------------------------
| ------- Fiber ---------------    ------- Fiber ---------------    ------ Fiber ---------------     |
| | beginWork -> completeWork | -> | beginWork -> completeWork | -> |beginWork -> completeWork | ... |
| -----------------------------   ------------------------------    ----------------------------     |
------------------------------------------------------------------------------------------------------
                      ↓↓↓
-----------------------------------------------------------------------
| commitAllWork(flush side effects computed in the above to the host) |
-----------------------------------------------------------------------
```


