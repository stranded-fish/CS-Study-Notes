# Bellman-Ford

**Bellman-Ford 主要用于解决 有向带权图 中 单源最短路径 问题，该算法边的权重可以为负值。**

Bellman-Ford 算法返回一个 `bool` 值，以表明是否存在一个从源结点可以到达的权重为负值的环路。如果存在这样的环路，算法将告诉我们不存在解决方案。如果没有这种环路存在，算法将给出最短路径及其权重。

目录：

- [Bellman-Ford](#bellman-ford)
  - [算法思想](#算法思想)
  - [时间复杂度分析](#时间复杂度分析)
  - [代码模板](#代码模板)
  - [参考资料](#参考资料)

## 算法思想

Bellman-Ford 算法通过对 **边** 进行松弛操作来渐近地降低从源节点 `s` 到每个结点 `v` 的最短路径的估计值 `v.d`，直到该估计值与实际的最短路径权重相同为止（最多需要进行 `V - 1` 次松弛操作）。

* Bellman-Ford 关注的是边
* Dijkstra 关注的是点

## 时间复杂度分析

时间复杂度为：$O(VE)$，其中 $V$ 为点集，$E$ 为边集（最多进行 $O(V)$ 次松弛操作，每次松弛操作需要 $O(E)$ 遍历所有边）。

## 代码模板

```C++
// 定于 边 结构 u->v，代价：cost
struct Edge {
    int u, v, cost;
    Edge(int _u = 0, int _v = 0, int _cost = 0) : u(_u), v(_v), cost(_cost) {}
};
vector<Edge> edges; // 记录输入边

const int INF = 0x3f3f3f3f; // 用于表示两个结点之间不连通
vector<int> dist;           // 用于记录源点到其他点的最短距离

/*输入：n 结点数，s 源点 */
bool Bellman_Ford(int n, int s) {
    dist.resize(n, INF);
    dist[s] = 0; // 起点初始化为 0

    // 最多进行 n - 1 次松弛操作
    for (int _ = 0; _ < n - 1; ++_) {
        bool flag = true;
        for (const auto &edge : edges) {

            // 判断 s...->v 能否优化为 s...->u->v
            if (dist[edge.v] > dist[edge.u] + edge.cost) {
                dist[edge.v] = dist[edge.u] + edge.cost;
                flag = false;
            }
        }

        /* 如果遍历完所有边，都没有进行松弛操作，则表示已经找到源点到
        其他所有结点的最短路径，没有负权环，可提前返回 */
        if (flag) return true;
    }

    // 检查负权环
    for (const auto &edge : edges) {

        // 判断能否继续优化，因为一旦有负权环，路径代价可无限缩小
        if (dist[edge.v] > dist[edge.u] + edge.cost) 
            return false; // 有负权环
    }

    return true; // 没有负权环
}
```

## 参考资料

* 《算法导论》
* [Bellman-Ford - wiki](https://zh.wikipedia.org/wiki/%E8%B4%9D%E5%B0%94%E6%9B%BC-%E7%A6%8F%E7%89%B9%E7%AE%97%E6%B3%95)
