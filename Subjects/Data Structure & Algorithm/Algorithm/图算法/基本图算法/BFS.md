# 广度优先搜索

广度优先搜索算法（Breadth-First Search，BFS）是一种图形搜索算法。该算法系统地展开并检查图中的所有节点，以找寻结果，且在遍历过程中不考虑结果的可能位置。如果所有节点均被访问或找到结果，则算法中止。

所谓广度优先，就是算法需要在发现所有距离源节点 $s$ 为 $k$ 的所有节点之后，才发现距离源节点为 $k + 1$ 的其他节点。这样做的结果是，BFS 算法找到的路径是从起点 $s$ 开始到目标节点 $v$ 的 **最短** 合法路径，即包含最少边数的路径。

广度优先搜索主要适用于以下场景：

* 图、树相关最短路问题。
* 一些可以抽象为无权图的最短路径的 求解最短、最少、最小的问题
* TODO

目录：

- [广度优先搜索](#广度优先搜索)
  - [算法模板](#算法模板)
    - [基本框架](#基本框架)
  - [双向 BFS](#双向-bfs)
  - [多源 BFS](#多源-bfs)
  - [参考链接](#参考链接)

## 算法模板

### 基本框架

```C++
// 计算从起点 start 到终点 target 的最近距离
int BFS(Node *start, Node *target) {
    queue<Node*> q;               // 核心数据结构
    unordered_set<Node*> visited; // 避免重复访问
    
    q.push(start);                // 起点入队
    visited.insert(start);        // 标记起点
    int step = 0;                 // 记录扩散的步数

    while (q not empty) {
        int size = q.size();

        // 将当前队列中的所有节点（同一层）向四周扩散
        while (size--) {
            Node *cur = q.front(); q.pop(); // 节点出队

            if (cur is target) return step; // 判断是否找到目标

            // 将 cur 的相邻节点加入队列
            for (Node *x : cur->adj()) {
                if (x not in visited) {     // 防止重复访问
                    q.push(x);
                    visited.insert(x);
                }
            }
        }

        // 更新步数
        ++step;
    }
}
```

* `visited`：标记数组，用于记录已访问的节点，以避免走回头路。该数组大部分情况下是必需的，但是如果是一般的树形结构，没有子节点到父节点的指针，天然不会有回头路，也就不需要 `visited`。
* `cur->adj()`：泛指 `cur` 的邻接节点。如在二维数组，`cur` 的上下左右四个节点为其邻接节点，在一般的二叉树中，其左右孩子节点为其邻接节点。

## 双向 BFS

## 多源 BFS

TODO

## 参考链接

* [广度优先搜索](https://zh.wikipedia.org/wiki/%E5%B9%BF%E5%BA%A6%E4%BC%98%E5%85%88%E6%90%9C%E7%B4%A2)
* [宽度优先搜索](https://bkso.baidu.com/item/%E5%AE%BD%E5%BA%A6%E4%BC%98%E5%85%88%E6%90%9C%E7%B4%A2/5224802?fromtitle=%E5%B9%BF%E5%BA%A6%E4%BC%98%E5%85%88%E7%AD%96%E7%95%A5&fromid=9028521)
