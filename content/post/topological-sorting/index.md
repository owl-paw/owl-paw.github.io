---
title: 拓扑排序及其两种实现
description: 拓扑排序的概念及其两种搜索实现方法
slug: topological-sorting
date: 2024-05-10 00:00:00+0800
math: true
toc: false
categories:
    - notes
tags:
    - 图论
---

**拓扑排序 (topological sorting)** 要解决的问题是给一个**有向无环图 (directed acyclic graph, DAG)** 的所有节点排序。概括地来说，拓扑排序对所有节点进行排序的依据是节点之间的指向关系，所以只有有向图有拓扑排序。在实际问题中，图中有向边的方向可以具体地理解为事物之间的依赖关系或者先后顺序。因此，当依赖关系出现循环等问题的时候，也就是有向图中出现环的时候，就无法进行拓扑排序。**一个图能进行拓扑排序的充要条件是它有向且无环。**

对于一个有向无环图 $G = \left(V, E\right)$，如果一个无重复元素的序列 $a$ 满足 $\forall u \in V, u \in a$，且 $\forall (u \rightarrow v) \in E$，$u$ 在序列中先于 $v$ 出现，则 $a$ 为 $G$ 的一个合法拓扑序。

![一个有向无环图](DAG.png)

以上图中所示的 DAG 为例，一个合法的拓扑序为 $\left \lbrace 1, 2, 4, 5, 3 \right \rbrace$。

## BFS 求拓扑序

从时间复杂度角度以及常数角度着想，我们通常通过 BFS 求拓扑排序，这一算法[最初由 A. Kahn 提出](https://web.archive.org/web/20240107085609/https://dl.acm.org/doi/pdf/10.1145/368996.369025)。首先，在一个集合中加入图中所有入度为 $0$ 的节点，这些节点没有前驱，所以一定是依赖链的最底层；接着，从集合中取出某一个元素，基于该节点进行扩展，将其所有的后继节点的入度 $-1$。重复此操作，直到某一个后继节点的入度降为 $0$。这时，这个后继节点的所有前驱就可以确保全部遍历完了，因此将该节点加入集合中，继续进行拓扑排序，直到队列为空为止。如果这样的操作完全作用于一个 DAG 上，则节点从队首或栈顶取出的顺序就是该 DAG 的一个合法的拓扑序列，其中**每一个节点都出现在其所有前驱的后面**。

如果上述操作被执行在一个有环图上，则最终得到的序列会出现以下几种情况：

- 最初加入集合时，找不到任何入度为 $0$ 的节点，说明图中一定有环；
- 所得序列所包含的节点数小于总节点数，说明在排序过程中在遍历完所有点之前集合就空了，说明这些点的入度均不为 $0$，则图中一定有环，并且没有进入集合的点就是环路上的点，因为从外界无法进入环路，自然也无法把环路上的点的入度降低为 $0$。

可以根据上述两条性质判断是否存在环并获取图中环上的节点。

由于拓扑排序的最终结果是将所有节点的入度全部降低至 $0$，而入度的总和就是边的总数，所以通过 BFS 求拓扑排序的时间复杂度为 $\mathcal{O}\left(V + E\right)$。

```cpp
vector<int> topological_sort(int n) {
    queue<int> q;
    vector<int> seq;

    for (int u = 1; u <= n; u++)
        if (in[u] == 0)
            q.push(u);

    while (!q.empty()) {
        int u = q.front();
        q.pop();

        seq.push_back(u);

        for (int v : edges[u])
            if (--in[v] == 0)
                q.push(v);
    }
    return seq;
}
```

## DFS 求拓扑序

还有一种通过 DFS 求拓扑排序的方法。这种方法不需要直接统计每个节点的度数。它的原理是不断从节点集中取出节点，并从该节点开始，对所有子节点进行 DFS，并在当前节点的所有子节点全部搜索完成之后将该节点插入序列中。由于子节点的搜索一定在当前节点被插入序列之前完成，所以最终得到的序列是一个合法的逆拓扑序，将其进行翻转即可得到一个合法的拓扑序。

由于每个节点最多被搜索一次，且每条边也最多被遍历一次，所以该算法的整体时间复杂度同样是 $\mathcal{O}\left(V + E\right)$。当然，在实际程序实现中，DFS 还需要考虑递归带来的额外时间消耗，所以使用常数较小的 BFS 还是更加常见的选择。

```cpp
void DFS(int u, vector<int>& seq) {
    vis[u] = true;
    for (int v : edges[u])
        if (!vis[v])
            DFS(v, seq);
    seq.push_back(u);
}

vector<int> topological_sort() {
    vector<int> seq;

    for (int u = 1; u <= n; u++)
        if (!vis[u])
            DFS(u, seq);

    reverse(seq.begin(), seq.end());
    return seq;
}
```
