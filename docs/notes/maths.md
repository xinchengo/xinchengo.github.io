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
	long long u = exgcd(b, a%b, y, x);
	y -= a / b * x;
	return u;
}
```

注意 `(x%mod+mod)%mod` 才是最终得到的逆元。

扩展欧几里得算法可以帮助我们解决**线性同余方程**的问题，因为 $ax+by=\gcd(a,b)$ 与 $ax\equiv gcd(a,b) \pmod{b}$ 在本质上是等价的。

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
pmod[0] = 1;
for(int i=1;i<=n;i++)
	pmod[i] = 1LL * pmod[i-1] * i % p;
exgcd(pmod[n], p, x, y);
pinv[n] = (x%p+p)%p;
for(int i=n;i>=1;i--)
	pinv[i-1] = 1LL*i*pinv[i]%p;
inv[1] = 1;
for(int i=2;i<=n;i++)
	inv[i] = 1LL*pmod[i-1]*pinv[i]%p;
```

!!! warning "注意"
	预处理阶乘和阶乘逆元时，一定要设 `pinv[0]=1` 和 `pmod[0]=1`，不然求组合数时会出错。

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
 \begin{cases} x &\equiv a_1 \pmod {n_1} \\ x &\equiv a_2 \pmod {n_2} \\ &\vdots \\ x &\equiv a_k \pmod {n_k} \\ \end{cases} 
$$

中国剩余定理的思想就是对于每个模数，构造一个满足该条件，并是其它所有模数的倍数的数字。它们的和显然符合要求。

具体地，设所有模数的积为 $n$，则方程组的解为：$x=\sum_{i=1}^{k}a_im_im_i^{-1}$，其中 $m_i =\frac{n}{n_i}$，$m_i^{-1}$ 是 $\bmod{n}$ 意义下的。$m_im_i^{-1}\equiv1\pmod{n_i}$，所以原式显然符合条件。

代码如下：

```cpp
prod = 1;
for (int i = 1; i <= n; i++)
	prod *= a[i];
for (int i = 1; i <= n; i++)
{
	long long m = prod / a[i];
	exgcd(m % a[i], a[i], x, y);
	x = (x % a[i] + a[i]) % a[i];
	ans = ((b[i] * m % prod) * x % prod + ans) % prod;
}
```

#### 扩展中国剩余定理
我们可以采取通过合并两个方程，合并整个方程组的思想。

$$
\begin{cases}a \equiv r_1 \pmod {m_1} \\ a \equiv r_2 \pmod {m_2}\end{cases}
$$

我们的目标是构造一个 $x$，满足 $a\equiv x\pmod{\operatorname{lcm}(m1,m2)}$，构造方式如下：

![excrt-formula.jpg](/_static/images/excrt-formula.jpg)

```cpp
ra = a[1], rb = b[1];
for(int i=2;i<=n;i++)
{
	long long d = exgcd(ra, a[i], x, y), l = ra / d * a[i];
	assert((b[i]-rb) % d == 0);
	x = regulate(x, l);
	rb = regulate(rb + mul(mul((b[i]-rb)/d,x,l),ra,l), l);
	ra = l;
}
```
## 多项式
### 拉格朗日插值
拉格朗日插值的思想是构造 $n$ 个通过其它点的零点和该点的函数，这些函数的和显然符合条件。

$$
f(k)=\sum_{i=1}^{n} y_i\prod_{j\neq i }\frac{k-x_j}{x_i-x_j}
$$

```cpp
// 此处的 regulate(x) 函数等价于 (x%p+p)%p
for(int i=1;i<=n;i++)
{
	int prod = y[i], inv, inv2;
	for(int j=1;j<=n;j++)
	{
		if(i != j)
		{
			prod = 1LL * prod * regulate(k - x[j]) % p;
			exgcd(regulate(x[i] - x[j]), p, inv, inv2);
			prod = 1LL * prod * regulate(inv) % p;
		}
	}
	ans = (ans + prod) % p;
}
```

差分数列的性质：有 $n$ 次通项公式的数列的一阶差分数列有 $n-1$ 次通项公式。