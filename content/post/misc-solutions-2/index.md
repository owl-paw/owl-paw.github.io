---
title: '杂题选解（二）'
description: 来自 NOI、ICPC 以及 USACO 的三道题！
slug: misc-solutions-2
date: 2024-05-10 00:00:04+0800
math: true
categories:
    - editorial
    - notes
tags:
    - 图论
    - 数据结构
---

本篇杂题选解包含三道题的文字讲解，题目难度从绿到蓝。主要涉及了一些区间类问题与倍增、双指针等算法的结合运用。

## 区间

- 出处：NOI 2016
- 题面与评测地址：[洛谷 P1712](https://www.luogu.com.cn/problem/P1712) & [LibreOJ #2086](https://loj.ac/p/2086)
- 参考题解：[by 宝硕](https://oi.baoshuo.ren/noi2016-interval/)

先从题目最浅层的问题开始分析。如何从所有合法方案中获取到最小的花费？根据题目中的定义，我们希望找到所选中的区间长度极差最小的情况，可以通过双指针来维护。

将所有区间按照长度从小到大排序，然后以最短区间作为左右指针的起始点开始枚举，双指针维护的是当前被选中的区间的范围左右界下标。每次枚举到一个区间，将其设为选中状态，即在数轴上将该区间内的所有位置的覆盖层数叠加一层。如果当前整个数轴上存在一个整数 $x$ 被覆盖了 $m$ 次，那么就已经有了一个可行解（左右指针之间区间的个数），此时我们利用该可行解尝试更新最优解，然后删除位于左指针的区间给数轴上叠加的一层，即令其被取消选中，并向右移动左指针，直到不存在被覆盖 $m$ 次的数，由此尝试缩小极差、取得更优答案；如果整个数轴上不存在任何整数被覆盖了 $m$ 次，那么向右移动右指针，尝试取得下一个可行解。由于双指针算法在有序序列上的正确性，一定能找到所有合法方案中的最优解。

根据上面的分析，我们发现过程中需要对数轴上元素进行区间修改。同时由于数轴的范围在 $10^9$ 级别，所以我们采用离散化权值线段树的方式来维护我们需要的内容。离散化不再赘述。对于线段树，具体地，我们根据离散化后的数轴上每一个单点被覆盖的次数，维护区间内被覆盖次数的最大值。当整个数轴上被覆盖次数最大值超过或等于 $m$ 的时候，我们根据前文提到的操作，利用可行解更新最优解。

在代码实现上，需要注意的地方包括：由于线段总数最大为 $5 \times 10^5$，那么离散化数组需要存储的端点数量最大就会达到 $10^6$，所以线段树需要开 $2 \times 4 n = 8 n$ 的空间；由于只需要对整个数轴范围上被覆盖次数的最大值进行查询，所以没有必要单独写一个查询函数，直接访问线段树的根节点即可。

{{< collapse "点击查看全部代码" >}}

``` cpp
#include <limits>
#include <vector>
#include <iostream>
#include <algorithm>

#define endl '\n'

using std::cin;
using std::cout;
using std::vector;

using PII = std::pair<int, int>;

const int MAX_N = 5e5 + 5;

int N, M;
PII seg[MAX_N];
vector<int> disc;

struct SegTree {
    struct Node {
        int l, r;
        int tag, max;
    } node[MAX_N << 3];

    void pushup(int u) { node[u].max = std::max(node[u << 1].max, node[u << 1 | 1].max); }

    void pushdown(int u) {
        if (node[u].tag) {
            node[u << 1].tag += node[u].tag;
            node[u << 1].max += node[u].tag;
            node[u << 1 | 1].tag += node[u].tag;
            node[u << 1 | 1].max += node[u].tag;
            node[u].tag = 0;
        }
    }

    void build(int l, int r, int u = 1) {
        node[u] = {l, r, 0, 0};

        if (l == r) return;

        int mid = (l + r) >> 1;

        build(l, mid, u << 1);
        build(mid + 1, r, u << 1 | 1);
    }

    void update(int l, int r, int x = 1, int u = 1) {
        if (node[u].l >= l && node[u].r <= r) {
            node[u].tag += x;
            node[u].max += x;
            return;
        }

        pushdown(u);

        int mid = (node[u].l + node[u].r) >> 1;

        if (l <= mid) update(l, r, x, u << 1);
        if (r > mid) update(l, r, x, u << 1 | 1);

        pushup(u);
    }
} SGT;

int main() {
    std::ios::sync_with_stdio(false);
    cin.tie(nullptr);

    cin >> N >> M;
    for (int i = 1; i <= N; i++) {
        cin >> seg[i].first >> seg[i].second;
        disc.push_back(seg[i].first);
        disc.push_back(seg[i].second);
    }

    auto cmp = [&](PII x, PII y) { return x.second - x.first < y.second - y.first; };
    std::sort(seg + 1, seg + N + 1, cmp);
    std::sort(disc.begin(), disc.end());
    disc.erase(std::unique(disc.begin(), disc.end()), disc.end());

    for (int i = 1; i <= N; i++) {
        seg[i].first = std::lower_bound(disc.begin(), disc.end(), seg[i].first) - disc.begin() + 1;
        seg[i].second = std::lower_bound(disc.begin(), disc.end(), seg[i].second) - disc.begin() + 1;
    }

    SGT.build(1, disc.size());

    int ans = std::numeric_limits<int>::max();
    auto len = [&](int x) { return disc[seg[x].second - 1] - disc[seg[x].first - 1]; };

    for (int i = 1, l = 1; i <= N; i++) {
        SGT.update(seg[i].first, seg[i].second);
        while (SGT.node[1].max >= M) {
            ans = std::min(ans, len(i) - len(l));
            SGT.update(seg[l].first, seg[l].second, -1);
            l++;
        }
    }

    if (ans == std::numeric_limits<int>::max()) ans = -1;
    cout << ans << endl;
    return fflush(stdout), 0;
}
```

{{< /collapse >}}

## Opportunity Cost

- 出处：ICPC 2020 World Finals
- 官方题面：[英语](https://icpc.global/worldfinals/problems/2020-ICPC-World-Finals/icpc2020.pdf)
- 中文题面及评测地址：[LibreOJ #6796](https://loj.ac/p/6796) & [洛谷 P8134](https://www.luogu.com.cn/problem/P8134)

根据题目所给的式子，我们不难发现：对于一部手机 $\left( x, y, z \right)$，这三种参数最终被计入这部手机的 opportunity cost，只有两种情况。一则该项参数在所有同种参数中最大，则该参数计入值为 $0$；二则该项参数在所有同种参数中并非最大，则该参数计入值为 $x_i - x$，$y_i - y$ 或 $z_i - z$。第一种情况显然是最理想的，但它同时也对我们计算 opportunity cost 没有任何贡献，所以我们只考虑这三项参数出现第二种情况时的处理方式。我们假设该手机的三种参数都出现了第二种情况，则这部手机的 opportunity cost 为：

$$
\begin{aligned}
    f(x, y, z) &= \max_{1 \leqslant i \leqslant n} \left( x_i - x + y_i - y + z_i - z \right) \newline
    &= \max_{1 \leqslant i \leqslant n} \left( x_i + y_i + z_i - x - y - z \right) \newline
    &= \max_{1 \leqslant i \leqslant n} \left( x_i + y_i + z_i \right) - x - y - z
\end{aligned}
$$

第二行到第三行的转化是因为 $x, y, z$ 对于我们当前所求的这部手机来说都是定值。因此，我们需要维护的就只有求所有手机三种参数之和的最大值这一部分内容了。当然，上面列举的只是当三种参数都出现第二种情况的时候的处理方式。而对于第一种情况，由于计入值为 $0$，我们可以直接从上面的表达式里删掉对应的项，并维护在删掉对应种类参数的情况下所有手机的剩下的参数之和的最大值即可。由于这三种参数都可能独立出现第一种情况和第二种情况，所以最终最多会有 $2^3 = 8$ 种状态的最大值需要维护。每一种状态维护最大值只需要 $\mathcal{O}\left(n\right)$ 的复杂度，所以可以直接通过暴力枚举解决。

在预处理完所需的最大值后，接下来就是根据我们推出的公式以及求得的最大值，对每一部手机的 opportunity cost 进行计算。在上面的暴力枚举过程中，会出现一种情况，即某一手机的某种参数并非同种参数中的最大值，但在预处理时却被统计成 $0$ 的情况。所以我们在计算时也需要把 $8$ 种情况全部跑一遍，求这 $8$ 种情况的三种参数计入值之和的最大值。同时，由于当某一手机的某种参数 $p$ 为同种参数 $p_i$ 中的最大值时，一定有 $\forall 1 \leqslant i \leqslant n,~ p_i - p \leqslant 0$，所以在我们统计最大值时，其中 $< 0$ 的情况会被其他情况中的 $0$ 覆盖掉，所以这样做不会影响答案的正确性。

代码实现上，注意三种参数之和最高能达到 $3 \times 10^9$，所以需要使用 `unsigned int` 或 `long long` 等类型存储。

{{< collapse "点击查看全部代码" >}}

``` cpp
#include <limits>
#include <iostream>

#define endl '\n'

using std::cin;
using std::cout;

using u32 = unsigned int;

const int MAX_N = 2e5 + 5;
const int CASES = 1 << 3;

u32 N, x[MAX_N], y[MAX_N], z[MAX_N];
u32 max[CASES];

void prep() {
    for (u32 i = 0; i < CASES; i++)
        for (u32 j = 1, sum; j <= N; j++) {
            sum = (i >> 0 & 1) ? x[j] : 0;
            sum += (i >> 1 & 1) ? y[j] : 0;
            sum += (i >> 2 & 1) ? z[j] : 0;
            max[i] = std::max(max[i], sum);
        }
}

int main() {
    std::ios::sync_with_stdio(false);
    cin.tie(nullptr);

    cin >> N;
    for (u32 i = 1; i <= N; i++)
        cin >> x[i] >> y[i] >> z[i];

    prep();

    u32 ans1 = std::numeric_limits<u32>::max(), ans2 = 0;
    for (u32 i = 1, cost; i <= N; i++) {
        cost = 0;

        for (u32 j = 0, sum; j < CASES; j++) {
            sum = (j >> 0 & 1) ? x[i] : 0;
            sum += (j >> 1 & 1) ? y[i] : 0;
            sum += (j >> 2 & 1) ? z[i] : 0;
            cost = std::max(cost, max[j] - sum);
        }

        if (cost < ans1) ans1 = cost, ans2 = i;
    }

    cout << ans1 << ' ' << ans2 << endl;
    return fflush(stdout), 0;
}
```

{{< /collapse >}}

## Running Away From the Barn

- 出处：USACO 2012 December Contest \(Gold\)
- 官方题面及评测地址：[英语](https://usaco.org/index.php?page=viewproblem2&cpid=213)
- 中文题面及评测地址：[洛谷 P3066](https://www.luogu.com.cn/problem/P3066)

看到数据范围，可以看到 $p_i < i$。也就是说，$\forall v \in T_u, u > v$。因此，树上任何一个节点到达其任意一个祖先的路径上所有节点的编号一定都是单调递减的。因此，我们可以倒序地维护一个差分数组，对于每一个节点，以其为起始端点，将其向根方向移动 $T$ 的距离内所能到达最远的节点作为结束端点，利用差分对起始端点到结束端点的路径上所有的节点被覆盖的次数进行区间 $+1$，即可得到每个节点最终被覆盖的总次数，即为答案。

这里的差分方法和普通的差分不同的地方在于：

- 差分维护的区间在数组中并不是连续的，普通数组中元素之间通过固定的下标之差来维护的差分，在本题中需要通过树上节点之间的亲子关系来维护；
- 执行区间 $+1$ 操作时，差分 $+1$ 的点其实是编号较大的节点，$-1$ 的点反而是编号较小的节点。这一点其实很好理解。我们希望，被差分修改的位置正好代表从某个节点到达其特定祖先的路径上的所有节点，因此我们需要从子节点出发朝根节点的方向进行修改。如果从祖先的位置开始修改，那么最终，以祖先为根节点的子树里，只有路径上的节点的差分最后 $-1$ 了，其他与这条路径无关的节点都会被额外 $+1$，导致最终结果错误。

由于数据范围为 $2 \times 10^5$，我们可以通过树上倍增来维护每个节点的各级祖先，由此在 $\mathcal{O} \left(\log N\right)$ 复杂度内找到查分的两个端点。所以这种解法的整体复杂度为 $\mathcal{O} \left(N \log N\right)$，可以通过本题。

{{< collapse "点击查看全部代码" >}}

``` cpp
#include <iostream>

#define endl '\n'

using std::cin;
using std::cout;

using i64 = long long;

const int MAX_N = 2e5 + 5;

int N, fa[MAX_N][20];
i64 T, depth[MAX_N], d[MAX_N];

int main() {
    std::ios::sync_with_stdio(false);
    cin.tie(nullptr);

    cin >> N >> T;

    d[1] = 1;

    for (int i = 2, p; i <= N; ++i) {
        i64 w;
        cin >> p >> w;

        fa[i][0] = p;
        depth[i] = depth[p] + w;

        for (int j = 1; j <= 18; ++j)
            fa[i][j] = fa[fa[i][j - 1]][j - 1];

        int u = i;
        for (int j = 18; ~j; --j)
            if (depth[i] - depth[fa[u][j]] <= T)
                u = fa[u][j];

        ++d[i], --d[fa[u][0]];
    }

    for (int i = N; i; --i)
        d[fa[i][0]] += d[i];

    for (int i = 1; i <= N; ++i)
        cout << d[i] << endl;

    return fflush(stdout), 0;
}
```

{{< /collapse >}}
