# 代码模板

## 数据结构

### ST 表

定义：

```cpp
int s[MAXN][MAXLOG];
```

预处理（此处 `max` 函数可以是任何满足可重复贡献性的函数，如 `min` ，`gcd` ）：

```cpp
slim = (int)(log((double)n)/log(2.0));
for(int i=1;i<=n;i++)
	s[i][0] = a[i];
for(int j=1;j<=slim;j++)
	for(int i=1;i<=n;i++)
		if(i+(1<<(j-1))<=n)
			s[i][j] = max(s[i][j-1],s[i+(1<<(j-1))][j-1]);
```

查询（有关 `max` 函数同上）：

```cpp
int query(int l, int r)
{
	int qlim = (int)(log((double)(r-l+1))/log(2.0));
	return max(s[l][qlim], s[r-(1<<qlim)+1][qlim]);
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