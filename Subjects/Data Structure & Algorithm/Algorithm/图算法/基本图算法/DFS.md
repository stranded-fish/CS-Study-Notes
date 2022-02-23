# 深度优先搜索

深度优先搜索算法 (Depth-First-Search，DFS) 是一种用于遍历或搜索树或图数据结构的算法。该算法从根节点开始（在图的情况下选择某个任意节点作为根节点）并在回溯之前沿着每个分支尽可能深的探索。

**深度优先搜索主要适用于以下场景：**

* 检测有环图
* 路径查找
* 拓扑排序
* 网格问题

目录：

- [深度优先搜索](#深度优先搜索)
  - [算法模板](#算法模板)
    - [基本框架](#基本框架)
  - [常见题型](#常见题型)
    - [网格问题](#网格问题)
    - [记忆化搜索](#记忆化搜索)
  - [参考链接](#参考链接)

## 算法模板

### 基本框架

**eg 1. 递归写法**

```C++
res = []
void dfs(node, visited) {

    // 1. 判断节点合法性
    if (node not meets requirements) return;

    // 2. 标记当前节点已被访问
    visited.add(node);
    
    // 3. 处理当前节点信息
    ...

    // 4. 遍历选择列表
    for (choice in selectList) {

        // 5. 递归，进入下一层决策树
        dfs(path, selectList);
    }
}
```

**eg 2. 非递归写法**

```C++
void dfs(node, visited) {
    stack = [];
    stack.push(node);
    while (!stack.empty()) {
        
        // 1. 节点出栈
        node = stack.pop();

        // 2. 判断节点合法性
        if (node not meets requirements) continue;

        // 3. 标记当前节点已被访问
        visited.add(node);

        // 4. 处理当前节点信息
        ...

        // 5. 遍历选择列表
        for (next_node in node.selectList) {

            // 6. 节点入栈
            stack.push(next_node);
        }
    }
}
```

* `node`：当前节点信息。
* `visited`：标记数组，用于记录已访问节点。

**深度优先遍历与回溯算法的关联**

> 回溯算法可视为一种更通用的算法，而深度优先遍历则可视为回溯算法的一种特别形式。—— [What's the difference between backtracking and depth first search?](https://stackoverflow.com/questions/1294720/whats-the-difference-between-backtracking-and-depth-first-search)

故而深度优先遍历框架也可以视为回溯算法的一种特别形式。

**DFS 入参细化：**

回溯算法入参的一般形式为：`backtrack(path, selectList, constraint, target);`

而一般的深度优先搜索不需要显性维护完整 `path` 记录，仅需要当前节点信息以及 `visited` 数组防止重复访问。

```C++
/* node -> path.back() 访问路径 path 的当前节点信息
visited -> constraint 约束条件 */
void dfs(node, visited);
```

特定的，二叉树的遍历，同样可看作由一般回溯算法变形而来：

```C++
void dfs(node) {
    if (node == nullptr) return;

    // 处理当前节点信息
    ...

    dfs(node.left);
    dfs(node.right);
}
```

* `path`：二叉树遍历通常不需要记录遍历过程中的完整 `path`，故只需要保留当前 `node` 节点信息，用于递归遍历即可；
* `selectList`：二叉树的访问列表总是当前节点的左右孩子节点，故不需要显性的 `selectList` 入参；
* `constraint`：二叉树不用担心重复访问，仅需要对节点合法性做一定判断，故也不需要 `constraint` 相关的入参；
* `target`：视题目要求添加相应的 `target` 入参。

**剪枝时机：**

**DFS 算法更多地强调遍历**，也就是进一步递归搜索的步骤，故通常情况下，在遍历选择列表时，不判断节点合法性，而是在递归一开始进行判断，类似于 **先污染，后治理**。这种处理思路，可以在最大程度上强调递归步骤，同时提升程序简洁性。

但值得注意的是，先污染，后治理的处理方法，在选择列表较多的情景下，可能会产生较大的空间消耗，特别是针对非递归形式，会增加额外的无效出入队列操作，故最终还是建议视情况灵活选择剪枝时机。

**选择框架的思考：**

总体而言，大多数情况下，同一个问题运用这两种算法思想均能够解决，具体使用哪一种应该由问题所强调的方向决定：

* **强调状态回溯，需要完整的 `path` 记录，则考虑回溯的一般形式；**
* **强调遍历过程，不需要完整的 `path` 记录，则考虑形式更简单的 DFS。**

## 常见题型

### 网格问题

网格问题是指在 $m$ $\times$ $n$ 个小方格组成的一个网格图中进行某种搜索的问题，其中每个小方格与其上下左右（和对角线）四（八）个方格被认为是相邻的。

常见的网格问题有岛屿问题。在岛屿问题中，每个格子中的数字可能是 `0` 或者 `1`。其中数字为 `0` 的格子看成海洋格子，数字为 `1` 的格子看成陆地格子，这样相邻的陆地格子就连接成一个岛屿。然后在该设定下，求解关于岛屿的各种问题，如：岛屿的数量、面积、周长等。

**网格相关问题的遍历，本质上可视为图的遍历**，同样可利用 DFS 遍历二叉树或图时的思想，来对该网格进行遍历。

**DFS 二叉树遍历**

```C++
void dfs(node) {
    if (node == nullptr) return;

    // 处理当前节点信息
    ...

    dfs(node.left);
    dfs(node.right);
}
```

**DFS 网格遍历**

网格遍历，可在基础的二叉树遍历上进行扩展。其中：

* `base case` 扩展为判断当前节点坐标是否合法;
* 相邻节点扩展为其上下左右邻接点。

```C++
void dfs(int row, int col, vector<vector<int>> grid) {
    // 判断 base case
    if (!isValid(row, col, grid)) return;

    // 注意：避免重复遍历
    if (grid[r][c] != 1) return; // 当格子不符合要求时返回（如：发生重复）
    grid[r][c] = 2;              // 将格子标记为 - 已遍历

    // 对当前节点执行操作 ...

    // 访问上、下、左、右四个相邻结点
    dfs(row - 1, col, grid);
    dfs(row + 1, col, grid);
    dfs(row, col - 1, grid);
    dfs(row, col + 1, grid);
}

// 判断坐标 (r, c) 是否在网格中
boolean isValid(int row, int col, vector<vector<int>> grid) {
    return row >= 0 && row < grid.length
            && col >= 0 && col < grid[0].length;
}
```

**注意：避免出现重复遍历**

由于网格结构本质上是一个图，我们可以把每个格子看成图中的结点，而每个结点有向上下左右的四条双向边。故在图中遍历时，自然可能遇到重复遍历结点。

可通过以下方式来避免重复遍历：

* 设置单独的 `visited` 集合

```C++
// 初始化 m*n 大小的 visited 数组为 false，表示当前地图的所有节点均未被访问
vector<vector<bool>> visited(grid.size(), vector<bool>(grid[0].size(), false));

// 标记坐标 [x,y] 的方格已被访问
visited[x][y] = true;
```

* 修改原地图作为标记

以岛屿问题为例，我们需要在所有值为 `1` 的陆地格子上做遍历。每走过一个陆地格子，就把格子的值改为 `2`，这样当我们遇到 `2` 的时候，就知道这是遍历过的格子了。即，格子的可能取值变为：

* `0` —— 海洋格子
* `1` —— 陆地格子（未遍历过）
* `2` —— 陆地格子（已遍历过）

> **Tips:**
>
> 在对网格结构进行遍历时，由于每个节点的选择列表均相同（均为其邻接方格），故可以设置一个 dir 数组方便搜索。
>
> ```C++
> // 上下左右
> const int dir[4][2] = {{1, 0}, {0, 1}, {-1, 0}, {0, -1}};
> 
> // 上下左右与对角线方向
> const int dir[8][2] = {{1, 0}, {0, 1}, {-1, 0}, {0, -1},
>                       {-1, -1}, {-1, 1}, {1, -1}, {1, 1}};
> 
> int next_x, next_y;
> for (int i = 0; i < 4 or 8; ++i) {
>     next_x = x + dir[i][0];
>     next_y = y + dir[i][1];
>     ...
> }
> ```

**扩展：Flood fill**

Flood fill 算法是从一个区域中提取若干个连通的点与其他相邻区域区分开（或分别染成不同颜色）的经典算法。因为其思路类似洪水从一个区域扩散到所有能到达的区域而得名。

该算法最简单的实现方法是采用深度优先搜索的递归方法，同时也可以采用广度优先搜索的迭代来实现。

**eg 1. 先递归，后判断：**

```C++
void flood_fill(int x, int y, int color) {
        // 1. 坐标不合法
    if (x >= grid.size() || x < 0 || y >= grid[0].size() || y < 0 

        // 2. 表示该点已访问过（可通过单独的 visited 数组记录，也可通过修改原数组记录）
        // || visited[row][col] = true;
        
        // 3. 该点不等于连通变量，不可访问  
        || grid[x][y] != originColor) return;

    // 染色
    grid[x][y] = color;

    // 递归
    flood_fill(x - 1, y, color);
    flood_fill(x, y - 1, color);
    flood_fill(x + 1, y, color);
    flood_fill(x, y + 1, color);
}
```

**eg 2. 先判断，后递归：**

```C++
void flood_fill(int x, int y, int color) {
    grid[x][y] = color;
    if(x > 0 && grid[x-1][y] == originColor) flood_fill(x - 1, y, color);
    if(y > 0 && grid[x][y-1] == originColor) flood_fill(x, y - 1, color);
    if(x < MAX_X && grid[x+1][y] == originColor) flood_fill(x + 1, y, color);
    if(y < MAX_Y && grid[x][y+1] == originColor) flood_fill(x, y + 1, color);
}
```

该算法思想除了用来解决 [733. 图像渲染](https://leetcode-cn.com/problems/flood-fill/) 等典型问题外，还可以进一步扩展用来简化相关网格、图像遍历问题，如：[1254. 统计封闭岛屿的数目](https://leetcode-cn.com/problems/number-of-closed-islands/) [1905. 统计子岛屿](https://leetcode-cn.com/problems/count-sub-islands/) 等，这类问题，均需要对网格进行多次遍历，而为了避免出现重复遍历和修改原图像后带来的错误遍历结果，可以采用 Flood fill 思想，将每次遍历的节点所连通的所有节点全部 “染色” 或称为 “沉岛”（即使已经判断出结果），以简化后续遍历操作。

典型例题：

* [733. 图像渲染](https://leetcode-cn.com/problems/flood-fill/)
* [79. 单词搜索](https://leetcode-cn.com/problems/word-search/)
* [200. 岛屿数量](https://leetcode-cn.com/problems/number-of-islands/)
* [130. 被围绕的区域](https://leetcode-cn.com/problems/surrounded-regions/)
* [463. 岛屿的周长](https://leetcode-cn.com/problems/island-perimeter/)
* [695. 岛屿的最大面积](https://leetcode-cn.com/problems/max-area-of-island/)
* [827. 最大人工岛](https://leetcode-cn.com/problems/making-a-large-island/)
* [1254. 统计封闭岛屿的数目](https://leetcode-cn.com/problems/number-of-closed-islands/)
* [1905. 统计子岛屿](https://leetcode-cn.com/problems/count-sub-islands/)

### 记忆化搜索

朴素 DFS 基本等价于暴力搜索，并且在搜索的过程中会有大量重复计算，在数据量较大的情况下会出现超时。此时一般可设置备忘录 `memo` 数据结构，作为缓存用于记录已经计算过的情况，以避免重复计算。

**`memo` 数据结构通常可以通过 DFS 递归方法的签名来决定：`memo` 维度 = 递归方法中的可变因素数量。** 例如：[688. 骑士在棋盘上的概率](https://leetcode-cn.com/problems/knight-probability-in-chessboard/)，DFS 方法签名有 3 个可变变量（k, row, col），则 memo 声明为一个 3 维数组，[518. 零钱兑换 II](https://leetcode-cn.com/problems/coin-change-2/)
 有 2 个可变变量（pos, money），则 memo 声明为一个 2 维数组。

设置 `memo` 后，在每次递归之前先检查  `memo` 中是否已经有过记录:

* 如果有则直接返回结果，
* 如果没有则再进一步递归计算并设置 `memo` 值，以供下次重复使用。

```C++
// 缓存命中，直接返回结果
if (memo[x][y] != -1) return memo[x][y];

// 未命中，进一步递归计算结果
int res = 0;
for (...) {
    res += dfs(next_x, next_y);
}

// 设置 memo 值，并返回
memo[x][y] = res;
return memo[x][y];
```

**记忆化搜索可以看作是动态规划的一种写法。** 两者一般能够相互转换，通常直接把记忆化搜索的缓存数组 `memo` 改成 dp 数组，再把递归改成迭代，就成了动态规划。

## 参考链接

* [深度优先搜索](https://zh.wikipedia.org/wiki/%E6%B7%B1%E5%BA%A6%E4%BC%98%E5%85%88%E6%90%9C%E7%B4%A2)
* [岛屿类问题的通用解法、DFS 遍历框架](https://leetcode-cn.com/problems/number-of-islands/solution/dao-yu-lei-wen-ti-de-tong-yong-jie-fa-dfs-bian-li-/)
* [【彤哥来刷题啦】一题两解：记忆化搜索 转 动态规划！](https://leetcode-cn.com/problems/knight-probability-in-chessboard/solution/tong-ge-lai-shua-ti-la-yi-ti-liang-jie-j-y92k/)
