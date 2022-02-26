# 2-SAT 问题
# 模板题
- [P4782 【模板】2-SAT 问题](https://www.luogu.com.cn/problem/P4782)
- [P5782 \[POI2001\] 和平委员会 ](https://www.luogu.com.cn/problem/P5782)
- [P4171 \[JSOI2010\] 满汉全席](https://www.luogu.com.cn/problem/P4171)

## P3825 \[NOI2017\] 游戏
[题目链接](https://www.luogu.com.cn/problem/P3825)"

大意：小 L 有 A、B、C 三辆赛车，n 场游戏中有 d 场 $(d \leq 8, n \leq 5 \times 10^4)$ 不限制赛车，其余的场次会禁止某一辆赛车。还有 m 条 $(m \leq 10^4)$ 规则限制如果在场次 $i$ 使用某一赛车，在场次 $j$ 必须使用某种赛车。判断是否有解，若有解，输出一种方案。

解法：考虑到绝大多数场次只能使用两车之一，枚举 $d$ 场不限赛车的场次限制 A 或限制 B，将原问题转化为 2-SAT 问题， 时间复杂度 $O((n+m)\cdot 2^d)$。

## P3209 \[HNOI2010\] 平面图判定
[题目链接](https://www.luogu.com.cn/problem/P3209)

大意：T 组数据 $(T\leq 300)$，给你一个存在哈密顿回路的图和其哈密顿回路，判断该图是否为平面图。$N\leq 200$ 。

解法：为了简化题目，我们可以用平面图定理 $m \leq 3\times n - 6$，初步排除不符合要求的情况。一条边可以在环外，也可以在环内，我们可以对边的矛盾建立关系，从而转化为 2-SAT 问题求解。