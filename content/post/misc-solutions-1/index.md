---
title: '杂题选解（一）'
description: 来自 JOI 系列比赛、LibreOJ 原创以及 Codeforces 的四道题！
slug: misc-solutions-1
date: 2024-05-10 00:00:03+0800
math: true
categories:
    - editorial
    - notes
tags:
    - 图论
    - 数据结构
---

本篇杂题选解包含四道题的文字讲解，题目难度从绿到紫。主要涉及了分治思想结合数据结构的应用，也涉及到一些图论方面的问题。

## Foehn Phenomena

- 出处：JOI 2017 Final
- 官方题面：[日语](https://www2.ioi-jp.org/joi/2016/2017-ho/2017-ho.pdf) & [英语](https://www2.ioi-jp.org/joi/2016/2017-ho/2017-ho-en.pdf)
- 评测地址：[AtCoder](https://atcoder.jp/contests/joi2017ho/tasks/joi2017ho_a) & [LibreOJ #2332](https://loj.ac/p/2332)

由于题目的设置与序列中相邻两个元素之间的差值有关，并且之后给出的每一次操作都需要进行一次区间加，就很容易想到通过差分维护。考虑到每次修改都是对区间的海拔进行整体升高或降低，故每次修改对差分数组的影响只会体现在左右两个端点上。也就是说，每次修改某个区间的海拔时，由于区间内部的相对地形不变，所以区间内部对海风温度产生影响的大小也是不变的，因而我们只需要根据两个端点处的差值对答案进行修改即可。

具体地，我们先确定初始情况下各相邻两个元素之间的差值，并且依此维护海风到达 $N$ 时最终的温度，然后再根据每次修改的左右端点进行操作。对于两个端点，我们先减去根据此前的差值确定的风的温度，然后将修改值更新到该位置对应的差分数组上，最后根据新的差值重新计算风的温度。

在代码实现上有两个小细节：

- 如果海拔从低到高，那么温度下降，此时的差值为正数，但对差分数组的增量应该是一个负数，也就是差值与 $S$ 的积的相反数；如果海拔从高到低，那么温度上升，此时的差值为负数，但对差分数组的增量应该是一个正数，也就是差值与 $T$ 的积的相反数；
- 如果修改的右端点是 $N$，那么右端点处的差分以及答案就不需要更新了，因为此时右端点的修改影响不到任何差值。

{{< collapse "点击查看全部代码" >}}

``` cpp
#include <iostream>

#define endl '\n'

using std::cin;
using std::cout;

using i64 = long long;

const int MAX_N = 2e5 + 5;

int N, Q, S, T;
i64 delta[MAX_N];

i64 alter(i64 i) {
    if (delta[i] > 0) return -delta[i] * S;
    return -delta[i] * T;
}

int main() {
    std::ios::sync_with_stdio(false);
    cin.tie(nullptr);

    int last = 0;
    i64 ans = 0;

    cin >> N >> Q >> S >> T >> last;

    for (int i = 1, curr; i <= N; i++) {
        cin >> curr;
        delta[i] = curr - last;
        ans += alter(i);
        last = curr;
    }

    while (Q--) {
        int L, R, X;
        cin >> L >> R >> X;

        ans -= alter(L);
        delta[L] += X;
        ans += alter(L);

        if (R < N) {
            ans -= alter(R + 1);
            delta[R + 1] -= X;
            ans += alter(R + 1);
        }

        cout << ans << endl;
    }

    return fflush(stdout), 0;
}
```

{{< /collapse >}}

## 暑假时在做什么？有没有空？可以来学物理吗？

- 出处：LibreOJ
- 官方题面及评测地址：[LibreOJ #6490](https://loj.ac/p/6490)
- 参考题解：[by ImALAS](https://www.cnblogs.com/663B/p/16653601.html)

这道题的朴素做法其实非常好想。对于每一个 $a_i$，暴力枚举区间长度，然后枚举区间端点，求区间和，取最大值即可。如果加上一个前缀和优化，那么朴素做法的复杂度将达到 $\mathcal{O} \left( n ^ 3 \right)$。

考虑如何优化。我们知道，对于一段子区间 $\left[ L, R \right]$，它的区间和可以通过前缀和得到，即 $\sum _ {i = L} ^ {R} = \text{pre} _ R - \text{pre} _ {L - 1}$。在本题中，当 $L, R$ 并非确定值，而是各自在一个确定的取值范围内浮动的时候，想要找到最大的子区间和，就需要找到最小的 $\text{pre} _ {L - 1}$ 以及最大的 $\text{pre} _ R$。因此我们可以考虑使用数据结构维护区间最大值和最小值，而所需要维护的序列便是前缀和数组。本题由于时限只有 400 毫秒、没有对数组元素的修改并且包含很多的查询操作，所以使用 ST 表来维护。

不过，这只是把对单个 $a_i$ 求答案的问题解决了，复杂度还不足以拿满分。为此，我们需要想想如何一次性求出对于所有元素的答案。考虑分治。对于每一层的问题规模，我们先递归求出只考虑左半边时，左半边元素的最优答案，以及只考虑右半边时右半边元素的最优答案，然后考虑左右两边合并时对规模内的所有元素的答案造成的影响。

具体来说，在合并考虑左右两边的影响时，以左半边为例，我们先顺序遍历一遍左半边 $\left[ l, \text{mid} \right]$ 的所有下标 $i$，用一个 $p$ 变量存储最优答案，每次查询用 ST 表查询

$$
\left( \max\left\lbrace i + l - 1, \text{mid} + 1\right\rbrace , \min\left\lbrace i + r - 1, R\right\rbrace \right)
$$

区间内的最大前缀和 $\text{pre} _ {\max}$（其中 $l, r$ 为区间长度的取值范围，$R$ 为问题规模的上界，$\text{mid}$ 为问题规模的中点），然后更新 $p = \max\left\lbrace p, \text{pre}_{\max}\right\rbrace$，$f_i = \max\left\lbrace f_i, p\right\rbrace$。以上是合并时考虑左半边的过程。对于右半边，需要倒序遍历并使用同样的方法进行维护，取得最优答案。

还是以左半边为例，这样做的正确性在于：

- $p$ 的范围一定会覆盖当前遍历到的位置，因为查询区间的起点在右半边，所以无论如何查询，最终结果一定覆盖了当前位置直到左半边的右端点在内的所有下标。
- 如果左边的某个元素在规模扩展到右边界后答案并不优于此前在分治左半边过程中确定的答案，那么 $f_i$ 不会被更新；如果扩展之后答案更优，那么 $f_i$ 会被更新为更优的答案。

代码实现上，需要关注的地方有：

- 记得开 `long long`；
- 根据以上方法使用 ST 表进行查询时，以左半边为例，当左边界取 $\text{mid} + 1$ 且右边界取 $i + r - 1$ 的时候，可能出现左边界大于右边界的情况。这种情况表示区间长度取不到右半边里，故返回极值来避免统计这种情况。

{{< collapse "点击查看全部代码" >}}

``` cpp
#include <climits>
#include <cstring>
#include <iostream>

#define endl '\n'

using namespace std;
using i64 = long long;

const int N = 1e5 + 5;
const int M = 20;

struct SparseTable {
    i64 mx[N][M];
    i64 mn[N][M];

    int log2(int x) {
        int exp = 0;
        while (x >>= 1)
            exp += 1;
        return exp;
    }

    void init(int x, i64* v) {
        for (int i = 0; i <= x; i++)
            mx[i][0] = mn[i][0] = v[i];
        for (int j = 1; 1 << j <= x; j++)
            for (int i = 0; i + (1 << j) - 1 <= x; i++) {
                mx[i][j] = max(mx[i][j - 1], mx[i + (1 << (j - 1))][j - 1]);
                mn[i][j] = min(mn[i][j - 1], mn[i + (1 << (j - 1))][j - 1]);
            }
    }

    i64 query_max(int l, int r) {
        if (l > r) return INT_MIN;
        int t = log2(r - l + 1);
        return max(mx[l][t], mx[r - (1 << t) + 1][t]);
    }

    i64 query_min(int l, int r) {
        if (l > r) return INT_MAX;
        int t = log2(r - l + 1);
        return min(mn[l][t], mn[r - (1 << t) + 1][t]);
    }
};

int n, L, R;
int a[N];
i64 ans[N], p[N];
SparseTable ST;

void solve(int l, int r) {
    if (r - l + 1 < L) return;

    if (l == r) {
        if (L == 1)
            ans[l] = max(ans[l], (i64)a[l]);
        return;
    }

    int mid = (l + r) >> 1;

    solve(l, mid), solve(mid + 1, r);

    i64 pre = LLONG_MIN, suf = LLONG_MIN;

    for (int i = l; i <= mid; i++) {
        int x = max(i + L - 1, mid + 1);
        int y = min(i + R - 1, r);
        pre = max(pre, ST.query_max(x, y) - p[i - 1]);
        ans[i] = max(ans[i], pre);
    }

    for (int i = r; i > mid; i--) {
        int x = max(i - R + 1, l);
        int y = min(i - L + 1, mid);
        suf = max(suf, p[i] - ST.query_min(x - 1, y - 1));
        ans[i] = max(ans[i], suf);
    }
}

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);

    cin >> n >> L >> R;
    for (int i = 1; i <= n; i++)
        cin >> a[i], p[i] = p[i - 1] + a[i], ans[i] = LLONG_MIN;

    ST.init(n, p);

    solve(1, n);

    for (int i = 1; i <= n; i++)
        cout << ans[i] << ' ';

    fflush(stdout);
    return 0;
}
```

{{< /collapse >}}

## Water Bottle

- 出处：JOISC 2014 Day 2
- 官方题面：[日语](https://www.ioi-jp.org/camp/2014/2014-sp-tasks/2014-sp-d2.pdf)
- 评测地址：[AtCoder](https://atcoder.jp/contests/joisc2014/tasks/joisc2014_e) & [LibreOJ #2876](https://loj.ac/p/2876)
- 参考题解：[公式解说](https://www2.ioi-jp.org/camp/2014/2014-sp-tasks/2014-sp-d2-bottle-review.pdf)

看到题之后，很容易发现一个获取答案的方法。我们假设求的是从建筑物 $A$ 到建筑物 $B$ 的路上水瓶至少需要的容量。那么首先，最基础地，水瓶最大需要的容量便是从 $A$ 直接走到 $B$ 途中完全不打水的情况下两点之间的最短路长度 $-1$。那么如果存在一种方式，使得先从 $A$ 到某其他建筑物 $C$ 打水，再从 $C$ 到 $B$ 的过程从所需水瓶的最大容量比直接走所需要的最大容量要小，那么我们就认为这种方式更优。同理，在优化之后的路线上，我们还可以继续优化，直到找到最优解。

前文中我们已经提到：从一个建筑物走到另一个建筑物上，如果不经过其他建筑物打水，那么途中需要消耗的水量就是两点之间的最短路长度 $-1$。也就是说，最优解的路径一定是一个或多个最短路的拼接，并且在这些最短路能够连通起点和终点的前提下，还要尽可能让这些最短路中的最大值尽可能小。这样说容易让人想到求最小生成树。实际上与最小生成树还是有区别的，因为不用连通所有的建筑物。但我们还是可以在这道题中应用 Kruskal 的思想。同时，由于用 LCA 在树上求最大边权的复杂度 $\mathcal{O} \left(\log n\right)$ 远低于在一般图上用 Dijkstra 求最短路和最大边权的复杂度 $\mathcal{O} \left(n \log n\right)$，故如果能把这张图化成一棵树，那么想必更有助于我们解决这道题。

如何快速找到所有建筑物两两之间的最短距离？在本题中，整体复杂度较小且比较简单的方法之一是多源 BFS。具体来说，我们维护整个方格地图中每一个格子距离最近的建筑物的编号以及二者之间的距离。最初将所有建筑物一次性全部压入队列中，然后以这若干个建筑物为源点一层层地向外扩散。在从某个源点 $u$ 开始的搜索中，如果当前搜索到的格子还没有标记，说明距离该格子最近的建筑物就是 $u$；如果已经存在了某个其他源点 $v$ 的标记，说明已经到达了当前源点管辖范围的边界。由于 BFS 的性质，在当前状态、当前格子下，当前格子距离自己的源点 $u$ 的距离最短，距离当前格子的标记所属的源点 $v$ 的距离也最短，所以这两个最短距离的和便是从 $u$ 到 $v$ 的最短距离。

我们得到了所有建筑物两两之间的最短距离后，接下来就是借助 Kruskal 算法思想的时候了。我们根据建筑物之间最短距离的长度，从小到大枚举所有的最短距离，并以最短距离长度为权值，在该最短距离沟通的两个建筑物之间连一条边。同一个连通块中的两个建筑物不重复连边。这样，当遍历完所有边之后，类似 Kruskal 构造最小生成树的原理，我们就建立了一片森林。这片森林中的每一棵树代表的连通块包含所有可连通的建筑物，并且树上的所有边中长度的最大值最小。树上求最短路问题复杂度就小很多了，对于每次询问，直接在这棵树上跑 LCA 求路径上的最大边权即可。

代码实现方面，值得一提的是我们对多源 BFS 的结果转移到 LCA 过程中的处理。我们可以简单地开 $N^2 \leqslant 4\times 10^6$ 个 `vector`（在空间足够的情况下）当作桶，当出现搜索到其他源点管辖的格子时，我们将自己的源点和对方的源点都存入对应的 `vector` 中，也就是下标为当前格子到自己源点和到对方源点的两个最短距离之和的 `vector` 处，方便按照边权从小到大枚举。

{{< collapse "点击查看全部代码" >}}

``` cpp
#include <queue>
#include <vector>
#include <numeric>
#include <iostream>
#include <algorithm>

#define endl '\n'

using std::cin;
using std::cout;

using PII = std::pair<int, int>;

const int MAX_N = 2e3 + 5;
const int MAX_P = 2e5 + 5;
const int dx[] = {0, 1, 0, -1};
const int dy[] = {1, 0, -1, 0};

int N, M, P, Q;
char map[MAX_N][MAX_N];

std::queue<PII> q;
int gov[MAX_N][MAX_N];
int dist[MAX_N][MAX_N];

std::vector<PII> graph[MAX_P];
std::vector<PII> graph2[MAX_N * MAX_N];

struct UnionFind {
    int fa[MAX_P];

    void init(int x) { std::iota(fa, fa + x + 1, 0); }

    int find(int x) {
        if (fa[x] == x) return x;
        return fa[x] = find(fa[x]);
    }

    bool merge(int x, int y) {
        x = find(x), y = find(y);
        if (x == y) return false;
        return fa[x] = y, true;
    }
} sets;

namespace LCA {
    int fa[MAX_P][20];
    int mx[MAX_P][20];
    int depth[MAX_P];

    void prep(int st) {
        std::queue<int> q;
        q.push(st);
        depth[st] = 1;

        while (!q.empty()) {
            int u = q.front();
            q.pop();

            for (auto [v, w] : graph[u]) {
                if (depth[v]) continue;

                depth[v] = depth[u] + 1;
                fa[v][0] = u, mx[v][0] = w;

                for (int i = 1; i < 20; i++) {
                    fa[v][i] = fa[fa[v][i - 1]][i - 1];
                    mx[v][i] = std::max(mx[v][i - 1], mx[fa[v][i - 1]][i - 1]);
                }

                q.push(v);
            }
        }
    }

    int query(int u, int v) {
        if (depth[u] > depth[v]) std::swap(u, v);

        int mxw = 0;

        for (int i = 19; ~i; i--)
            if (depth[fa[v][i]] >= depth[u]) {
                mxw = std::max(mxw, mx[v][i]);
                v = fa[v][i];
            }

        if (u == v) return mxw;

        for (int i = 19; ~i; i--)
            if (fa[u][i] != fa[v][i]) {
                mxw = std::max({mxw, mx[u][i], mx[v][i]});
                u = fa[u][i], v = fa[v][i];
            }

        mxw = std::max({mxw, mx[u][0], mx[v][0]});
        return mxw;
    }
}  // namespace LCA

int main() {
    std::ios::sync_with_stdio(false);
    cin.tie(nullptr);

    cin >> N >> M >> P >> Q;

    for (int i = 1; i <= N; i++)
        for (int j = 1; j <= M; j++)
            cin >> map[i][j];

    for (int i = 1, x, y; i <= P; i++) {
        cin >> x >> y;
        gov[x][y] = i;
        q.emplace(x, y);
    }

    while (!q.empty()) {
        auto [x, y] = q.front();
        q.pop();

        for (int i = 0; i < 4; i++) {
            int tx = x + dx[i];
            int ty = y + dy[i];

            if (tx < 1 || ty < 1 || tx > N || ty > M) continue;
            if (map[tx][ty] == '#') continue;

            if (!gov[tx][ty]) {
                gov[tx][ty] = gov[x][y];
                dist[tx][ty] = dist[x][y] + 1;
                q.emplace(tx, ty);
            } else if (gov[tx][ty] != gov[x][y])
                graph2[dist[tx][ty] + dist[x][y]].emplace_back(gov[x][y], gov[tx][ty]);
        }
    }

    sets.init(P);

    for (int w = 0; w < N * N; w++)
        for (auto [u, v] : graph2[w])
            if (sets.merge(u, v)) {
                graph[u].emplace_back(v, w);
                graph[v].emplace_back(u, w);
            }

    for (int i = 1; i <= P; i++)
        if (!LCA::depth[i])
            LCA::prep(i);

    while (Q--) {
        int S, T;
        cin >> S >> T;

        if (sets.find(S) != sets.find(T)) cout << -1 << endl;
        else cout << LCA::query(S, T) << endl;
    }

    return fflush(stdout), 0;
}
```

{{< /collapse >}}

## Yet Another Array Counting Problem

- 出处：Codeforces Round 833
- 官方题面及评测地址：[Codeforces](https://codeforces.com/problemset/problem/1748/E)
- 中文翻译：[洛谷 RemoteJudge](https://www.luogu.com.cn/problem/CF1748E)
- 参考题解：[Official Editorial](https://codeforces.com/blog/entry/108319)

我们令 $\text{lmax}(i, j)$ 表示原序列 $a$ 的区间 $\left[i, j\right]$ 上 leftmost maximum 的位置。则可以得到一个结论，即对于任意子区间 $\left[l, r\right]$，令 $k = \text{lmax}(l, r)$，则 $a_{\text{lmax}(l, k - 1)} < a_k \leqslant a_{\text{lmax}(k + 1, r)}$（左半边不取等号是由 leftmost maximum 的性质决定的，请注意这一点在下面的分析以及代码实现中的体现）。

根据上面的分析，我们发现 $\text{lmax}$ 的取值范围是从更小的问题规模转移过来的，并且由于每个区间内的 $\text{lmax}$ 的值是可以提前预处理出来的，因此我们可以考虑以当前问题规模 $\left[l, r\right]$ 的 leftmost maximum 值 $k = \text{lmax}(l, r)$ 为中点进行分治来解决问题。

我们令 $k = \text{lmax}(l, r)$，则令 $f(k, j)$ 为当满足 $b_{\text{lmax}(l, r)} = j$ 时组成 $b_l \ldots b_r$ 的方案数。接下来思考如何转移。题目要求我们求所有子区间的 $\text{lmax}$ 都与 $a$ 序列相同的 $b$ 序列的数量，其实说白了就是对于每一个确定的 $a$ 序列上的 $\text{lmax}$，将其同时视为 $b$ 序列同一区间上的 $\text{lmax}$ 后，以 $\text{lmax}$ 为分界点的左右两边的总方案数之积（乘法原理）。由此可以分类讨论出以下几种情况：

- $b_{\text{lmax}(l, r)} = 1$ 且左半边问题规模不为 $\varnothing$ 时，根据定义，方案数为 $0$；
- $\text{lmax}(l, r) = l$ 时，当前问题规模的左半边为 $\varnothing$，此时方案数只考虑右半边；
- $\text{lmax}(l, r) = r$ 时，当前问题规模的右半边为 $\varnothing$，此时方案数只考虑左半边；
- $\text{lmax}(l, r)$ 的左右半边问题规模都不为 $\varnothing$ 时，此时方案数为左边总方案数和右边总方案数的积；
- $\text{lmax}(l, r)$ 的左右半边问题规模都为 $\varnothing$ 时，此时方案数为 $1$。

为了优化时间复杂度，我们需要随着递归过程维护总方案数 $g(k, j) = \sum_{i = 1}^{j}f(k, i)$。

综上，若令 $k = \text{lmax}(l, r)$，$lc = \text{lmax}(l, k - 1)$，$rc = \text{lmax}(k + 1, r)$，则有：

$$
f(k, j) =
\begin{cases}
0, \quad &\mathrm{if} ~ {b_{\text{lmax}(l, r)} = 1} \land {k \neq l}\newline
1, \quad &\mathrm{if} ~ {k = l = r} \newline
g(rc, j), \quad &\mathrm{if} ~ {k = l} \newline
g(lc, j - 1), \quad &\mathrm{if} ~ {k = r} \newline
g(lc, j - 1) \cdot g(rc, j), &\mathrm{otherwise.}
\end{cases}
$$

代码实现上，$\text{lmax}$ 的函数值可以通过线段树、ST 表等数据结构实现查询；另一个需要小心的地方是不能一次性把 $f$ 和 $g$ 数组开成静态的，因为所需要的空间太大了，又会造成很多空间浪费（$n, m \leqslant 2 \times 10^5$，但 $nm \leqslant 10^6$），因此需要采用开 `vector` 代替的方法进行替代。

{{< collapse "点击查看全部代码" >}}

``` cpp
#include <vector>
#include <cstring>
#include <iostream>

#define endl '\n'

using namespace std;
using i64 = long long;

const int N = 2e5 + 5;
const int MOD = 1e9 + 7;

int n, m, a[N];
vector<i64> f[N], g[N];

struct SparseTable {
    int tb[N][20];

    int log2(int x) {
        int exp = 0;
        while (x >>= 1)
            exp += 1;
        return exp;
    }

    void prep() {
        int lmt = log2(n);
        for (int i = 1; i <= n; i++) {
            tb[i][0] = i;
            f[i].assign(m + 1, 0);
            g[i].assign(m + 1, 0);
        }
        for (int j = 1; j <= lmt; j++)
            for (int i = 1, x, y; i + (1 << j) - 1 <= n; i++)
                if (a[x = tb[i][j - 1]] >= a[y = tb[i + (1 << (j - 1))][j - 1]]) tb[i][j] = x;
                else tb[i][j] = y;
    }

    int max(int l, int r) {
        int t = log2(r - l + 1), x, y;
        if (a[x = tb[l][t]] >= a[y = tb[r - (1 << t) + 1][t]]) return x;
        return y;
    }
} ST;

int divide_and_conquer(int l, int r) {
    if (l > r) return -1;

    int mid = ST.max(l, r);
    int lc = divide_and_conquer(l, mid - 1);
    int rc = divide_and_conquer(mid + 1, r);

    for (int i = 1; i <= m; i++) {
        if (lc != -1 && i == 1) f[mid][1] = 0;
        else f[mid][i] = (lc != -1 ? g[lc][i - 1] : 1LL) * (rc != -1 ? g[rc][i] : 1LL) % MOD;
        g[mid][i] = (g[mid][i - 1] + f[mid][i]) % MOD;
    }

    return mid;
}

void solve() {
    cin >> n >> m;
    for (int i = 1; i <= n; i++)
        cin >> a[i];

    ST.prep();

    int mid = divide_and_conquer(1, n);
    cout << g[mid][m] << endl;
}

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);

    int T;
    cin >> T;
    while (T--) solve();

    return fflush(stdout), 0;
}
```

{{< /collapse >}}
