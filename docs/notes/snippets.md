# 代码模板

## 数据结构

### ST 表

定义：

```cpp
int s[MAXN][MAXLOG];
```

预处理（此处 `max` 函数可以是任何满足可重复贡献性的函数，如 `min` ，`gcd` ）：

```cpp
slim = (int)(log((double)n) / log(2.0));
for (int i = 1; i <= n; i++)
    s[i][0] = a[i];
for (int j = 1; j <= slim; j++)
    for (int i = 1; i <= n; i++)
        if (i + (1 << (j - 1)) <= n)
            s[i][j] = max(s[i][j - 1], s[i + (1 << (j - 1))][j - 1]);
```
查询（有关 `max` 函数同上）：

```cpp
int query(int l, int r)
{
    int qlim = (int)(log((double)(r - l + 1)) / log(2.0));
    return max(s[l][qlim], s[r - (1 << qlim) + 1][qlim]);
}
```

### 树状数组

基本操作：
```cpp
int c[maxn];
void add(int x, int k)
{
    for (; x <= n; x += x & -x)
        c[x] += k;
}
int ask(int x)
{
    int k = 0;
    for (; x; x -= x & -x)
        k += c[x];
    return k;
}
```

倍增（以 `kth` 为例）：
```cpp
int kth(int x)
{
    int p = 0;
    for (int k = maxlog; k >= 0; k--)
    {
        p += (1 << k);
        if (p > n || c[p] >= x)
            p -= (1 << k);
        else
            x -= c[p];
    }
    return p + 1;
}
```

!!! note "解释"
	倍增的思想就是如果超出就回退，在上述示例的代码中，位置如果超出范围或者大小太大就执行回退。

