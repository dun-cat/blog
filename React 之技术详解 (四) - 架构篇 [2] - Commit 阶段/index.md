## React 之技术详解 (四) - 架构篇 [2] - Commit 阶段 
### 流程概览

上一章我们介绍了，`commitRoot 方法` 是 `commit 阶段` 工作的起点。fiberRootNode 会作为传参。

``` javascript
commitRoot(root);
```

在 rootFiber.firstEffect 上保存了一条需要执行`副作用`的 Fiber 节点的`单向链表`effectList，这些 Fiber 节点的 updateQueue 中保存了变化的 props。

这些`副作用`对应的 DOM 操作在 `commit 阶段` 执行。

除此之外，一些生命周期钩子 (比如 componentDidXXX) 、hook (比如 useEffect) 需要在 `commit 阶段` 执行。

 `commit 阶段` 的主要工作 (即 Renderer 的工作流程) 分为三部分：

* before mutation 阶段 (执行 DOM 操作前)
* mutation 阶段 (执行 DOM 操作)
* layout 阶段 (执行 DOM 操作后)

你可以从这里看到[commit 阶段](https://github.com/facebook/react/blob/1fb18e22ae66fdb1dc127347e169e73948778e5a/packages/react-reconciler/src/ReactFiberWorkLoop.new.js)的完整代码。

在 `before mutation 阶段` 之前和 `layout 阶段` 之后还有一些额外工作，涉及到比如 useEffect 的触发、优先级相关的重置、 ref 的绑定/解绑。

这些对我们当前属于超纲内容，为了内容完整性，在这节简单介绍。

#### before mutation 之前

 `commitRootImpl 方法` 中直到第一句 if (firstEffect !== null) 之前属于 before mutation 之前。

我们大体看下他做的工作，现在你还不需要理解他们：

``` javascript
do {
    // 触发 useEffect 回调与其他同步任务。由于这些任务可能触发新的渲染，所以这里要一直遍历执行直到没有任务
    flushPassiveEffects();
  } while (rootWithPendingPassiveEffects !== null);

  // root 指 fiberRootNode
  // root.finishedWork 指当前应用的 rootFiber
  const finishedWork = root.finishedWork;

  // 凡是变量名带 lane 的都是优先级相关
  const lanes = root.finishedLanes;
  if (finishedWork === null) {
    return null;
  }
  root.finishedWork = null;
  root.finishedLanes = NoLanes;

  // 重置 Scheduler 绑定的回调函数
  root.callbackNode = null;
  root.callbackId = NoLanes;

  let remainingLanes = mergeLanes(finishedWork.lanes, finishedWork.childLanes);
  // 重置优先级相关变量
  markRootFinished(root, remainingLanes);

  // 清除已完成的 discrete updates，例如：用户鼠标点击触发的更新。
  if (rootsWithPendingDiscreteUpdates !== null) {
    if (
      !hasDiscreteLanes(remainingLanes) &&
      rootsWithPendingDiscreteUpdates.has(root)
    ) {
      rootsWithPendingDiscreteUpdates.delete(root);
    }
  }

  // 重置全局变量
  if (root === workInProgressRoot) {
    workInProgressRoot = null;
    workInProgress = null;
    workInProgressRootRenderLanes = NoLanes;
  } else {
  }

  // 将 effectList 赋值给 firstEffect
  // 由于每个 fiber 的 effectList 只包含他的子孙节点
  // 所以根节点如果有 effectTag 则不会被包含进来
  // 所以这里将有 effectTag 的根节点插入到 effectList 尾部
  // 这样才能保证有 effect 的 fiber 都在 effectList 中
  let firstEffect;
  if (finishedWork.effectTag > PerformedWork) {
    if (finishedWork.lastEffect !== null) {
      finishedWork.lastEffect.nextEffect = finishedWork;
      firstEffect = finishedWork.firstEffect;
    } else {
      firstEffect = finishedWork;
    }
  } else {
    // 根节点没有 effectTag
    firstEffect = finishedWork.firstEffect;
  }
```

可以看到，`before mutation` 之前主要做一些`变量赋值`，`状态重置`的工作。

这一长串代码我们只需要关注最后赋值的 firstEffect，在 commit 的三个子阶段都会用到他。

#### layout 之后

接下来让我们简单看下 layout 阶段执行完后的代码，现在你还不需要理解他们：

``` javascript
const rootDidHavePassiveEffects = rootDoesHavePassiveEffects;

// useEffect 相关
if (rootDoesHavePassiveEffects) {
  rootDoesHavePassiveEffects = false;
  rootWithPendingPassiveEffects = root;
  pendingPassiveEffectsLanes = lanes;
  pendingPassiveEffectsRenderPriority = renderPriorityLevel;
} else {}

// 性能优化相关
if (remainingLanes !== NoLanes) {
  if (enableSchedulerTracing) {
    // ...
  }
} else {
  // ...
}

// 性能优化相关
if (enableSchedulerTracing) {
  if (!rootDidHavePassiveEffects) {
    // ...
  }
}

// ...检测无限循环的同步任务
if (remainingLanes === SyncLane) {
  // ...
} 

// 在离开 commitRoot 函数前调用，触发一次新的调度，确保任何附加的任务被调度
ensureRootIsScheduled(root, now());

// ...处理未捕获错误及老版本遗留的边界问题


// 执行同步任务，这样同步任务不需要等到下次事件循环再执行
// 比如在 componentDidMount 中执行 setState 创建的更新会在这里被同步执行
// 或 useLayoutEffect
flushSyncCallbackQueue();

return null;
```

你可以在[这里](https://github.com/facebook/react/blob/1fb18e22ae66fdb1dc127347e169e73948778e5a/packages/react-reconciler/src/ReactFiberWorkLoop.new.js#L2195)看到这段代码。

主要包括三点内容：

* useEffect 相关的处理。我们会在讲解layout阶段时讲解。
* 性能追踪相关。
源码里有很多和 `interaction` 相关的变量。他们都和追踪 React 渲染时间、性能相关，在 Profiler API 和 DevTools 中使用。

你可以在这里看到 [interaction](https://gist.github.com/bvaughn/8de925562903afd2e7a12554adcdda16) 的定义。

在 `commit 阶段` 会触发一些生命周期钩子 (如 componentDidXXX) 和 hook (如 useLayoutEffect、useEffect) 。
在这些回调方法中可能触发新的更新，新的更新会开启新的 render-commit 流程。考虑如下Demo：

 `useLayoutEffect` 会在 `layout 阶段` 同步执行回调。回调中我们触发了状态更新 setCount(randomNum)，这会重新调度一个同步任务。

该任务会在在如上 commitRoot 倒数第二行代码处被同步执行。

``` javascript
flushSyncCallbackQueue();
```

### before mutation 阶段

在本节正式开始前，让我们复习下这一章到目前为止所学的。

Renderer 工作的阶段被称为 `commit 阶段` 。commit 阶段可以分为三个子阶段：

* before mutation 阶段 (执行 DOM 操作前)

* mutation 阶段 (执行 DOM 操作)

* layout 阶段 (执行 DOM 操作后)

本节我们看看 before mutation 阶段 (执行 DOM 操作前) 都做了什么。

#### 概览

before mutation 阶段的代码很短，整个过程就是遍历 effectList 并调用 `commitBeforeMutationEffects 函数` 处理。

这部分[源码](https://github.com/facebook/react/blob/1fb18e22ae66fdb1dc127347e169e73948778e5a/packages/react-reconciler/src/ReactFiberWorkLoop.new.js)在这里。为了增加可读性，示例代码中删除了不相关的逻辑。

``` javascript
// 保存之前的优先级，以同步优先级执行，执行完毕后恢复之前优先级
const previousLanePriority = getCurrentUpdateLanePriority();
setCurrentUpdateLanePriority(SyncLanePriority);

// 将当前上下文标记为 CommitContext，作为 commit 阶段的标志
const prevExecutionContext = executionContext;
executionContext |= CommitContext;

// 处理 focus 状态
focusedInstanceHandle = prepareForCommit(root.containerInfo);
shouldFireAfterActiveInstanceBlur = false;

// beforeMutation 阶段的主函数
commitBeforeMutationEffects(finishedWork);

focusedInstanceHandle = null;
```

我们重点关注 beforeMutation 阶段的主函数 commitBeforeMutationEffects 做了什么。

#### commitBeforeMutationEffects

大体代码逻辑：

``` javascript
function commitBeforeMutationEffects() {
  while (nextEffect !== null) {
    const current = nextEffect.alternate;

    if (!shouldFireAfterActiveInstanceBlur && focusedInstanceHandle !== null) {
      // ...focus blur 相关
    }

    const effectTag = nextEffect.effectTag;

    // 调用 getSnapshotBeforeUpdate
    if ((effectTag & Snapshot) !== NoEffect) {
      commitBeforeMutationEffectOnFiber(current, nextEffect);
    }

    // 调度 useEffect
    if ((effectTag & Passive) !== NoEffect) {
      if (!rootDoesHavePassiveEffects) {
        rootDoesHavePassiveEffects = true;
        scheduleCallback(NormalSchedulerPriority, () => {
          flushPassiveEffects();
          return null;
        });
      }
    }
    nextEffect = nextEffect.nextEffect;
  }
}
```

整体可以分为三部分：

1. 处理 DOM 节点渲染/删除后的 autoFocus、blur 逻辑；
2. 调用 `getSnapshotBeforeUpdate 生命周期钩子` ；
3. 调度 useEffect。

我们讲解下2、3两点。

#### 调用 getSnapshotBeforeUpdate

commitBeforeMutationEffectOnFiber 是 commitBeforeMutationLifeCycles 的别名。

在该方法内会调用 `getSnapshotBeforeUpdate`，你可以在[这里](https://github.com/facebook/react/blob/1fb18e22ae66fdb1dc127347e169e73948778e5a/packages/react-reconciler/src/ReactFiberCommitWork.new.js#L222)看到这段逻辑。

从 React 16 开始，componentWillXXX 钩子前增加了 `UNSAFE_前缀` 。

究其原因，是因为 Stack Reconciler 重构为 Fiber Reconciler 后，render 阶段的任务可能中断/重新开始，对应的组件在render 阶段的生命周期钩子 (即componentWillXXX) 可能`触发多次`。

这种行为和 React 15 不一致，所以标记为 `UNSAFE_` 。更详细的解释参照这里

为此，React 提供了`替代`的生命周期钩子 `getSnapshotBeforeUpdate` 。

我们可以看见，getSnapshotBeforeUpdate 是在 `commit 阶段` 内的 `before mutation 阶段` 调用的，由于 `commit 阶段` 是同步的，所以不会遇到多次调用的问题。

#### 调度 useEffect

在这几行代码内，scheduleCallback 方法由 `Scheduler 模块` 提供，用于以某个优先级异步调度一个回调函数。

``` javascript
// 调度 useEffect
if ((effectTag & Passive) !== NoEffect) {
  if (!rootDoesHavePassiveEffects) {
    rootDoesHavePassiveEffects = true;
    scheduleCallback(NormalSchedulerPriority, () => {
      // 触发 useEffect
      flushPassiveEffects();
      return null;
    });
  }
}
```

在此处，被`异步调度`的回调函数就是触发 useEffect 的方法 `flushPassiveEffects` 。

我们接下来讨论 `useEffect 如何被异步调度`，以及为什么要异步 (而不是同步) 调度。

##### 如何异步调度

在 `flushPassiveEffects 方法` 内部会从`全局变量`rootWithPendingPassiveEffects 获取 effectList。

在我们讲到，effectList中 保存了需要执行`副作用`的 Fiber 节点。其中副作用包括：

* 插入 DOM 节点 (Placement)
* 更新 DOM 节点 (Update)
* 删除 DOM 节点 (Deletion)

除此外，当一个 `FunctionComponent` 含有 `useEffect` 或 `useLayoutEffect`，他对应的 Fiber 节点也会被赋值  effectTag。你可以从[这里](https://github.com/facebook/react/blob/1fb18e22ae66fdb1dc127347e169e73948778e5a/packages/react-reconciler/src/ReactHookEffectTags.js)看到 hook 相关的 effectTag。

在 `flushPassiveEffects 方法` 内部会遍历 rootWithPendingPassiveEffects (即 effectList) 执行 effect 回调函数。

如果在此时直接执行，rootWithPendingPassiveEffects === null。

那么 rootWithPendingPassiveEffects 会在何时赋值呢？

在上一节 `layout 之后` 的代码片段中会根据 rootDoesHavePassiveEffects === true ? 决定是否赋值 rootWithPendingPassiveEffects。

``` javascript
const rootDidHavePassiveEffects = rootDoesHavePassiveEffects;
if (rootDoesHavePassiveEffects) {
  rootDoesHavePassiveEffects = false;
  rootWithPendingPassiveEffects = root;
  pendingPassiveEffectsLanes = lanes;
  pendingPassiveEffectsRenderPriority = renderPriorityLevel;
}
```

所以整个 `useEffect 异步调用` 分为三步：

1. `before mutation 阶段` 在 `scheduleCallback` 中调度 `flushPassiveEffects` ；
2. `layout 阶段` 之后将 `effectList` 赋值给 `rootWithPendingPassiveEffects` ；
3. `scheduleCallback` 触发 `flushPassiveEffects`，`flushPassiveEffects` 内部遍历 `rootWithPendingPassiveEffects` 。

##### 为什么需要异步调用？

摘录自 React 文档[effect 的执行时机](https://zh-hans.reactjs.org/docs/hooks-reference.html)：

> 与 componentDidMount、componentDidUpdate 不同的是，在浏览器完成布局与绘制之后，传给 useEffect 的函数会延迟调用。这使得它适用于许多常见的副作用场景，比如设置订阅和事件处理等情况，因此`不应`在函数中`执行阻塞浏览器更新屏幕的操作`。

可见，`useEffect 异步执行` 的`主要原因`是防止同步执行时阻塞浏览器渲染。

#### 总结

经过本节学习，我们知道了在 before mutation 阶段，会遍历 effectList，依次执行：

1. 处理 DOM 节点渲染/删除后的 autoFocus、blur 逻辑；
2. 调用 `getSnapshotBeforeUpdate 生命周期` 钩子；
3. 调度 `useEffect` 。

### mutation 阶段

终于到了执行 DOM 操作的 `mutation 阶段` 。

#### 概览

类似 before mutation 阶段，`mutation 阶段` 也是`遍历 effectList`，执行函数。这里执行的是 commitMutationEffects。

``` javascript
nextEffect = firstEffect;
do {
  try {
      commitMutationEffects(root, renderPriorityLevel);
    } catch (error) {
      invariant(nextEffect !== null, 'Should be working on an effect.');
      captureCommitPhaseError(nextEffect, error);
      nextEffect = nextEffect.nextEffect;
    }
} while (nextEffect !== null);
```

#### commitMutationEffects

代码如下，你可以在[这里](https://github.com/facebook/react/blob/1fb18e22ae66fdb1dc127347e169e73948778e5a/packages/react-reconciler/src/ReactFiberWorkLoop.new.js#L2392)看到 `commitMutationEffects 源码` ：

``` javascript
function commitMutationEffects(root: FiberRoot, renderPriorityLevel) {
  // 遍历 effectList
  while (nextEffect !== null) {

    const effectTag = nextEffect.effectTag;

    // 根据 ContentReset effectTag 重置文字节点
    if (effectTag & ContentReset) {
      commitResetTextContent(nextEffect);
    }

    // 更新 ref
    if (effectTag & Ref) {
      const current = nextEffect.alternate;
      if (current !== null) {
        commitDetachRef(current);
      }
    }

    // 根据 effectTag 分别处理
    const primaryEffectTag =
      effectTag & (Placement | Update | Deletion | Hydrating);
    switch (primaryEffectTag) {
      // 插入 DOM
      case Placement: {
        commitPlacement(nextEffect);
        nextEffect.effectTag &= ~Placement;
        break;
      }
      // 插入 DOM 并 更新 DOM
      case PlacementAndUpdate: {
        // 插入
        commitPlacement(nextEffect);

        nextEffect.effectTag &= ~Placement;

        // 更新
        const current = nextEffect.alternate;
        commitWork(current, nextEffect);
        break;
      }
      // SSR
      case Hydrating: {
        nextEffect.effectTag &= ~Hydrating;
        break;
      }
      // SSR
      case HydratingAndUpdate: {
        nextEffect.effectTag &= ~Hydrating;

        const current = nextEffect.alternate;
        commitWork(current, nextEffect);
        break;
      }
      // 更新 DOM
      case Update: {
        const current = nextEffect.alternate;
        commitWork(current, nextEffect);
        break;
      }
      // 删除 DOM
      case Deletion: {
        commitDeletion(root, nextEffect, renderPriorityLevel);
        break;
      }
    }

    nextEffect = nextEffect.nextEffect;
  }
}
```

 `commitMutationEffects` 会`遍历 effectList`，对每个 Fiber 节点执行如下三个操作：

1. 根据 ContentReset effectTag 重置文字节点；
2. 更新 ref；
3. 根据 effectTag 分别处理，其中 effectTag 包括(Placement | Update | Deletion | Hydrating)；

我们关注步骤三中的 Placement | Update | Deletion。Hydrating 作为服务端渲染相关，我们先不关注。

#### Placement effect

当 Fiber 节点含有 Placement effectTag，意味着该 Fiber 节点对应的 DOM 节点需要插入到页面中。

调用的方法为 commitPlacement，你可以在[这里](https://github.com/facebook/react/blob/970fa122d8188bafa600e9b5214833487fbf1092/packages/react-reconciler/src/ReactFiberCommitWork.new.js#L1156)看到 `commitPlacement` 源码。

该方法所做的工作分为三步：

1. 获取父级 DOM 节点。其中 finishedWork 为传入的 Fiber 节点。

``` javascript
const parentFiber = getHostParentFiber(finishedWork);
// 父级 DOM 节点
const parentStateNode = parentFiber.stateNode;
```

2. 获取 Fiber 节点的 DOM 兄弟节点。

``` javascript
const before = getHostSibling(finishedWork);
```

3. 根据 DOM 兄弟节点是否存在决定调用 `parentNode.insertBefore` 或 `parentNode.appendChild` 执行 DOM 插入操作。

``` javascript
// parentStateNode是否是rootFiber
if (isContainer) {
  insertOrAppendPlacementNodeIntoContainer(finishedWork, before, parent);
} else {
  insertOrAppendPlacementNode(finishedWork, before, parent);
}
```

值得注意的是，`getHostSibling`(获取兄弟 DOM 节点) 的执行很耗时，当在同一个父 Fiber 节点下依次执行多个插入操作，`getHostSibling 算法` 的复杂度为指数级。

这是由于 Fiber 节点不只包括 HostComponent，所以 Fiber 树和渲染的 DOM 树节点并不是一一对应的。要从 Fiber 节点找到 DOM 节点很可能跨层级遍历。

考虑如下例子：

``` jsx
function Item() {
  return <li><li>;
}

function App() {
  return (
    <div>
      <Item/>
    </div>
  )
}
ReactDOM.render(<App/>, document.getElementById('root'));
```

对应的 Fiber 树和 DOM 树结构为：

``` text
// Fiber 树
          child      child      child       child
rootFiber -----> App -----> div -----> Item -----> li

// DOM 树
#root ---> div ---> li
```

当在 div 的子节点 Item 前插入一个新节点 p，即 App 变为：

``` jsx
function App() {
  return (
    <div>
      <p></p>
      <Item/>
    </div>
  )
}
```

对应的 Fiber 树和 DOM 树结构为：

``` text
// Fiber 树
          child      child      child
rootFiber -----> App -----> div -----> p 
                                       | sibling       child
                                       | -------> Item -----> li 
// DOM 树
#root ---> div ---> p
             |
               ---> li
```

此时 DOM 节点 p 的兄弟节点为 li，而 Fiber 节点 p 对应的兄弟 DOM 节点为：

``` javascript
fiberP.sibling.child
```

即 fiber p 的兄弟 fiber Item 的子 fiber li。

#### Update effect

当 Fiber 节点含有 Update effectTag，意味着该 Fiber 节点需要更新。调用的方法为 `commitWork`，他会根据 Fiber.tag 分别处理。你可以在[这里](https://github.com/facebook/react/blob/970fa122d8188bafa600e9b5214833487fbf1092/packages/react-reconciler/src/ReactFiberCommitWork.new.js#L1441)看到 `commitWork 源码` 。

这里我们主要关注 `FunctionComponent` 和 `HostComponent` 。

##### FunctionComponent mutation

当 fiber.tag 为 FunctionComponent，会调用 `commitHookEffectListUnmount` 。该方法会遍历 effectList，执行所有 useLayoutEffect hook 的销毁函数。你可以在[这里](https://github.com/facebook/react/blob/970fa122d8188bafa600e9b5214833487fbf1092/packages/react-reconciler/src/ReactFiberCommitWork.new.js#L314)看到 `commitHookEffectListUnmount 源码` 。

所谓`销毁函数`，见如下例子：

``` jsx
 useLayoutEffect(() => {
  // ...一些副作用逻辑

  return () => {
    // ...这就是销毁函数
  }
})
```

你不需要很了解 `useLayoutEffect`，我们会在下一节详细介绍。你只需要知道在 `mutation 阶段` 会执行 `useLayoutEffect 的销毁函数` 。

##### HostComponent mutation

当 fiber.tag 为 `HostComponent`，会调用 `commitUpdate`，你可以在[这里](https://github.com/facebook/react/blob/970fa122d8188bafa600e9b5214833487fbf1092/packages/react-dom/src/client/ReactDOMHostConfig.js#L423)看到 `commitUpdate 源码` 。

最终会在 `updateDOMProperties` 中将 `render 阶段` completeWork 中为 Fiber 节点赋值的 `updateQueue` 对应的内容渲染在页面上。

``` javascript
for (let i = 0; i < updatePayload.length; i += 2) {
  const propKey = updatePayload[i];
  const propValue = updatePayload[i + 1];

  // 处理 style
  if (propKey === STYLE) {
    setValueForStyles(domElement, propValue);
  // 处理 DANGEROUSLY_SET_INNER_HTML
  } else if (propKey === DANGEROUSLY_SET_INNER_HTML) {
    setInnerHTML(domElement, propValue);
  // 处理 children
  } else if (propKey === CHILDREN) {
    setTextContent(domElement, propValue);
  } else {
  // 处理剩余 props
    setValueForProperty(domElement, propKey, propValue, isCustomComponentTag);
  }
}
```

#### Deletion effect

当 Fiber 节点含有 Deletion effectTag，意味着该 Fiber 节点对应的 DOM 节点需要从页面中删除。调用的方法为commitDeletion，你可以在[这里](https://github.com/facebook/react/blob/970fa122d8188bafa600e9b5214833487fbf1092/packages/react-reconciler/src/ReactFiberCommitWork.new.js#L1421)看到 `commitDeletion 源码` 。

该方法会执行如下操作：

1. 递归调用 Fiber 节点及其子孙 Fiber 节点中 `fiber.tag` 为 `ClassComponen` 的 `componentWillUnmount 生命周期` 钩子，从页面移除 Fiber 节点对应 DOM 节点；
2. 解绑 ref；
3. 调度 useEffect 的销毁函数。

#### 总结

从这节我们学到 `mutation 阶段` 会`遍历effectList`，依次执行 `commitMutationEffects` 。

该方法的主要工作为“根据 effectTag 调用不同的处理函数处理 Fiber。

### layout 阶段

该阶段之所以称为 layout，因为该阶段的代码都是在 DOM 渲染完成 ( `mutation 阶段完成` ) 后执行的。

该阶段触发的生命周期钩子和 hook `可以直接访问`到`已经改变后的 DOM`，即该阶段是可以参与 DOM layout 的阶段。

#### 概览

与前两个阶段类似，`layout 阶段` 也是`遍历 effectList`，执行函数。

具体执行的函数是 `commitLayoutEffects` 。

``` javascript
root.current = finishedWork;

nextEffect = firstEffect;
do {
  try {
    commitLayoutEffects(root, lanes);
  } catch (error) {
    invariant(nextEffect !== null, "Should be working on an effect.");
    captureCommitPhaseError(nextEffect, error);
    nextEffect = nextEffect.nextEffect;
  }
} while (nextEffect !== null);

nextEffect = null;
```

#### commitLayoutEffects

代码如下，你可以在[这里](https://github.com/facebook/react/blob/970fa122d8188bafa600e9b5214833487fbf1092/packages/react-reconciler/src/ReactFiberWorkLoop.new.js#L2302)看到 `commitLayoutEffects 源码` ：

``` javascript
function commitLayoutEffects(root: FiberRoot, committedLanes: Lanes) {
  while (nextEffect !== null) {
    const effectTag = nextEffect.effectTag;

    // 调用生命周期钩子和 hook
    if (effectTag & (Update | Callback)) {
      const current = nextEffect.alternate;
      commitLayoutEffectOnFiber(root, current, nextEffect, committedLanes);
    }

    // 赋值 ref
    if (effectTag & Ref) {
      commitAttachRef(nextEffect);
    }

    nextEffect = nextEffect.nextEffect;
  }
}
```

 `commitLayoutEffects` 一共做了两件事：

1. commitLayoutEffectOnFiber (调用生命周期钩子和 hook 相关操作) ；
2. commitAttachRef (赋值 ref) 。

#### commitLayoutEffectOnFiber

 `commitLayoutEffectOnFiber 方法` 会根据 `fiber.tag` 对不同类型的节点分别处理。

你可以在[这里](https://github.com/facebook/react/blob/970fa122d8188bafa600e9b5214833487fbf1092/packages/react-reconciler/src/ReactFiberCommitWork.new.js#L459)看到 `commitLayoutEffectOnFiber 源码` ( commitLayoutEffectOnFiber 为别名，方法原名为 commitLifeCycles) 。

对于 `ClassComponent`，他会通过 current === null ? 区分是 mount 还是 update，调用 `componentDidMount` 或 `componentDidUpdate` 。

触发状态更新的 `this.setState` 如果赋值了第二个参数回调函数，也会在此时调用。

``` javascript
this.setState({ xxx: 1 }, () => {
  console.log("i am update~");
});
```

对于 `FunctionComponent` 及相关类型，他会调用 `useLayoutEffect hook` 的`回调函数`，调度 useEffect 的`销毁`与`回调函数`。

> `相关类型`指特殊处理后的 `FunctionComponent`，比如 ForwardRef、React.memo 包裹的 FunctionComponent。

``` javascript
  switch (finishedWork.tag) {
    // 以下都是 FunctionComponent 及相关类型
    case FunctionComponent:
    case ForwardRef:
    case SimpleMemoComponent:
    case Block: {
      // 执行 useLayoutEffect 的回调函数
      commitHookEffectListMount(HookLayout | HookHasEffect, finishedWork);
      // 调度 useEffect 的销毁函数与回调函数
      schedulePassiveEffects(finishedWork);
      return;
    }
```

你可以从[这里](https://github.com/facebook/react/blob/1fb18e22ae66fdb1dc127347e169e73948778e5a/packages/react-reconciler/src/ReactFiberCommitWork.new.js#L465)看到这段代码。

在上一节介绍时介绍过，`mutation 阶段` 会执行 useLayoutEffect hook 的`销毁函数`。

结合这里我们可以发现，useLayoutEffect hook 从上一次更新的`销毁函数调用`到本次更新的回调函数调用是`同步执行`的。

而 useEffect 则需要`先调度`，在 `Layout 阶段` 完成后再`异步执行`。

这就是 `useLayoutEffect` 与 `useEffect` 的区别。

对于 HostRoot，即 rootFiber，如果赋值了第三个参数回调函数，也会在此时调用。

``` javascript
ReactDOM.render(<App />, document.querySelector("#root"), function() {
  console.log("i am mount~");
});
```

#### commitAttachRef

 `commitLayoutEffects` 会做的第二件事是 `commitAttachRef`，你可以在[这里](https://github.com/facebook/react/blob/970fa122d8188bafa600e9b5214833487fbf1092/packages/react-reconciler/src/ReactFiberCommitWork.new.js#L823)看到 `commitAttachRef 源码` 。

``` javascript
function commitAttachRef(finishedWork: Fiber) {
  const ref = finishedWork.ref;
  if (ref !== null) {
    const instance = finishedWork.stateNode;

    // 获取DOM实例
    let instanceToUse;
    switch (finishedWork.tag) {
      case HostComponent:
        instanceToUse = getPublicInstance(instance);
        break;
      default:
        instanceToUse = instance;
    }

    if (typeof ref === "function") {
      // 如果ref是函数形式，调用回调函数
      ref(instanceToUse);
    } else {
      // 如果ref是ref实例形式，赋值ref.current
      ref.current = instanceToUse;
    }
  }
}
```

代码逻辑很简单：获取 DOM 实例，更新 ref。

#### current Fiber 树切换

至此，整个 `layout 阶段` 就结束了。

在结束本节的学习前，我们关注下这行代码：

``` javascript
root.current = finishedWork;
```

你可以在[这里](https://github.com/facebook/react/blob/970fa122d8188bafa600e9b5214833487fbf1092/packages/react-reconciler/src/ReactFiberWorkLoop.new.js#L2022)看到这行代码。

在我们介绍过，workInProgress Fiber 树在 `commit 阶段` 完成渲染后会变为 current Fiber 树。这行代码的作用就是切换 fiberRootNode 指向的 current Fiber树。

那么这行代码为什么在这里呢？ (在 `mutation 阶段` 结束后，`layout 阶段` 开始前。)

我们知道 `componentWillUnmount` 会在 `mutation 阶段` 执行。此时 current Fiber 树还指向前一次更新的 Fiber 树，在生命周期钩子内获取的 DOM 还是更新前的。

 `componentDidMount` 和 `componentDidUpdate` 会在 `layout 阶段` 执行。此时current Fiber 树已经指向更新后的 Fiber 树，在生命周期钩子内获取的 DOM 就是更新后的。

#### 总结

从这节我们学到，`layout 阶段` 会`遍历 effectList`，依次执行 `commitLayoutEffects` 。该方法的主要工作为“根据 effectTag 调用不同的处理函数处理 Fiber 并更新 ref。
