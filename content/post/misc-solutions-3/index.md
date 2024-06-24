---
title: "「美食家」题解"
description: "来自 NOI 2020 的一道 DP 优化好题。"
slug: sol-luogu6772
date: 2024-06-24 00:00:00+0800
math: true
categories:
  - editorial
tags:
  - 动态规划
  - 线性代数
---

- 出处：NOI 2020
- 题面与评测地址：[洛谷 P6772](https://www.luogu.com.cn/problem/P6772) & [LibreOJ \#3339](https://loj.ac/p/3339)
- 参考题解：[By Plozia](https://www.cnblogs.com/Plozia/p/16773840.html)

## 暴力部分分

先考虑朴素做法如何实现。根据题目的设问方式可以发现，这道题需要用到动态规划进行解决。于是我们可以设计这样一个状态：$f_{t, v}$，它表示第 $t$ 天走到了城市 $v$ 时所能持有的最大愉悦值。令图中起点为 $u$、终点为 $v$、耗时为 $w$ 的有向边为 $(u, v, w)$，则该状态的转移满足：

$$
f_{t, v} = \max_{(u, v, w) \in E} \left\lbrace f_{t - w, u} \right\rbrace + c_v + \text{fest}_{t, v}
$$

其中，$\text{fest}_{t, v}$ 为在第 $t$ 天城市 $v$ 所举办的美食节的加成。若这一天在这座城市没有美食节举办，该值等于 $0$。当然，实际编码时还要考虑 $i - w$ 不小于 $1$ 等等问题，这里就不详述了。仅仅是这样暴力的 $\mathcal{O}\left (Tn\right )$ 复杂度的 DP，也可以拿到 40 分，这充分说明了好好打部分分的重要性 😮

## 矩阵优化的引入

假设现在所有边的边权 $w$ 均为 $1$，且暂时不考虑美食节的影响，即 $k = 0$。那么原问题简化为：在一个 $n$ 点 $m$ 边的图中，恰好经过 $p$ 条边的最长路。沿用此前的转移，我们有：

$$
\begin{aligned}
f_{t, v} &= \max \left\lbrace f_{t - w, u} \right\rbrace + c_v + \text{fest} _ {t, v} \newline
&= \max \left\lbrace f_{t - 1, u} \right\rbrace + c_v \newline
&= \max \left\lbrace f_{t - 1, u} + c_v \right\rbrace
\end{aligned}
$$

这里的 $c_v$ 可看作是每个点的点权，考虑将其转化为边权，令 $d_{u, v}$ 表示起点为 $u$、终点为 $v$ 的边中，终点 $v$ 的愉悦值 $c_v$。那么上式就可以写为

$$
f_{t, v} = \max\left\lbrace f_{t - 1, u} + d_{u, v} \right\rbrace
$$

你可能会发现，这样的形式与矩阵乘法有些相似——两个 $f$ 的第一项都与 $t$ 相关，后面的 $f$ 的第二项与 $d$ 的第一项相同，前面的 $f$ 的第二项与 $d$ 的第二项相同，就像一个 $(t - 1) \times u$ 的矩阵在跟一个 $u \times v$ 的矩阵做乘法，得到了一个 $t \times u$ 的矩阵。于是，我们尝试将上述形式推广成一个更通用的结论。

定义一种矩阵运算 $\otimes$。若有 $A \otimes B = C$，则 $A$ 的列数和 $B$ 的行数相同，$C$ 的行数与 $A$ 相同，列数与 $B$ 相同，且对矩阵 $C$ 中的每一个元素，都有：$C_{i, j} = \max\left\lbrace A_{i, k} + B_{k, j} \right\rbrace$。根据取最大值 $\max$ 运算的结合律，可以证明，该运算同样满足结合律。

由于图中点的数量是固定的，所以 $d$ 可以视为一个行列数相等的矩阵，又由于 $\otimes$ 运算具有结合律，因此 $d$ 可以通过对数复杂度的「矩阵快速幂」（但其中的「乘法」实际上是 $\otimes$ 运算）进行迭代。在这样的观点下，我们可以将 $f_t$ 视为一个行向量，其每一个元素为 $f_{t, v}$，并得到一个非常神奇 ✨ 的转移方程：

$$
f_t = f_0 \otimes d^t
$$

下面用人话来解释一下这个方程的意思：

> 根据我们刚才定义的运算所具有的性质，我们可以将同一类 $d$ 的，从初始状态 $t_0 = 0$ 到特定状态 $t$ 的贡献累积起来，但该操作是通过反复进行 $\otimes$ 运算，用「矩阵快速幂」的形式将转移本身优化为 $\mathcal{O}(\log t)$ 复杂度，由此计算出 $d^t$ 来实现的。接下来，我们再通过一次 $\otimes$ 运算将计算出来的贡献叠加到初始状态 $f_0$ 上，由此获得 $f_t$ 的状态。
>
> 由于 $f$ 是行向量，其行数可视为 $1$，列数可视为元素数 $n$；而 $d$ 为 $n \times n$ 矩阵，所以 $f_0 \otimes d^t$ 所得到的结果的行数依然为 $1$，列数依然为 $n$，也就依然符合 $f$ 的行向量的形式。

我们可以很容易地根据 $c$ 数组构建 $d$ 矩阵（没有边相连的地方，值为 $-\infty$），那么现在的任务就是确定初始状态 $f_0$ 实际上应该是什么了。考虑 $f$ 的定义：$f_{t, v}$ 表示第 $t$ 天走到了城市 $v$ 时所能持有的最大愉悦值。由题目可知，主人公最初（第 $0$ 天）在城市 $1$，根据数据范围，第 $0$ 天不会有美食节（$1 \leqslant t_i \leqslant 10^9$），所以 $f_0$ 的元素中只有一个有值，即 $f_{0, 1} = c_1$，其他地方的值都是 $-\infty$。

## 矩阵优化的推广

还记得吗？上一小节里的推导都是基于 $w = 1$ 的前提的。而题目中 $w$ 的值可能在 $[1, 5]$ 之间变化，所以我们还需要思考如何将原来的结论应用到题目中来。当 $w = 1$ 时，我们对 $d$ 重复进行的 $\otimes$ 运算，本质上是以 $w = 1$ 为单位在对贡献进行叠加。或者说，如果 $w \in [1, 5]$，我们可以考虑将这些边权拆成 $1$ 到 $5$ 条 $w = 1$ 的边，在原有的这条边的起点和终点之间再加入 $w - 1$ 个点来承接这些新加的边，再将其贡献进行叠加。说白了就是，如果将此前我们推导时使用的 $d$ 写成 $d^1$，那么现在一条边权为 $w$ 的边的贡献就是 $d^w$，只是我们将指数 $w$ 掺到了整体的指数里进行计算。

不过这样拆有个小问题。极端情况下，对于所有的 $m_{\max} = 501$ 条边，每一条边的边权都为 $w_{\max} = 5$，那么就需要新增 $\left (w_{\max} - 1 \right ) m_{\max} = 2004$ 个点。别忘了我们所需要用到的矩阵快速幂中，对一个具有 $n$ 点的图的邻接矩阵执行矩阵乘法本身也是具有 $\mathcal{O}(n^3)$ 的复杂度的！假如我们的点数总共为 $\left[ n_{\max} + \left( w_{\max} - 1 \right) m \right] _ {\max} = 2054$，那么光执行一次矩阵乘法就已经要超时了。有没有一种拆分的方法能够较小地影响实际的点数呢？

我们考虑这样一种拆法：将每一个点拆成 $w$ 个点，这些点之间都用一条边权为 $0$ 的边连接。{{< highlights >}}对于每一条边 $(u, v, w)$，让 $u$ 拆出来的第 $w$ 个点 $u_w$ 与 $v$ 拆出来的第一个点 $v_1$ 之间用一条边权为 $w$ 的边相连。{{< /highlights >}}这样做，首先只会用到 $w_{\max}n = 5n$ 个点（由于每条边的 $w \in [1, 5]$，$u$ 拆出来每个点上都可能有向其他点的连边，所以总共的点数其实就是钉死了的 $5n$），至少让矩阵乘法的复杂度降到了一个可以控制的范围。至于这样做为什么正确，其实就是利用了此前就已经明确的「无论指数是多少最后都会揉到一起去」的观点，并且无论指数是多少所经过的转移都恰好是 $w$ 步。

这样，我们就成功将推得的结论应用到了 $w \in [1, 5]$ 的情况。摆在我们面前的难题越来越少了！

## 额外因素的考虑

别忘了还有「美食节」这一设定！回收伏笔啦！我们要想办法将美食节的贡献加入我们现有的体系中，该如何做呢？首先可以肯定的是，美食节显然没有办法直接加入我们的矩阵转移，因为美食节的时间地点和贡献大小都是不确定的。但是我们可以根据美食节的具体信息，在转移过程中修改对应时间、城市的贡献值。

由于转移过程是根据 $t$ 从小到大进行的，所以我们可以将所有美食节 $\left(t, x, y \right)$ 根据 $t$ 进行排序，并在转移时，每次执行「矩阵快速幂」直到下一个美食节发生的时间 $t$，并将这个美食节 $\left(t, x, y \right)$ 的贡献 $y$ 添加到第 $t$ 天的城市 $x$ 上，然后继续通过「矩阵快速幂」从当前时间 $t$ 为起点进行转移。整个过程就像是乘坐游轮，在海上航行的过程就是执行矩阵快速幂来优化转移时间复杂度的过程，而每次靠岸、下船游玩则是在当前位置叠加美食节的贡献的、时间复杂度朴素的过程。

## 最后一公里

不考虑美食节的情况下，拆点之后，「矩阵乘法」执行一次的复杂度为 $\mathcal{O}(N^3)$，其中 $N = 5n$，所以「矩阵快速幂」的复杂度就为 $\mathcal{O}(N^3 \log T)$。但由于我们在考虑美食节之后，由于将原有的一整个长度为 $\log T$ 的快速幂段分解成了最多 $201$ 个小段（$k_{\max} = 200$），所以实际的复杂度为 $\mathcal{O}\left(N^3 \sum\log\left(t_i - t_{i - 1}\right)\right)$。由于 $N_{\max} = 250$，有 ${N ^ 3}_{\max} = 1.5625 \times 10^7$，再乘上一个 $200$ 很显然依然会超时。

怎么办呢？DP 的过程已经想不出任何可以优化的地方了，此时我们考虑再次优化「矩阵快速幂」。我们知道，此前我们说「矩阵乘法」执行一次的复杂度为 $\mathcal{O}(N^3)$，是因为是在对一个 $N \times N$ 的邻接矩阵进行计算，其结果最终落脚到转移矩阵 $f_t$ 上，而 $f_t$ 是一个行向量。如果能够直接将操作在 $f_t$ 上面执行，不借用一整个邻接矩阵，我们或许就可以将原本 $\mathcal{O}(N^3)$ 的复杂度降低到 $\mathcal{O}(N^2)$，从而通过此题。

由于 $\otimes$ 运算具有结合律，所以对原本的 DP 转移式 $f_t = f_0 \otimes d^t$，在考虑美食节后即 $f_{t_i} = f_{t_{i - 1}} \otimes d^{t_i - t_{i - 1}}$。我们可以将快速幂 $d^{t_i - t_{i - 1}}$ 中分解出来的若干个 $d^{2^k}$ 直接 $\otimes$ 乘到 $f_{t_{i - 1}}$ 上面！由于 $f_t$ 是行向量，相对于整个邻接矩阵来说少了一维，就可以像上面所说的实现 $\mathcal{O}(N^2)$ 转移了！为此，我们其实只需要在得出整个邻接矩阵 $d$ 后，用 $\mathcal{O}(N^3 \log T)$ 的复杂度将所有满足 $2 ^ k \leqslant T$ 的 $d^{2^k}$ 全部计算一遍就好了。计算出来之后，两次美食节之间就可以 $\mathcal{O}\left(N^2 \sum\log\left(t_i - t_{i - 1}\right)\right)$ 复杂度转移，总共 $k$ 次，所以总复杂度就为：

$$
\mathcal{O} \left (k \log k + N^3 \log T + k \cdot N^2 \sum\log\left(t_i - t_{i - 1}\right) \right ).
$$

## 代码

{{< collapse 点击查看全部代码 >}}

```cpp
#include <tuple>
#include <iostream>
#include <algorithm>

using LL = long long;
using TUP = std::tuple<int, int, int>;

const int MAX_N = 55;
const int MAX_LG_T = 35;
const int MAX_K = 2e2 + 5;
const int MAX_N_S = MAX_N * 5;
const LL LL_MIN = 0xC0C0C0C0C0C0C0C0;

struct Matrix {
  LL data[MAX_N_S][MAX_N_S];
  int row, col;

  void fill(const LL x) {
    for (int i = 1; i <= row; ++i)
      for (int j = 1; j <= col; ++j)
        data[i][j] = x;
  }

  Matrix operator*(const Matrix& B) const {
    Matrix C(row, B.col, LL_MIN);
    for (int i = 1; i <= row; ++i)
      for (int j = 1; j <= col; ++j)
        for (int k = 1; k <= B.col; ++k)
          C.data[i][k] = std::max(C.data[i][k], data[i][j] + B.data[j][k]);
    return C;
  }

  Matrix() {}
  Matrix(const int _r, const int _c, const LL x) :
    row(_r), col(_c) { fill(x); }
};

int n, m, T, K;
int c[MAX_N], logT;

TUP fest[MAX_K];
Matrix G, pow2[MAX_LG_T], vec;

void prep() {
  pow2[0] = G;
  for (int i = 1; i <= logT; ++i) pow2[i] = pow2[i - 1] * pow2[i - 1];
}

void power(Matrix& A, int b) {
  if (b == 0) return;
  for (int i = 0; i <= logT; ++i)
    if ((b >> i) & 1)
      A = pow2[i] * A;
}

int main() {
  std::ios::sync_with_stdio(false);
  std::cin.tie(nullptr);

  std::cin >> n >> m >> T >> K;

  logT = std::__lg(T);               // 计算 log(T) 的值
  G = Matrix(n * 5, n * 5, LL_MIN);  // 创建邻接矩阵并用负无穷填充

  // 初始化每个点所拆成的五个点之间的连边
  for (int i = 1; i <= n; ++i)
    for (int w = 1; w < 5; ++w)
      G.data[n * w + i][n * (w - 1) + i] = 0;

  for (int i = 1; i <= n; ++i) std::cin >> c[i];

  for (int i = 1, u, v, w; i <= m; ++i) {
    std::cin >> u >> v >> w;
    G.data[v][n * (w - 1) + u] = c[u];  // 按照边的信息把边连到拆开的点上
  }

  for (int i = 1; i <= K; ++i) {
    auto& [t, x, y] = fest[i];
    std::cin >> t >> x >> y;
  }

  prep();  // 二进制预处理邻接矩阵

  std::sort(fest + 1, fest + K + 1);  // 对美食节进行排序

  Matrix vec(n * 5, 1, LL_MIN);  // 创建 f 行向量
  vec.data[1][1] = 0;            // 初始化

  // 分段转移
  int current_time = 0;

  for (int i = 1; i <= K; ++i) {
    auto [time, city, joy] = fest[i];
    power(vec, time - current_time);
    vec.data[city][1] += joy;
    current_time = time;
  }

  power(vec, T - current_time);

  // 输出答案
  LL ans = vec.data[1][1] + c[1];                  // 由于转移不考虑第一个城市，需要额外加上
  std::cout << (ans < 0 ? -1 : ans) << std::endl;  // 输出答案或者报告无解

  return 0;
}
```

{{< /collapse >}}
