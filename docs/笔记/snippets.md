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
