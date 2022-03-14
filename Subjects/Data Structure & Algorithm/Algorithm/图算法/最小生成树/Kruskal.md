# Kruskal

**Kruskal 主要用于求解 无向连通带权图 中的 最小生成树 [^最小生成树]。**

目录：

- [Kruskal](#kruskal)
  - [算法思想](#算法思想)
  - [时间复杂度分析](#时间复杂度分析)
  - [代码模板](#代码模板)
  - [参考资料](#参考资料)

## 算法思想

在 Kruskal 算法中，集合 `A` 是一个 **森林**，其结点就是给定图的结点。每次加入到集合 `A` 中的安全边永远是权重最小的连接两个不同分量的边。显然，Kruskal 算法属于贪心算法。

**基本步骤：**

1. 将边集合 `A` 初始化为一个空集合，并创建 `V` 棵树，每棵树仅包含一个结点；
2. 将原图中所有的边按权值从小到大排序；
3. 从权值最小的边开始，如果这条边连接的两个结点不在同一个连通分量中，则添加这条边到生成树的边集 `A` 中，并将此边所连的两个顶点 `u`、`v` 的集合进行 `unite`；
4. 重复 3，直到边集 `A` 数量为 `n - 1` 时停止。

## 时间复杂度分析

时间复杂度为： $O(E \lg E)$，其中 $E$ 为图的边集（来源于对边权重进行排序的操作）。

Kruskal 主要针对边进行操作，需要对所有边权重进行排序，因此 **适用于稀疏图。**

## 代码模板

```C++
/* 输入：点集 n（编号从 0 开始） 与 边集合 edge[a, b, cost]
   输出：最小生成树权值 */
int minimumCost(int n, vector<vector<int>>& edges) {

    // 定义并查集，初始化 n 个结点
    UnionFind uf(n);

    // 将边按权值进行排序
    sort(edges.begin(), edges.end(), []
            (const auto &e1, const auto &e2) { return e1[2] < e2[2]; });

    int res = 0;
    for (auto &e : edges) {

        // 尝试将两个结点合并，如果两个结点不在同一个连通分量中，返回 true
        if (uf.unite(e[0], e[1])) {
            res += e[2];
            if (uf.getCnt() == 1) return res;
        }
    }
    
    return -1; // 图不连通
}
```

## 参考资料

* 《算法导论》
* [Kruskal - wiki](https://zh.wikipedia.org/wiki/%E5%85%8B%E9%B2%81%E6%96%AF%E5%85%8B%E5%B0%94%E6%BC%94%E7%AE%97%E6%B3%95)

[^最小生成树]: [最小生成树 - wiki](https://zh.wikipedia.org/wiki/%E6%9C%80%E5%B0%8F%E7%94%9F%E6%88%90%E6%A0%91)
