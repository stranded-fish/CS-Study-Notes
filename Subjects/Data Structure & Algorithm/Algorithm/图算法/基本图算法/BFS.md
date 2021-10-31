# 广度优先搜索

广度优先搜索算法（Breadth-First Search，BFS）是一种图形搜索算法。该算法系统地展开并检查图中的所有节点，以找寻结果，且在遍历过程中不考虑结果的可能位置。如果所有节点均被访问或找到结果，则算法中止。

所谓广度优先，就是算法需要在发现所有距离源节点 $s$ 为 $k$ 的所有节点之后，才发现距离源节点为 $k + 1$ 的其他节点。这样做的结果是，BFS 算法找到的路径是从起点 $s$ 开始到目标节点 $v$ 的 最短合法路径，即包含最少边数的路径。

广度优先搜索主要适用于以下场景：

* 无权图中找最短路径
* 可以抽象为图论的求解「最短」、「最少」、「最近」 等极小值的问题

目录：

- [广度优先搜索](#广度优先搜索)
  - [算法模板](#算法模板)
    - [基本框架](#基本框架)
  - [BFS 优化](#bfs-优化)
    - [双向 BFS](#双向-bfs)
    - [多源 BFS](#多源-bfs)
  - [BFS 与 DFS](#bfs-与-dfs)
  - [参考链接](#参考链接)

## 算法模板

### 基本框架

```C++
// 计算从起点 start 到终点 target 的最近距离
int BFS(Node *start, Node *target) {
    if (start == nullptr) return -1; // 注意 1. 特例判断

    queue<Node*> q;                  // 核心数据结构
    unordered_set<Node*> visited;    // 避免重复访问
    
    q.push(start);                   // 起点入队
    visited.insert(start);           // 标记起点
    int step = 0;                    // 记录扩散的步数

    while (q not empty) {
        int size = q.size();         // 注意 2. 提前保存队列中的元素个数

        // 将当前队列中的所有节点（同一层）向四周扩散
        while (size--) {
            Node *cur = q.front(); q.pop(); // 节点出队

            if (cur is target) return step; // 判断是否找到目标

            // 注意 3. 将 cur 的相邻 非空节点 加入队列
            for (Node *x : cur->adj()) {
                if (x not in visited) {     // 防止重复访问
                    q.push(x);
                    visited.insert(x);      // 注意 4. 节点加入队列后，立即标记为「已经访问」
                }
            }
        }

        // 更新步数
        ++step;
    }
}
```

* `visited`：标记集合，用于记录已访问的节点，以避免走回头路。该数组大部分情况下是必需的，但是如果是一般的树形结构，没有子节点到父节点的指针，天然不会有回头路，也就不需要 `visited`。
* `cur->adj()`：泛指 `cur` 的邻接节点。如在二维数组，`cur` 的上下左右四个节点为其邻接节点，在一般的二叉树中，其左右孩子节点为其邻接节点。

**注意：**

* 通常在不进行额外判断的情况下，不能向队列中加入空节点，因为空节点在访问其邻接节点时，会出现 `runtime error: member access within null pointer` 访问空指针的错误，故需要特别注意以下两点：
  * 判断初始起点 `Node *start` 是否为空，如果为空则直接返回结果。
  * 将 `cur` 的相邻节点加入队列时，除判断是否重复以外，还要检查其是否为空，只有在非空的情况下才入队。
* 遍历当前队列中的所有节点时，不能直接用 `q.size()` 作循环判断条件，因为 `q.size()` 在循环中是变量，正确的做法是：遍历队列时提前保存队列中的元素的个数 `int size = q.size();`。
* 广度优先搜索作用于图论问题的时候，将节点加入队列后，需要立即将其标记为「已经访问」（即添加到 `visited` 标记集合中），否则可能导致相同节点重复入队（如：同一层的多个节点含有相同邻接点的情况）。特别注意：初始加入起点后，也需要立即标记「已经访问」。

**运用框架的思考：**

广度优先搜索用于求解「无权图」的最短路径，因此一定要认清「无权图」这个前提条件。如果是带权图，就需要使用相应的专门的算法去解决它们。事实上，这些「专门」的算法的思想也都基于「广度优先搜索」的思想，例举如下：

* 带权有向图、且所有权重都非负的单源最短路径问题：[Dijkstra 算法](https://baike.baidu.com/item/%E8%BF%AA%E5%85%8B%E6%96%AF%E7%89%B9%E6%8B%89%E7%AE%97%E6%B3%95/23665989?fr=aladdin)；
* 带权有向图的单源最短路径问题：[Bellman-Ford 算法](https://baike.baidu.com/item/%E8%B4%9D%E5%B0%94%E6%9B%BC-%E7%A6%8F%E7%89%B9%E7%AE%97%E6%B3%95)；
* 一个图的所有结点对的最短路径问题：[Floy-Warshall 算法](https://baike.baidu.com/item/floyd-warshall%E7%AE%97%E6%B3%95/9705345)。

广度优先搜索算法与回溯算法在解题过程中均涉及到对问题的抽象建模，即，将问题转化为对一个图的搜索问题，故在使用广度优先搜索算法时也可以借鉴回溯算法框架，在对问题的图结构的建模过程中思考以下问题：

* 分支产生：分支如何产生？即每个出队节点的下一层的入队节点的产生。
  * 通常为每个节点的邻接节点；
* 剪枝条件：哪些分支是重复的或无效的？总共可归纳为哪几类剪枝的情景？即阻止节点入队的条件。
  * 树型图中通常会判断节点是否为空来避免空节点入队；
  * 无向图中通常有 `visited` 数组来避免节点重复入队；
  * 二维平面中除了有 `visited` 数组以外，通常还会判断坐标合法性以避免越界坐标入队；
* 解的产生：题目所需要的解的产生位置？叶子节点？非叶子节点？从根节点到叶子节点的路径？即算法终止的条件。
  * 通常为找到题目所需目标，此时的解可能是广度优先遍历的步数，即达到目标的最短距离。

## BFS 优化

### 双向 BFS

在朴素的 BFS 实现中，算法空间的瓶颈主要取决于搜索空间中的最大宽度。为了在保证搜索到目标结果的同时，降低搜索空间的最大宽度，以进一步突破瓶颈，可以采用双向 BFS 进行优化。

双向 BFS 适用于目标节点已知的情况。同时从起点和终点两个方向开始搜索，一旦在两个扩展方向上出现同一个结点，就意味着找到了一条联通起点和终点的最短路径，搜索结束。

基本实现思路：

* 创建「两个队列」分别用于两个方向的搜索；
* 创建「两个哈希集合」用于「解决相同节点重复搜索」；
* 为了尽可能让两个搜索方向「均衡」，每次从队列中取值进行扩展时，先判断哪个队列容量较少，优先选择较小的集合进行扩展；
* 如果在搜索过程中「搜索到另一个方向搜索过的节点」，说明找到了最短路径。

**注意：** 双向 BFS 在无解的情况下不如单向 BFS，因此可以先进行一定的预处理，判断起点和终点是否连通，如果不连通，则直接返回结果，连通的话再调用双向 BFS。

代码模板 1：

```C++
// q1、q2 为两个方向的队列
// m1、m2 为两个方向的哈希表 
    
while(!q1.empty() && !q2.empty()) {
    if (q1.size() < q2.size()) {
        update(q1, m1, m2);
    } else {
        update(q2, m2, m1);
    }
}

// update 为从队列 d 中取出一个元素进行「一次完整扩展」的逻辑
void update(queue d, Map cur, Map other) {}
```

代码模板 2：

```C++
// 创建「两个队列」分别用于两个方向的搜索
queue<string> q1, q2;
q1.push(start); // q1 代表从起点 s 开始搜索（正向）
q2.push(end);   // q2 代表从结尾 t 开始搜索（反向）

// 创建「两个哈希集合」用于「解决相同节点重复搜索」
unordered_set<string> visited1, visited2;
visited1.insert(start);
visited2.insert(end);

// 统计正反扩展步数
int step = 0;

/* 只有当两个队列都不为空时，才继续搜索
当其中一个队列为空，则说明该方向无法搜索到另外一个方向搜索过的节点，
即两个方向之间没有交集，s 到 t 之间没有最短路 */
while (!q1.empty() && !q2.empty()) {

    // 优化：每次都选择较小的队列进行扩散
    if (q1.size() > q2.size()) {
        swap(q1, q2);
        swap(visited1, visited2);
    }

    // 整体逻辑同朴素 BFS
    int size = q1.size();
    while (size--) {
        auto cur = q1.front(); q1.pop();

        /* 当某一个方向搜索过程中「搜索到另一个方向搜索过的节点」，
        说明找到了最短路径，搜索结束 */
        if (visited2.count(cur)) return step;

        for (Node *x : cur->adj()) {
            if (x not in visited1) {
                q1.push(x);
                visited1.insert(x);
            }
        }
    }
    ++step;
}

return -1;
```

> **Tips:** 可利用 `swap()` 方法高效地交换容器，从而避免对 `q1`、`q2` 大小关系的额外处理，简化编码。

典型例题：

* [127. 单词接龙](https://leetcode-cn.com/problems/word-ladder/)
* [433. 最小基因变化](https://leetcode-cn.com/problems/minimum-genetic-mutation/)
* [752. 打开转盘锁](https://leetcode-cn.com/problems/open-the-lock/)

### 多源 BFS

朴素 BFS 求最短路，针对的是：从特定起点出发、求解到达特点终点的最短距离。而有一类问题求解的是一个集合的「点集」到另外一个「点集」的最短路，即「多源最短路」。针对这类问题：

* 若仍采用朴素 BFS 的方法，需要对每个起始点，做一次 BFS，如果是二维平面搜索问题，单次 BFS 的最坏情况需要扫描完整个矩阵，复杂度为 $O(n^2)$，同时，最多有 $n^2$ 个单元格需要做 BFS，因此总时间复杂度为 $O(n^4)$。
* 而采用多源 BFS 初始状态会将源点集合中的「点集」全部入队，然后同时开始 BFS 搜索，即只进行一次 BFS，故可将复杂度降至 $O(n^2)$。

多源 BFS 与 单源 BFS 的核心搜索模块均一致，唯一区别在于单源 BFS 只有一个源点，多源 BFS 可以有多个，同时初始状态需要将多个源点同时入队。同时我们也可以通过建立虚拟「超级源点」的方式，将「多源 BFS」转换回「单源 BFS」问题，原先的多个源点即为超级源点开始的第二层。

> **Tips:** 在运用多源 BFS 求解问题时，可以灵活地反转「源点/起点」以简化求解难度。

典型例题：

* [542. 01 矩阵](https://leetcode-cn.com/problems/01-matrix/)
* [934. 最短的桥](https://leetcode-cn.com/problems/shortest-bridge/)
* [1162. 地图分析](https://leetcode-cn.com/problems/as-far-from-land-as-possible/)

## BFS 与 DFS

* 求解「最短」、「最少」、「最近」 等极小值的问题，一般使用 BFS。
* 其他问题，若 DFS 和 BFS 均能求解，一般 DFS 实现将更简单。

## 参考链接

* [广度优先搜索](https://zh.wikipedia.org/wiki/%E5%B9%BF%E5%BA%A6%E4%BC%98%E5%85%88%E6%90%9C%E7%B4%A2)
* [宽度优先搜索](https://baike.baidu.com/item/%E5%AE%BD%E5%BA%A6%E4%BC%98%E5%85%88%E6%90%9C%E7%B4%A2/5224802)
* [LeetBook - 广度优先搜索](https://leetcode-cn.com/leetbook/detail/bfs/)
* [双向 BFS 基本思路（含模板）以及两种「启发式搜索」算法](https://juejin.cn/post/6977640016436527117)
* [【宫水三叶】如何使用「多源 BFS」降低时间复杂度](https://leetcode-cn.com/problems/as-far-from-land-as-possible/solution/gong-shui-san-xie-ru-he-shi-yong-duo-yua-vlea/)
