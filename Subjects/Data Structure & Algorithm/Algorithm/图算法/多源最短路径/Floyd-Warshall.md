# Floyd-Warshall

**Floyd-Warshall 主要用于解决 任意两点间的最短路径 问题，可以正确处理有向图或负权（但不可存在负权回路）的最短路径问题。**

目录：

- [Floyd-Warshall](#floyd-warshall)
  - [算法思想](#算法思想)
  - [复杂度分析](#复杂度分析)
  - [代码模板](#代码模板)
  - [参考资料](#参考资料)

## 算法思想

Floyd-Warshall 算法的原理是动态规划。

设 $D[k][i][j]$ 为从 $i$ 到 $j$ 的只以 $(1..k)$ 集合中的节点为中间节点的最短路径的长度。

* 若最短路径经过点 $k$，则 $D[k][i][j] = D[k - 1][i][k] + D[k - 1][k][j]$；
  * 即通过结点 $k$ 将路径分解成两段，来判断经过结点 $k$ 能否缩短路径。
* 若最短路径不经过点 k，则 $D[k][i][j] = D[k - 1][i][j]$。
  * 即不考虑结点 $k$ 的影响。

因此，$D[k][i][j] = min(D[k - 1][i][j], D[k - 1][i][k] + D[k - 1][k][j])$。

## 复杂度分析

* 时间复杂度：$O(N^3)$
* 空间复杂度：$O(N^2)$

## 代码模板

```C++
// dist[m][n] 保存任意两点间的最短距离，初始化为两点直连距离，如果不连通则为 INF
// path[m][n] 保存任意两点间的最短路径的中间结点，初始化为 -1 表示两点之间没有中间结点

for (int k = 0; k < n; k++) {
    for (int i = 0; i < n; i++) {
        for (int j = 0; j < n; j++) {
            if (dist[i][j] > dist[i][k] + dist[k][j]) {
                dist[i][j] = dist[i][k] + dist[k][j];
                path[i][j] = k;
            }
            // 若不考虑最短路径，只考虑最终距离，则可以简化为：
            // dist[i][j] = min(dist[i][j], dist[i][k] + dist[k][j]);
        }  
    }
}

// 输出 u v 两点间的最短路径
void printPath(int u, int v, vector<vector<int>> &path) {
    if (path[u][v] == -1) {
        // 输出中间路径 u->v...
    } else {
        int mid = path[u][v];
        printPath(u, mid, path);
        printPath(mid, v, path);
    }
}
```

## 参考资料

* 《算法导论》
* [Floyd-Warshall 算法 - wiki](https://zh.wikipedia.org/wiki/Floyd-Warshall%E7%AE%97%E6%B3%95)
