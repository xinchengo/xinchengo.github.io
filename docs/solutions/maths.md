# 数学
## 组合数学

### P2675 《瞿葩的数字游戏》T3-三角圣地
[题目链接](https://www.luogu.com.cn/problem/P2675)

大意：$p$ 是一个 1 到 n $(0\leq n\leq 10^6)$ 的排列，求 $\sum_{i=0}^{n-1}C_{n-1}^{i}\cdot p_i$ 的最大值 $\bmod 10007$ 的结果。

解法：Lucas 定理求组合数模拟计算即可。

??? note "参考代码"
    ```cpp
    --8<-- "code/P2675.cpp"
    ```

### P4345 [SHOI2015]超能粒子炮·改
[题目链接](https://www.luogu.com.cn/problem/P4345)

大意：求 $\sum_{i=0}^{k}C_n^i\bmod 2333$ $(n,k\leq10^{18})$ 的值。

解法：Lucas 定理的本质是按位分组相乘。可以考虑递归计算一些共同部分，从而节省时间。记 $f(n,k)$ 为答案，则：

![P4345-formula.jpg](/_static/images/P4345-formula.jpg)

该解法需要的组合数都较小，可以预处理。

??? note "参考代码"
    ```cpp
    --8<-- "code/P4345.cpp"
    ```

## 同余
### P3306 [SDOI2013] 随机数生成器
[题目链接](https://www.luogu.com.cn/problem/P3306)

大意：给定 $p,a,b,x_1,t$，数列 $x$ 满足递推关系 $x_{i+1} \equiv a \times x_i+b \pmod p$，求 $x$ 数列中最早出现 $t$ 的位置，或输出无解。

解法：列举前几项，可以发现规律

## 拉格朗日插值
### CF622F The Sum of the k-th Powers
[题目链接](https://www.luogu.com.cn/problem/CF622F)

大意：求  $\sum_{i=1}^ni^k \bmod (10^9+7)$ 的值。

解法：
## 位运算
### 概念
- 基本概念：原码，反码，补码。**正数的三者相等**，负数的反码是原码的取反，补码是反码加 1。
- 基本操作：`lowbit`, `getbit`, `revbit`, `setbit`。
- 二进制枚举子集：`for(int t=x;t;t=x&(t-1))`。

### P3048 \[USACO12FEB\]Cow IDs S
[题目链接](https://www.luogu.com.cn/problem/P3048)

大意：给出汉明权重为 k $(1 \leq k \leq 10)$ 的第 n $(1\leq n\leq 10^7)$ 个二进制数。

解法：这道题的标准解法是排列组合，但是我们还是可以模拟，并用位运算优化。求相同汉明权重的后继的代码[[hamming|见此]]。注意，要特判 $k = 1,2,3$ 的情况才能过题。

??? note "参考代码"
    ```cpp
    --8<-- "code/P3048.cpp"
    ```

### CF1508B Almost Sorted
[题目链接](https://www.luogu.com.cn/problem/CF1508B)

大意：一个长度为 n 的“几乎有序”的排列 a 满足 $\forall i\in [1,n-1]\cap \mathbf{N}^* , a_{i+1}\geq a_{i}-1$，我们称其为“几乎有序的”。给定长度 n，k，求出第 k 个长度为 n 的“几乎有序”排列。

解法：“几乎有序”的排列肯定可以分为若干段，其中在每一段中，后项等于前项加 1 或者前项减 1，易得，这些排列的个数共有 $2^{n-1}$ 个（首项是上升还是下降不影响结果）。若将上升和下降的状态看作二进制数中的 0 和 1，由于连续的 1 代表把后面的大数提前，不管在数值还是对应排列的字典序上都大于中间有 0 的排列，容易证明，二进制状态的大小直接反映排列的顺序。因此可以用二进制状态反推排列（写法类似双指针）。注意 n 较大时的特判。

??? note "参考代码"
    ```cpp
    --8<-- "code/CF1508B.cpp"
    ```

### CF1491D Zookeeper and The Infinite Zoo
[题目链接](https://www.luogu.com.cn/problem/CF1491D)

大意：若 $x\&y=y$，则 x 可以变成 x+y，问 u 是否可以经过若干次操作变为 v。$(1\leq u,v\leq 2^{30})$

解法：$x\&y=y$ 可以等同为 y 是 x 的子集。在操作后，1 只会向高位移动，且数量一定不变或者减少。从低到高枚举每位即可。

??? note "参考代码"
    ```cpp
    --8<-- "code/CF1491D.cpp"
    ```

### CF627A XOR Equation
[题目链接](https://www.luogu.com.cn/problem/CF627A)

大意：两个整数 a,b 满足 $a+b=s$，$a\oplus b=t$，求满足条件的正整数 a,b 数目。

解法：异或的本质是不进位的加法，满足 $a\oplus b+2\times a\&b=a+b$，所以 $a\&b=\frac{1}{2}(s-t)$，二进制枚举每一位统计即可。

??? note "参考代码"
    ```cpp
    --8<-- "code/CF627A.cpp"
    ```

<!--## P4060 \[Code+#1\]可做题
[题目链接](https://www.luogu.com.cn/problem/P4060)

大意：给定一个长度为 n $(1\leq n\leq 10^9)$，有 m $(1\leq m\leq 10^5)$ 个元素已知的序列。求序列异或前缀和的前缀和的最小值。

解法：考虑位运算每一位互不影响，对每一位分别计算。-->