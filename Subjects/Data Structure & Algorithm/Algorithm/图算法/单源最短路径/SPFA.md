# SPFA

SPFA（Shortest Path Faster Algorithm，最短路径快速算法）通常认为是队列优化的Bellman-Ford 算法。

**SPFA 主要用于解决 有向带权图 中 单源最短路径 以及 判断负权环 问题，该算法边的权重可以为负值。**

目录：

- [SPFA](#spfa)
  - [算法思想](#算法思想)
  - [时间复杂度分析](#时间复杂度分析)
  - [代码模板](#代码模板)
  - [参考资料](#参考资料)

## 算法思想

SPFA 基本思路与 Bellman-Ford 算法相同，即每个节点都被用作用于松弛其相邻节点的备选节点。但相较于 Bellman-Ford 算法，SPFA 算法的改进之处在于它并不盲目地尝试所有节点，而是维护一个备选的节点队列，并且仅有节点被松弛后才会放入队列中。整个流程不断重复直至没有节点可以被松弛。

## 时间复杂度分析

由于 SPFA 算法本质上依然是 Bellman-Ford 算法的一个特例，故 SPFA 算法的最差复杂度与 Bellman-Ford 相同，均为：$O(VE)$，其中 $V$ 为点集，$E$ 为边集。

## 代码模板

```C++
// 邻接表：first - next，second - cost
vector<vector<pair<int, int>>> adj(n);

bool SPFA(int start, int n) {
    vector<int> dist(n, INF);
    dist[start] = 0;
    
    queue<int> q;
    q.push(start);

    // 用于标记目前已在队列中的结点，防止重复入队
    vector<bool> visited(n);
    visited[start] = true;

    // cnts[i] 为入队列次数，用来判定是否存在负权环
    vector<int> cnts(n);
    cnts[start] = 1;

    while (!q.empty()) {
        auto cur = q.front(); q.pop();
        visited[cur] = false;
        for (const auto &[next, nd] : adj[cur]) {
            if (dist[next] > dist[cur] + nd) {
                dist[next] = dist[cur] + nd;
                if (!visited[next]) {
                    q.push(next);
                    visited[next] = true;
                    if (++cnt[v] > n) // 存在负权环
                        return false;
                }
            }
        }
    }

    return true;
}
```

## 参考资料

* [最短路径快速算法 - wiki](https://zh.wikipedia.org/wiki/%E6%9C%80%E7%9F%AD%E8%B7%AF%E5%BE%84%E5%BF%AB%E9%80%9F%E7%AE%97%E6%B3%95)
