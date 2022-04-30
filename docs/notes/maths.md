# 数学知识
## 同余
### 扩展欧几里得法
扩展欧几里得算法（Extended Euclidean algorithm, EXGCD），常用于求 $ax+by=\gcd(a,b)$ 的一组可行解。[^note1]

[^note1]: [OI Wiki 中的有关描述](https://oi-wiki.org/math/number-theory/gcd/#_7)

推导如下：

$$
\begin{aligned}
\because\ &ax_1+by_1=gcd(a,b)\\
&bx_2+(a\bmod b)y_2=gcd(b,a\bmod b)\\
\therefore\ &ax_1+by_1=bx_2+(a\bmod b)y_2\\
\textsf{又}\because\ & a\bmod b=a-(\lfloor\frac{a}{b}\rfloor\times b)\\
\therefore\ &ax_1+by_1=bx_2+(a-(\lfloor\frac{a}{b}\rfloor\times b))y_2\\
\textsf{即}\ &ax_1+by_1=ay_2-b(x_2-\lfloor\frac{a}{b}\rfloor y_2)
\end{aligned}
$$

故回溯过程为：$x_1=y_2,y_1=x_2-\lfloor\frac{a}{b}\rfloor y_2$。

```cpp
long long exgcd(long long a, long long b, long long &x, long long &y)
{
	if(b == 0)
	{
		x = 1, y = 0;
		return a;
	}
	exgcd(b, a%b, y, x);
	y -= a / b * x;
}
```

### 乘法逆元

- 逆元的存在条件：a 与 p 互质。
- 单个数的逆元求法：快速幂法（要求是质数）和扩展欧几里得法（不要求是质数）。
- 逆元的线性递推：`inv[i]=(p-p/i)*inv[p%i]%p`。
- 阶乘逆元：
	1. 求出 1~n 的阶乘；
	2. 求出 n! 的逆元；
	3. 由 i! 的逆元逆推出 (i-1)! 的逆元。
	
	1~n 的阶乘逆元可以反推出 1~n 的逆元。前缀积逆元可以反推出任意 n 个数的逆元。

以上两种方法的时间复杂度都是 O(n)，方法 1 更为简单，但方法 2 的理解更为直观。

逆元的使用场景：$C_n^m=\dfrac{n!}{m!\cdot(n-m)!}$

线性求 1~n 的逆元：

```cpp
inv[1] = 1;
for(int i=2;i<=n;i++)
	inv[i] = 1LL*(p-p/i)*inv[p%i]%p;
```

求阶乘逆元和 1~n 的逆元：
```cpp
pmod[1] = 1;
for(int i=2;i<=n;i++)
	pmod[i] = 1LL * pmod[i-1] * i % p;
exgcd(pmod[n], p, x, y);
pinv[n] = (x%p+p)%p;
for(int i=n;i>=1;i--)
	pinv[i-1] = 1LL*i*pinv[i]%p;
inv[1] = 1;
for(int i=2;i<=n;i++)
	inv[i] = 1LL*pmod[i-1]*pinv[i]%p;
```

### Lucas 定理

$$
C_n^m=\prod_{i=0}^{k}{C_{n_i}^{m_i}}\bmod p
$$

在 OI 中常用的形式是

$$
C_n^m=C_{n/p}^{m/p}\cdot C_{n\ \bmod\ p}^{m\ \bmod\ p}\bmod p
$$

两公式是等价的，OI 中的公式可化为

$$
C_n^m=\left(\prod_{i=1}^{k}{C_{n_i}^{m_i}}\right)\cdot C_{n_0}^{m_0}\bmod p
$$

```cpp
long long Lucas(long long n, long long m, long long p)
{
    if(m==0) return 1;
    else return C(n%p, m%p,p) * Lucas(n/p,m/p,p) % p;
}
```

该算法适用于 p 不大的情况。

### 中国剩余定理
对于一个模线性方程组

$$
\begin{cases}x\equiv a_1\;\;(mod\;\;m_1)\\x\equiv a_2\;\;(mod\;\;m_2)\\ \cdots \cdots\\x\equiv a_k\;\;(mod\;\;m_k)\\\end{cases}
$$

中国剩余定理的思想就是对于每个模数，构造一个满足该条件，并是其它所有模数的倍数的数字。它们的和显然符合要求。