### 线段树
以[洛谷模板](https://www.luogu.com.cn/problem/P3372)为例。

建树：

```cpp
long long a[maxn];
long long d[maxn*4], z[maxn*4];
void build(int id, int l, int r)
{
	if(l == r)
	{
		d[id] = a[l];
		return;
	}
	int m = (l+r)/2;
	build(id<<1, l, m);
	build(id<<1|1, m+1, r);
	d[id] = d[id<<1] + d[id<<1|1];
}
```

建树时只涉及 `pushup` 操作，而不涉及 `pushdown` 操作。

下面是其他操作的代码：

```cpp
void add(int id, int l, int r, int ql, int qr, long long k)
{
	if(ql <= l && r <= qr)
	{
		d[id] += 1LL * (r-l+1) * k;
		z[id] += k;
		return;
	}
	int m = (l+r)/2;
	if(z[id] && l != r)
	{
		d[id<<1] += z[id] * (m-l+1);
		d[id<<1|1] += z[id] * (r-m);
		z[id<<1] += z[id];
		z[id<<1|1] += z[id];
		z[id] = 0;
	}
	if(ql <= m)
		add(id<<1, l, m, ql, qr, k);
	if(m < qr)
		add(id<<1|1,m+1,r,ql,qr, k);
	d[id] = d[id<<1] + d[id<<1|1]; 
}
long long ask(int id, int l, int r, int ql, int qr)
{
	if(ql <= l && r <= qr)
		return d[id];
	int m = (l+r)/2;
	long long sum = 0;
	if(z[id] && l != r)
	{
		d[id<<1] += z[id] * (m-l+1);
		d[id<<1|1] += z[id] * (r-m);
		z[id<<1] += z[id];
		z[id<<1|1] += z[id];
		z[id] = 0;
	}
	if(ql <= m)
		sum += ask(id<<1, l, m, ql, qr);
	if(m < qr)
		sum += ask(id<<1|1,m+1,r,ql,qr);
	return sum;
}
```

`pushdown` 操作的性质是**操作前后，树的本质不变**。只要涉及从上往下的访问，都需要进行 `pushdown` 操作。而 `pushup` 操作只有在涉及修改的情况下需要进行。特别的是，单纯 `pushdown` 之后不需要 `pushup`。

#### 动态开点线段树
动态开点线段树不提前建树，而是在访问每个节点的时候判断当前节点是否建立，如果没有则新开这个节点。以 [P3369 【模板】普通平衡树](https://www.luogu.com.cn/problem/P3369) 为例。

```cpp
int d[maxn], cnt=1, rt=1;
int ls[maxn], rs[maxn];
void add(int &id, int l, int r, int x, int k)
{
	if(!id)
		id=++cnt;
	if(l==r)
	{
		d[id]+=k;
		return;
	}
	int m = (l+r)>>1;
	if(x<=m)
		add(ls[id],l,m,x,k);
	else
		add(rs[id],m+1,r,x,k);
	d[id] = d[ls[id]] + d[rs[id]];
}
int ask(int id, int l, int r, int ql, int qr)
{
	if(!id)
		return 0;
	if(ql <= l && r <= qr)
		return d[id];
	int m = (l+r)>>1, sum = 0;
	if(ql <= m)
		sum += ask(ls[id], l, m, ql, qr);
	if(m < qr)
		sum += ask(rs[id], m+1,r,ql, qr);
	return sum;
}
int kth(int id, int l, int r, int k)
{
	if(!id)
		return -1;
	if(l == r)
		return l;
	int m = (l+r)>>1;
	if(k <= d[ls[id]])
		return kth(ls[id], l, m, k);
	else
		return kth(rs[id], m+1,r,k-d[ls[id]]);
}
```

!!! warning "注意"
    动态开点线段树往往需要应对负数区间，注意在除以 `2` 的时候使用右移运算符，而不要使用除法运算符，因为我们需要的是“**向下**取整”，不是“**向零**取整”。C++ 的 `/` 运算符是**向零取整**的。

### Splay 树

目前我使用的 Splay 是指针版的。

```cpp
struct node
{
    node *ch[2];
    int val, cnt, siz;
    node(int v) : ch{nullptr, nullptr}, val(v), cnt(1), siz(1) {}
    void pushup() { siz = (ch[0] == nullptr ? 0 : ch[0]->siz) + cnt + (ch[1] == nullptr ? 0 : ch[1]->siz); }
    int find(int v) { return v == val ? -1 : v > val; }
    int chsiz(int x) { return ch[x] == nullptr ? 0 : ch[x]->siz; }
} *root = nullptr;
void rotate(node *&cur, bool type)
{
    node *rt = cur->ch[type];
    cur->ch[type] = rt->ch[!type], rt->ch[!type] = cur, cur = rt;
    cur->ch[!type]->pushup(), cur->pushup();
}
void splay(node *&cur, int val)
{
    int type1 = cur->find(val);
    if (type1 == -1 || cur->ch[type1] == nullptr)
        return;
    int type2 = cur->ch[type1]->find(val);
    if (type2 == -1 || cur->ch[type1]->ch[type2] == nullptr)
    {
        rotate(cur, type1);
        return;
    }
    splay(cur->ch[type1]->ch[type2], val);
    if (type1 == type2)
        rotate(cur, type1), rotate(cur, type2);
    else
        rotate(cur->ch[type1], type2), rotate(cur, type1);
}
```

## 图论

### 树的重心

```cpp
int root, siz[maxn], minch, tsiz;
void get_root(int u, int fa)
{
    siz[u] = 1;
    int maxch = 0;
    for(int i=head[u];i;i=nxt[i])
    {
        int v = to[i];
        if(v != fa && !blocked[v])
        {
            get_root(v, u);
            siz[u] += siz[v];
            maxch = max(maxch, siz[v]);
        }
    }
    maxch = max(maxch, tsiz-siz[u]);
    if(maxch < minch)
        minch = maxch, root = u;
}
```

### 链式前向星存图

```cpp
int head[MAXN], nxt[MAXM], to[MAXM], cnt;
void add(int u, int v)
{
	nxt[++cnt]=head[u];
	head[u]=cnt;
	to[cnt]=v;
}
```

从 1 开始，预处理为 0；从 0 开始，预处理为 -1。

### 图的连通性

缩点

```cpp
int dfn[MAXN], low[MAXN], dcnt;
int s[MAXN], ins[MAXN], scnt;
int belong[MAXN], bs[MAXN], bcnt;
void dfs(int u)
{
	dfn[u] = low[u] = ++dcnt;
	s[++scnt] = u;
	ins[u] = 1;
	for(int i=head[u];i;i=nxt[i])
	{
		int v=to[i];
		if(!dfn[v])
		{
			dfs(v, u);
			low[u] = min(low[u], low[v]);
		}
		if(ins[v])
			low[u] = min(low[u], dfn[v]);
	}
	if(dfn[u] == low[u])
	{
		bcnt++;
		while(s[scnt] != u)
			ins[s[scnt]] = 0, bs[bcnt]++, belong[s[scnt--]]=bcnt;
		ins[s[scnt]] = 0, bs[bcnt]++, belong[s[scnt--]]=bcnt;
	}
}
```

不建议写 `if(dfn[v] < dfn[u] && v != fa)`，因为只有当图**连通**时才正确。

### 重链剖分

树上启发式合并的预处理

```cpp
void predfs(int u, int fa)
{
    dep[u] = dep[fa] + 1;
    siz[u] = 1;
    for (int i = head[u]; i; i = nxt[i])
    {
        int v = to[i];
        if (v != fa)
        {
            predfs(v, u);
            siz[u] += siz[v];
            if (!heavy[u] || siz[heavy[u]] < siz[v])
                heavy[u] = v;
        }
    }
}
```
