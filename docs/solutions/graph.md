# 图论
## 图的连通性
### 模板题
- [P2341 [USACO03FALL / HAOI2006] 受欢迎的牛 G](https://www.luogu.com.cn/problem/P2341)

同求唯一的出度为 0 的强连通分量的大小。

??? note "参考代码"
	```cpp
	--8<-- "code/P2341.cpp"
	```

- [P2863 [USACO06JAN]The Cow Prom S](https://www.luogu.com.cn/problem/P2863)

同求点数大于 1 的强连通分量的个数。

??? note "参考代码"
	```cpp
	--8<-- "code/P2863.cpp"
	```

### P2746 \[USACO5.3\]校园网Network of Schools
[题目链接](https://www.luogu.com.cn/problem/P2746)

大意：一张有 $N$ $(2\leq N\leq 100)$个节点的有向图，求出最小的点集大小，使得对于任何不在点集中的节点，都存在一条点集中某点到该点的简单路径；并求出最小的加边数目，使得该图强连通。

解法：缩点后的图是一个森林，前者的答案就是森林树根的数目，即入度为 0 的强连通分量的个数；要通过加边的方式使森林强连通，就要把所有叶节点都连到根节点上，并使所有根节点相连，此时一个很好的方法就是将各个连通块首尾相连，剩余的节点依次连入环中，需要加边的条数就是入度为 0 的强连通分量数与出度为 0 的强连通分量数的较大值。

??? note "参考代码"
	```cpp
	--8<-- "code/P2746.cpp"
	```
## 2-SAT 问题
### 模板题
- [P4782 【模板】2-SAT 问题](https://www.luogu.com.cn/problem/P4782)
??? note "参考代码"
	```cpp
	--8<-- "code/P4782.cpp"
	```
- [P5782 \[POI2001\] 和平委员会 ](https://www.luogu.com.cn/problem/P5782)
??? note "参考代码"
	```cpp
	--8<-- "code/P5782.cpp"
	```
- [P4171 \[JSOI2010\] 满汉全席](https://www.luogu.com.cn/problem/P4171)
??? note "参考代码"
	```cpp
	--8<-- "code/P4171.cpp"
	```

### P3825 \[NOI2017\] 游戏
[题目链接](https://www.luogu.com.cn/problem/P3825)
	
大意：小 L 有 A、B、C 三辆赛车，n 场游戏中有 d 场 $(d \leq 8, n \leq 5 \times 10^4)$ 不限制赛车，其余的场次会禁止某一辆赛车。还有 m 条 $(m \leq 10^4)$ 规则限制如果在场次 $i$ 使用某一赛车，在场次 $j$ 必须使用某种赛车。判断是否有解，若有解，输出一种方案。

解法：考虑到绝大多数场次只能使用两车之一，枚举 $d$ 场不限赛车的场次限制 A 或限制 B，将原问题转化为 2-SAT 问题， 时间复杂度 $O((n+m)\cdot 2^d)$。

??? note "参考代码"
	```cpp
	--8<-- "code/P3825.cpp"
	```

### P3209 \[HNOI2010\] 平面图判定
[题目链接](https://www.luogu.com.cn/problem/P3209)

大意：T 组数据 $(T\leq 300)$，给你一个存在哈密顿回路的图和其哈密顿回路，判断该图是否为平面图。$N\leq 200$ 。

解法：为了简化题目，我们可以用平面图定理 $m \leq 3\times n - 6$，初步排除不符合要求的情况。一条边可以在环外，也可以在环内，我们可以对边的矛盾建立关系，从而转化为 2-SAT 问题求解。

??? note "参考代码"
	```cpp
	--8<-- "code/P3209.cpp"
	```

## 点分治
### 模板题
- [P3806 【模板】点分治1](https://www.luogu.com.cn/problem/P3806)
??? note "参考代码"
	```cpp
	--8<-- "code/P3806.cpp"
	```
- [P4178 Tree](https://www.luogu.com.cn/problem/P4178)

## 树上启发式合并

[^note1]: [木每立兄豪《dsu on tree(树上启发式合并)算法总结+习题》](https://blog.csdn.net/qq_43472263/article/details/104150940)

### 概念
见[树链剖分](/notes/topics/hld)。本节参考了这篇文章[^note1]。

### CF600E Lomsat gelral
[题目链接](https://www.luogu.com.cn/problem/CF600E)
	
大意：给定一个 n 个节点的树 $(n \leq 10^5)$ ，每个节点都有一个颜色，求每个节点的子树上出现次数最多的颜色的编号和（可能有多种颜色出现次数最多）。

解法：由于无法记录每个点的子树每一颜色的出现次数，我们考虑使用一个数组，先遍历所有的轻儿子节点形成的子树，统计答案但是不保留数据，然后遍历重儿子，统计答案并且保留数据，最后再遍历轻儿子以及父节点，合并重儿子统计过的答案。

??? note "参考代码"
	```cpp
	--8<-- "code/CF600E.cpp"
	```

该题是记录出现次数题目的代表。

### CF208E Blood Cousins
[题目链接](https://www.luogu.com.cn/problem/CF208E)

大意：给你一片 n 个节点 $(n \leq 10^5)$ 的树，每次询问一个点与多少个点拥有共同的 K 级祖先。

解法：本题同问以给定点为根，深度为 k 的点的个数减一。维护一个 dis 数组记录给定深度的点的个数，方法同上。

??? note "参考代码"
	```cpp
	--8<-- "code/CF208E.cpp"
	```

该题是记录深度题目的代表。

### CF570D Tree Requests
[题目链接](https://www.luogu.com.cn/problem/CF570D)

大意：给定一个以 1 为根的 n 个结点 $(n \leq 5\times 10^5)$ 的树，每个点上有一个字母（`a`-`z`），树的深度从 1 开始计算。每次询问 a, b 查询以 a 为根的子树内深度为 b 的结点上的字母重新排列之后是否能构成回文串。

解法：判断若干个字母能构成回文串的条件是最多只能有一个出现奇数次的字母，所以这一题就等同于求某一点以下给定深度出现字母的个数。维护一个二维数组 dis 记录给定深度每一字母出现的次数，树上启发式合并即可。该题需要注意 IO 常数。

!!! warning
	此题可能要注意 IO 常数。

??? note "参考代码"
	```cpp
	--8<-- "code/CF570D.cpp"
	```

### CF741D Arpa’s letter-marked tree and Mehrdad’s Dokhtar-kosh paths
[题目链接](https://www.luogu.com.cn/problem/CF741D)

大意：给定一棵根为 1 的 n 个节点 $(n \leq 5\times 10^5)$ 的树，每条边上有一个字符（a-v共22种）。一条简单路径被称为 Dokhtar-kosh 当且仅当路径上的字符经过重新排序后可以变成一个回文串。求每个子树中最长的 Dokhtar-kosh 路径的长度。

解法：能否构成回文串的判定同 [[dsu-on-tree#cf570d-tree-requests|CF570D]]。由于 n 的数量小于 22，可以考虑状态压缩。使用一个 f 数组维护当前枚举子树下，所有状态出现的最大深度，树上启发式合并即可。

??? note "参考代码"
	```cpp
	--8<-- "code/CF741D.cpp"
	```

该题是树上启发式合并的压轴题。



