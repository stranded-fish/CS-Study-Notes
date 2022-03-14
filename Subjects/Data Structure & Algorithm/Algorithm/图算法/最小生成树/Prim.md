# Prim

**Prim 主要用于求解 无向连通带权图 中的 最小生成树 [^最小生成树]。**

目录：

- [Prim](#prim)
  - [算法思想](#算法思想)
  - [时间复杂度分析](#时间复杂度分析)
  - [代码模板](#代码模板)
  - [参考资料](#参考资料)

## 算法思想

在 Prim 算法中，集合 A 是一颗 **树**。每次加入到 `A` 中的安全边永远是连接 `A` 和 `A` 之外某个结点的边中权重最小的边。Prim 算法同 Kruskal 算法一样都属于贪心算法。

**基本步骤：**

1. 以某一个点开始，寻找当前该点可以访问的所有的边；
2. 在已经寻找的边中发现最小边，这个边必须有一个点还没有访问过，将还没有访问的点加入我们的集合，记录添加的边；
3. 寻找当前集合可以访问的所有边，重复 2 的过程，直到没有新的点可以加入；
4. 此时由所有边构成的树即为最小生成树。

## 时间复杂度分析

Prim 算法的时间复杂度取决于最小优先队列的实现：

数据结构  |  时间复杂度（总计）
:---:|:---
邻接矩阵 | $O(V^2)$
二叉堆、邻接表 | $O(E \log V)$  
斐波那契堆、邻接表 | $O(E + V \log V)$

Prim 一般用邻接矩阵实现，因此 **适用于稠密图。**

## 代码模板

**邻接矩阵实现：**

```C++
/* 输入：点集 n（编号从 0 开始） 与 边集合 edge[a, b, cost]
   输出：最小生成树权值 */
const int INF = 0x3f3f3f3f;
int minimumCost(int n, vector<vector<int>>& edges) {
    int res = 0;
    
    // 由 边集 构造 邻接矩阵
    vector<vector<int>> cost(n, vector<int>(n, INF));
    for (auto &e : edges) {
        cost[e[0]][e[1]] = e[2];
        cost[e[1]][e[0]] = e[2];
    }

    // 结点当前到最小生成树距离
    vector<int> minCost(n, INF);

    // 选择结点 0 作为初始结点，初始化其他结点与其距离
    for (int i = 1; i < n; ++i) 
        minCost[i] = cost[i][0];
    
    // 初始化 visited 数组
    vector<bool> visited(n, false);
    visited[0] = true;
    
    for (int i = 1; i < n; ++i) {
        int minc = INF;
        int idx = -1;

        // 寻找集合之外权重最小的边
        for (int j = 0; j < n; ++j) {
            if (!visited[j] && minCost[j] < minc) {
                minc = minCost[j];
                idx = j;
            }
        }
        if (minc == INF) return -1; // 图不连通
        
        res += minc;
        visited[idx] = true;

        // 松弛操作
        for (int j = 0; j < n; ++j) {
            if (!visited[j] && cost[j][idx] < minCost[j]) 
                minCost[j] = cost[j][idx];
        }
    }
    return res;
}
```

## 参考资料

* 《算法导论》
* [Prim - wiki](https://zh.wikipedia.org/wiki/%E6%99%AE%E6%9E%97%E5%A7%86%E7%AE%97%E6%B3%95)
* [prim算法 上 - 一步一步写算法 - 极客学院Wiki](https://wiki.jikexueyuan.com/project/step-by-step-learning-algorithm/prim-algorithm1.html)

[^最小生成树]: [最小生成树 - wiki](https://zh.wikipedia.org/wiki/%E6%9C%80%E5%B0%8F%E7%94%9F%E6%88%90%E6%A0%91)
