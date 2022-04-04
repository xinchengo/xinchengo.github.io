# 树上启发式合并

## 概念
见[树链剖分](/notes/topics/hld)

## CF600E Lomsat gelral
[题目链接](https://www.luogu.com.cn/problem/CF600E)
	
大意：给定一个 n 个节点的树 $(n \leq 10^5)$ ，每个节点都有一个颜色，求每个节点的子树上出现次数最多的颜色的编号和（可能有多种颜色出现次数最多）。

解法：由于无法记录每个点的子树每一颜色的出现次数，我们考虑使用一个数组，先遍历所有的轻儿子节点形成的子树，统计答案但是不保留数据，然后遍历重儿子，统计答案并且保留数据，最后再遍历轻儿子以及父节点，合并重儿子统计过的答案。

??? note "参考代码"
	```cpp
	--8<-- "code/CF600E.cpp"
	```

该题是记录出现次数题目的代表。

## CF208E Blood Cousins
[题目链接](https://www.luogu.com.cn/problem/CF208E)

大意：给你一片 n 个节点 $(n \leq 10^5)$ 的树，每次询问一个点与多少个点拥有共同的 K 级祖先。

解法：本题同问以给定点为根，深度为 k 的点的个数减一。维护一个 dis 数组记录给定深度的点的个数，方法同上。

??? note "参考代码"
	```cpp
	--8<-- "code/CF208E.cpp"
	```

该题是记录深度题目的代表。

## CF570D Tree Requests
[题目链接](https://www.luogu.com.cn/problem/CF570D)

大意：给定一个以 1 为根的 n 个结点 $(n \leq 5\times 10^5)$ 的树，每个点上有一个字母（`a`-`z`），树的深度从 1 开始计算。每次询问 a, b 查询以 a 为根的子树内深度为 b 的结点上的字母重新排列之后是否能构成回文串。

解法：判断若干个字母能构成回文串的条件是最多只能有一个出现奇数次的字母，所以这一题就等同于求某一点以下给定深度出现字母的个数。维护一个二维数组 dis 记录给定深度每一字母出现的次数，树上启发式合并即可。该题需要注意 IO 常数。

!!! warning
	此题可能要注意 IO 常数。

??? note "参考代码"
	```cpp
	--8<-- "code/CF570D.cpp"
	```

## CF741D Arpa’s letter-marked tree and Mehrdad’s Dokhtar-kosh paths
[题目链接](https://www.luogu.com.cn/problem/CF741D)

大意：给定一棵根为 1 的 n 个节点 $(n \leq 5\times 10^5)$ 的树，每条边上有一个字符（a-v共22种）。一条简单路径被称为 Dokhtar-kosh 当且仅当路径上的字符经过重新排序后可以变成一个回文串。求每个子树中最长的 Dokhtar-kosh 路径的长度。

解法：能否构成回文串的判定同 [[dsu-on-tree#cf570d-tree-requests|CF570D]]。由于 n 的数量小于 22，可以考虑状态压缩。使用一个 f 数组维护当前枚举子树下，所有状态出现的最大深度，树上启发式合并即可。

??? note "参考代码"
	```cpp
	--8<-- "code/CF741D.cpp"
	```

该题是树上启发式合并的压轴题。

## 参考资料
[^note1]: [木每立兄豪《dsu on tree(树上启发式合并)算法总结+习题》](https://blog.csdn.net/qq_43472263/article/details/104150940)


