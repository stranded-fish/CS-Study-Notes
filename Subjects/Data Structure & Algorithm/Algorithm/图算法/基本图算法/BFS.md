# 广度优先搜索

广度优先搜索算法（Breadth-First Search，BFS）是一种图形搜索算法。该算法属于一种盲目搜索法，目的是系统地展开并检查图中的所有节点，以找寻结果，且在遍历过程中不考虑结果的可能位置。如果所有节点均被访问或找到结果，则算法中止。

所谓广度优先，就是每次都尝试访问同一层的节点，只有当同一层都访问完了，再访问下一层。这样做的结果是，BFS 算法找到的路径是从起点开始的 **最短** 合法路径。换言之，这条路所包含的边数最小。

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
    unordered_set<Node*> visited; // 避免走回头路
    
    q.push(start); // 将起点加入队列
    visited.insert(start);
    int step = 0; // 记录扩散的步数

    while (q not empty) {
        int size = q.size();
        /* 将当前队列中的所有节点向四周扩散 */
        while (size--) {
            Node *cur = q.front(); q.pop();
            /* 划重点：这里判断是否到达终点 */
            if (cur is target)
                return step;
            /* 将 cur 的相邻节点加入队列 */
            for (Node *x : cur.adj())
                if (x not in visited) {
                    q.push(x);
                    visited.insert(x);
                }
        }
        /* 划重点：更新步数在这里 */
        step++;
    }
}
```

* `start`：
* `target`：
* `q`：
* `cur.adj()`：
* `visited`：

TODO

## 双向 BFS

## 多源 BFS

TODO

## 参考链接

* [广度优先搜索](https://zh.wikipedia.org/wiki/%E5%B9%BF%E5%BA%A6%E4%BC%98%E5%85%88%E6%90%9C%E7%B4%A2)
* [宽度优先搜索](https://bkso.baidu.com/item/%E5%AE%BD%E5%BA%A6%E4%BC%98%E5%85%88%E6%90%9C%E7%B4%A2/5224802?fromtitle=%E5%B9%BF%E5%BA%A6%E4%BC%98%E5%85%88%E7%AD%96%E7%95%A5&fromid=9028521)
