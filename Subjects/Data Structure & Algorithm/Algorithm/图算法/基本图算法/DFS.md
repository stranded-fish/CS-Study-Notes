# DFS 深度优先搜索算法

Flood fill 思想

### 地图、网格图相关

TODO 

阅读典型例题的相关题解，并进行总结和学习

重刷过程中，先从回溯角度进行思考，判断递归之后是否需要状态重置（回溯），如果不需要总结不需要的情况。



涉及到地图、网格图相关的问题，

1. 通常可以将其求解结构抽象为树型，然后可以使用回溯算法进行求解
2. 

**网格的方向数组 tip**

const int dir[4][2] = {};

**回溯算法和DFS的区别**

两者可以统一起来

回溯算法是一种更通用的算法，DFS是回溯的一种特例，

DFS缺少回溯（状态重置）的步骤

回溯算法也可以通过复制传递，省略回溯步骤，变成DFS

**针对使用回溯框架的思考：**

* **分支如何产生？也即节点的可选择列表 `selectList` 如何产生？**
  1. 针对网格图，选择列表为其 上下左右的邻接点
  2. 针对地图，选择列表为其该起点的邻接点（地图的表示）

* **题目所需要的解的产生位置？叶子节点？非叶子节点？从根节点到叶子节点的路径？**
  1. 可能不会由回溯算法直接产生所需解，回溯算法只是找到可行解的一种手段
  2. 

* **哪些分支是重复的？总共可归纳为哪几类剪枝的情景？**
  1. 针对网格图，首先是 检查边界条件进行剪枝
  2. 针对网格图，如果不能重复（不能走回头路）则设置 visited 数组进行标记


典型例题：

网格：

* [733. 图像渲染](https://leetcode-cn.com/problems/flood-fill/)
* [79. 单词搜索](https://leetcode-cn.com/problems/word-search/)
* [200. 岛屿数量](https://leetcode-cn.com/problems/number-of-islands/)
* [130. 被围绕的区域](https://leetcode-cn.com/problems/surrounded-regions/)
* [463. 岛屿的周长](https://leetcode-cn.com/problems/island-perimeter/)
* [695. 岛屿的最大面积](https://leetcode-cn.com/problems/max-area-of-island/)
* [827. 最大人工岛](https://leetcode-cn.com/problems/making-a-large-island/)

地图：

* [797. 所有可能的路径](https://leetcode-cn.com/problems/all-paths-from-source-to-target/)