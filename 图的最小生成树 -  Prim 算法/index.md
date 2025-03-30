## 图的最小生成树 -  Prim 算法 
### 简介

[Robert Clay Prim](https://en.wikipedia.org/wiki/Robert_C._Prim)（1921－2021，100岁）是美国数学家、计算机科学家和工程师，因提出 Prim 算法（最小生成树算法）而闻名。他是 20 世纪计算机科学和运筹学领域的重要先驱之一。

> 美音：/ˈrɑːbərt/ /kleɪ/ /prɪm/

> “优化问题往往始于一个简单的需求：如何用最少的资源实现最大的连通性。” -- Robert Clay Prim

Prim 算法是一种用于寻找`无向连通图`的`最小生成树`（MST，Minimum Spanning Tree）的`贪心`算法。其核心思想是通过逐步扩展生成树，每次选择连接已选节点集和未选节点集的最小权重边。

> 生成树的特点：所有顶点都有`最小化边的个数`，若一个顶点消失，就不再是一个生成树。最小生成树：所有边的`总权重最小`。

- **算法范式**：贪心策略。逐步扩展生成树，每次选择连接已选节点集与未选节点集的最小权重边。
- **存储结构**：邻接表
- **时间复杂度**：
  - **邻接表 + 二叉堆**：O(E log V)，适用于稀疏图。
  - **邻接矩阵**：O(V²)，适用于稠密图。
  - **斐波那契堆**：O(E + V log V)，理论最优。
- **算法原理**
  - 初始化一个节点为起点，记录每个节点到生成树的最小边权重（key[]）。
  - 使用优先队列（最小堆）动态选择当前最小边对应的节点。
  - 将节点加入生成树后，更新其邻接节点的最小边权重。
  - 最终所有节点被访问时，parent[]数组记录的边构成最小生成树（MST）。

### 算法逻辑

1. **初始化**：
   - 选择任意一个节点作为起始点。
   - 维护一个数组 `key[]`，记录每个节点到生成树的最小边权重，初始化为无穷大（`INF`），起始点的 `key` 设为 0。
   - 维护一个数组 `parent[]`，记录生成树中每个节点的父节点。
   - 使用优先队列（最小堆）存储节点及其当前`key`值，按 `key` 排序。
   - 标记数组 `visited[]`，记录节点是否已加入生成树。

2. **迭代扩展生成树**：
   - 将起始节点加入优先队列。
   - 当队列非空时：
     - 取出 `key` 最小的节点 `u`，标记为已访问。
     - 遍历 `u` 的所有邻接节点 `v`：
       - 若 `v` 未被访问，且边 `u-v` 的权重小于 `v` 的当前 `key` 值：
         - 更新 `v` 的 `key` 值为该权重，并记录 `parent[v] = u`。
         - 将 `v` 及其新 `key` 值插入优先队列。

3. **生成结果**：
   - `parent`数组中的边构成最小生成树。
   - `key`数组的总和为生成树的总权重。

### 实现

```python
import heapq

def prim(graph, start):
    n = len(graph)
    key = [float('inf')] * n
    parent = [-1] * n
    visited = [False] * n
    heap = []
    
    key[start] = 0
    heapq.heappush(heap, (0, start))
    
    while heap:
        current_key, u = heapq.heappop(heap)
        if visited[u]:
            continue
        visited[u] = True
        for v, weight in graph[u]:
            if not visited[v] and weight < key[v]:
                key[v] = weight
                parent[v] = u
                heapq.heappush(heap, (key[v], v))
    
    total_weight = sum(key)
    return total_weight, parent

# 示例图的邻接表表示（节点索引：0 = A, 1 = B, 2 = C, 3 = D）
graph = [
    [(1, 1), (2, 3)],        # A 的邻接边
    [(0, 1), (2, 2), (3,4)], # B 的邻接边
    [(0, 3), (1, 2), (3,5)], # C 的邻接边
    [(1,4), (2,5)]           # D 的邻接边
]

total, parent = prim(graph, 0)
print("Total weight of MST:", total)  # 输出：7
print("Parent array:", parent)        # 例如：[-1, 0, 1, 1]
```
