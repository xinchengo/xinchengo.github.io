# 概率 DP

# P1654 OSU!

[题目链接](https://www.luogu.com.cn/problem/P1654)

大意：一个长度为 n $(n\leq 10^5)$ 的 01 串，每一位上出现 1 有一个特定的概率，连续的 $x$ 个 $1$ 可以贡献  $x^3$ 的分数，求分数和的期望。

  

解法：由于 n 的范围，考虑线性 DP，逐位枚举。增添一个 1 后，因为连续 1 的个数加 1，如果第 $i-1$ 个数的期望是（$F_{i-x-1}$ 为 $i$ 前连续的 $x$ 个 $1$ 之前的期望，$\prod_{k=i-x}^{i-1} p_k$ 为 $i$ 前连续的 $x$ 个 $1$ 出现的概率）：$$F_{i-1} = \sum_{x=1}^{i-1}(F_{i-x-1} + \prod_{k=i-x}^{i-1} p_k \cdot x^3)$$

那么下一个数选 0 的期望是：$$F_{i_{0}} = (1-p_i)\cdot \sum_{x=1}^{i-1}(F_{i-x-1} + \prod_{k=i-x}^{i-1} p_k \cdot x^3)$$

选 1 的期望是：
$$
\begin{align*}

F_{i_{1}} &= p_i\cdot\sum_{x=1}^{i-1}(F_{i-x-1} +  \prod_{k=i-x}^{i-1} p_k \cdot (x+1)^3) \\

&= p_i\cdot\sum_{x=1}^{i-1}(F_{i-x-1} +  \prod_{k=i-x}^{i-1} p_k \cdot (x^3+3\cdot x^2+3\cdot x+1)) \\

&= p_i\cdot\sum_{x=1}^{i-1}(F_{i-x-1} +  \prod_{k=i-x}^{i-1} p_k \cdot x^3+3\cdot \prod_{k=i-x}^{i-1}x^2+3\cdot\prod_{k=i-x}^{i-1} x+1)

\end{align*}

$$

总期望是：

$$

\begin{align*}F_i &= F_{i_0}+F_{i_1} \\

&= (1-p_i)\cdot \sum_{x=1}^{i-1}(F_{i-x-1} + \prod_{k=i-x}^{i-1} p_k \cdot x^3) + p_i\cdot\sum_{x=1}^{i-1}(F_{i-x-1} +  \prod_{k=i-x}^{i-1} p_k \cdot x^3+3\cdot \prod_{k=i-x}^{i-1}x^2+3\cdot\prod_{k=i-x}^{i-1} x+1)) \\

&= \sum_{x=1}^{i-1}(F_{i-x-1} + \prod_{k=i-x}^{i-1} p_k \cdot x^3) + p_i\cdot\sum_{x=1}^{i-1}(3\cdot \prod_{k=i-x}^{i-1}x^2+3\cdot\prod_{k=i-x}^{i-1} x+1) \\

&= F_{i-1} + p_i\cdot\sum_{x=1}^{i-1}(3\cdot \prod_{k=i-x}^{i-1}x^2+3\cdot\prod_{k=i-x}^{i-1} x+1)

\end{align*}

$$

设三个数组：
 m, 
$$

\begin{align*}x1_i &= \sum_{x=1}^{i}\prod_{k=i-x}^{i-1} x \\

x2_i &= \sum_{x=1}^{i-1}\prod_{k=i-x}^{i} x^2 \\ 

F_i
\end{align*}

$$