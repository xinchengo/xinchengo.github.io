# 其他题目
## AT2362 [AGC012B] Splatter Painting
[题目链接](https://www.luogu.com.cn/problem/AT2362)

大意：给一个 n 个点 m 条边的无向图，有 q 次操作 第 i 次操作，给出 v，d，c，把所有到点 v 的距离不超过 d 的点都染上颜色 c，问最后每个点的颜色。$(n, m, q, c \leq 10^5, d \leq 10)$

解法：考虑到影响每个点颜色的是最后一次操作，我们用数组 $f_{i,j}$ 记录第 i 个点，能影响距离为 j 的最大操作序号。由于 $f_{u,d} \rightarrow f_{v,d-1}$，我们可以按照 d 的倒序，枚举每个点的状态，并转移至相邻边即可，时间复杂度 $O(d\cdot (n+m))$
