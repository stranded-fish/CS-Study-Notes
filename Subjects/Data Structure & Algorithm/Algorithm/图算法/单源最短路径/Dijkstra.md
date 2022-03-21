# Dijkstra

**Dijkstra 主要用于解决 有向带权图 中 单源最短路径 问题，该算法要求所有边的权重非负。**

目录：

- [Dijkstra](#dijkstra)
  - [算法思想](#算法思想)
  - [时间复杂度分析](#时间复杂度分析)
  - [代码模板](#代码模板)
  - [参考资料](#参考资料)

## 算法思想

Dijkstra 算法在运行过程中维护一组结点集合 $S$（从源节点 $s$ 到该集合中每个结点之间的最短路径都已经被找到）。算法重复从结点集 $V - S$ 中选择最短路径估计最小的结点 $u$，将 $u$ 加入到集合 $S$，然后对所有从 $u$ 发出的边进行松弛。

因为 Dijkstra 算法每次总是选择集合 $V - S$ 中最优的结点加入到集合 $S$ 中，该算法使用的是贪心策略。

## 时间复杂度分析

Dijkstra 算法的时间复杂度取决于最小优先队列的实现：

数据结构  |  时间复杂度（总计）
:---:|:---
邻接矩阵 | $O(V^2)$
二叉堆 | $O((E + V) \log V)$  
斐波那契堆 | $O(E + V \log V)$

## 代码模板

**存图方式选择：**

* **邻接矩阵：** 使用二维矩阵来进行存图。
  * 适用于边数较多的稠密图使用，当边数量接近点的数量的平方，**即 $m \approx n^2$ 时，可定义为 稠密图**。
* **邻接表：** 与数组存储单链表的实现一致，又叫链式前向星存图。
  * 适用于边数较少的稀疏图使用，当边数量接近点的数量，**即 $m \approx n$ 时，可定义为 稀疏图**。

**eg 1. 朴素 Dijkstra**

```C++
using pii = pair<int, int>; // 用于记录 邻接点 及其 距离
const int INF = 0x3f3f3f3f; // 用于表示两个结点之间不连通
vector<int> Dijkstra(vector<vector<int>>& times, int n, int k) {
    
    // step 1. 构建邻接表
    vector<vector<pii>> adj(n);
    
    // 预分配空间，避免稠密图多次插入造成空间重新分配的消耗
    for (auto &e : adj) e.reserve(n);
    for (const auto &t : times) {
        adj[t[0]].emplace_back(t[1], t[2]);
    }
    
    // step 2. 定义 dist 数组，用于记录源点到其他点的最短距离
    vector<int> dist(n, INF);
    dist[k] = 0; // 起点初始化为 0

    // step 3. 定义 visited，用于标记该节点是否已经确定了最短距离
    vector<bool> visited(n, false);

    // step 4. 遍历所有结点，需要找到源点到所有结点的最短路（遍历 n 次）
    for (int i = 0; i < n; ++i) {
        
        // step 4.1 遍历当前 dist 数组，寻找到源点距离最短的点
        int minCost = INF, cur = -1;
        for (int j = 0; j < n; ++j) {
            if (!visited[j] && minCost > dist[j]) {
                minCost = dist[j];
                cur = j;
            }
        }
        if (cur == -1) return -1; // 图不连通
        visited[cur] = true; 

        // step 4.2 松弛操作
        for (const auto &[next, ntime] : adj[cur]) {
            if (!visited[next] && dist[next] > dist[cur] + ntime) {
                dist[next] = dist[next] + ntime;
            }
        }
    }

    return dist;
}
```

**注意：**

* `visited` 用于标记该节点是否已经确定了最短距离，一旦一个结点在 `step 4.1` 中被选择为了最近的结点，则此时也确定了该结点距源点的最短距离，后续可能有其他结点的邻接点指向该点，但都不可能使得距离更短了（可以用反证思想证明）。所以可以用该 `visited` 避免重复重复计算。

**复杂度分析：**

* 时间复杂度：$O(n^2)$
* 空间复杂度：$O(n^2)$

**适用情况：**

* 数据量较小
* 稠密图

**eg 2. 堆优化 Dijkstra**

```C++
using pii = pair<int, int>; // 用于记录 邻接点 及其 距离
const int INF = 0x3f3f3f3f; // 用于表示两个结点之间不连通
vector<int> Dijkstra(vector<vector<int>>& times, int n, int k) {
    
    // step 1. 构建邻接表
    vector<vector<pii>> adj(n);

    // 预分配空间，避免稠密图多次插入造成空间重新分配的消耗
    for (auto &e : adj) e.reserve(n);
    for (const auto &t : times) {
        adj[t[0]].emplace_back(t[1], t[2]);
    }
    
    
    // step 2. 定义 dist 数组，用于记录所有点到源点的最短距离
    vector<int> dist(n, INF);
    dist[k] = 0;
    
    // step 3. 定义 visited，用于标记该结点是否已经确定了最短距离
    vector<bool> visited(n, false);

    // step 4. 定义最小优先队列
    priority_queue<pii, vector<pii>, greater<pii>> q;
    q.emplace(0, k);

    // 同 BFS 操作
    while (!q.empty()) {
        auto [time, cur] = q.top(); q.pop();
        if (visited[cur] == true) continue;
        visited[cur] = true; // 表示已经找到最短路
        for (const auto &[next, ntime] : adj[cur]) {
            if (!visited[next] && dist[next] > dist[cur] + ntime) {
                dist[next] = dist[cur] + ntime;
                q.emplace(dist[next], next); // 注意：first 为代价
            }
        }
    }

    return dist;
}
```

**注意：**

* 将 `pair<int, int>` 作为优先队列存储对象时，先比较 `first` 元素，后比较 `second` 元素，故需要以 `{cost, idx}` 形式保存。
* 优先队列 `q` 中保存的 `{cost, idx}` 不代表 `idx` 结点到 `源结点` 的最短距离为 `cost`，**只代表目前搜寻到有一条从 `源节点` 到 `idx` 结点的代价为 `cost` 的路径（`q` 中可能会同时保存多条路径，但其中只有一条是最短的）**，只有每次从队列中弹出的对象，才能保证是最短的路径，此时将弹出的 `idx`，进行标价 `visited[idx] = true`，表明已确定 `idx` 与 源结点 之间的最短路径，用于跳过后续弹出的更长的路径。

由于 `dist` 始终保存着目前 `源结点` 到 `idx` 结点的最短距离，故可以用其来代替 `visited` 进行简化，即，如果 `q` 弹出的路径代价比已经计算得出的 `dist[idx]` 要大，则可以直接跳过。

**简化：**

```C++
while (!q.empty()) {
    auto [time, cur] = q.top(); q.pop();
    if (time > dist[cur]) continue; // 直接用 dist 进行判断，不用 visited 
    for (const auto &[next, ntime] : adj[cur]) {
        if (dist[next] > dist[cur] + ntime) {
            dist[next] = dist[cur] + ntime;
            q.emplace(dist[next], next);
        }
    }
}
```

**复杂度分析：**

* 时间复杂度：$O(m\log{n} + n)$
* 空间复杂度：$O(m)$

其中 $n$ 为点数，$m$ 为边数。

**适用情况：**

* 数据量较大
* 稀疏图

**扩展 1：** 只需要计算源点到终点的最短路径

因为每次从队列中弹出的结点，都已经确定了到源点的最短路径，故如果弹出节点是终点 `end`，那么此时 `dist[end]` 就是从 `start` 到 `end` 的最短距离，可直接返回。

```C++
while (!q.empty()) {
    auto [time, cur] = q.top(); q.pop();
    if (cur == end) return dist[end]; // 注意：当弹出结点为 end 时，直接返回到 end 最短距离
    if (time > dist[cur]) continue;
    for (const auto &[next, ntime] : adj[cur]) {
        if (dist[next] > dist[cur] + ntime) {
            dist[next] = dist[cur] + ntime;
            q.emplace(dist[next], next);
        }
    }
}
```

**扩展 2：** 计算最短路的同时，统计最短路径数量

```C++
// 核心代码

/* 声明 cnts 计数数组，用于统计每个点到源点的最短路径数量
源点初始为 1，其余点初始为 0 */
vector<int> cnts(n);
cnts[0] = 1;

while (!q.empty()) {
    auto [d, cur] = q.top(); q.pop();
    if (dist[cur] < d) continue;
    for (const auto &[next, nd] : adj[cur]) {
        if (dist[next] > dist[cur] + nd) {
            /* 注意 1：当寻找到一条更优路径时，修改最短距离的同时，
            更改 cnts 数组为更优路径数量 */
            cnts[next] = cnts[cur];
            dist[next] = dist[cur] + nd;
            q.emplace(dist[next], next);
        } else if (dist[next] == dist[cur] + nd) {
            /* 注意 2：当两者距离相同时，表示寻找到了两条最短距离相同的路径，
            此时累加两者路径数 */
            cnts[next] += cnts[cur];
            cnts[next] %= MOD;
        }
    }
}

// 返回起点到终点的最短路径的路径数
return cnts[n - 1];
```

典型例题：[1976. 到达目的地的方案数](https://leetcode-cn.com/problems/number-of-ways-to-arrive-at-destination/)

## 参考资料

* 《算法导论》
* [Dijkstra - wiki](https://zh.wikipedia.org/wiki/%E6%88%B4%E5%85%8B%E6%96%AF%E7%89%B9%E6%8B%89%E7%AE%97%E6%B3%95)
* [【宫水三叶】涵盖所有的「存图方式」与「最短路算法（详尽注释）」](https://leetcode-cn.com/problems/network-delay-time/solution/gong-shui-san-xie-yi-ti-wu-jie-wu-chong-oghpz/)
