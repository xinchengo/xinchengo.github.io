# Splay 树
Splay 树是一种优化的二叉查找树，其保持时间复杂度的核心是把每个访问的节点**上旋至根**。

同许多平衡树一样，Splay 树保持平衡也采用旋转的方式，一般来说，为了简化程序的编写，我们可以不记录节点的父节点，而把两类旋转编入一个函数中。
## 旋转
旋转的思想是上提一个节点并下放一个节点，同时要维护二叉查找树的性质。

![[Pasted image 20220104214206.png]]

为了方便，我将要上提的节点称为该节点，以它为标准称呼其父节点与祖节点。

具体来说，旋转可以分为三步：
1. 更新父节点
2. 让父节点成为该节点子树
3. 同时也要更新祖节点中子节点信息

其中第三步可以用引用调用的方式实现，在修改引用调用的同时，也修改了祖节点中子节点的信息。

如果我们每个节点的实现代码如下：

```cpp
struct node
{
	node *ch[2];
	int val, cnt, siz;
	node(int v):ch{nullptr, nullptr}, val(v), cnt(1), siz(1){}
	void pushup()
	{
		siz = (ch[0] == nullptr ? 0 : ch[0]->siz)
			+ cnt
			+ (ch[1] == nullptr ? 0 : ch[1]->siz)
	}
	int find(int v) // 权值为v的节点在该节点，左子树，还是右子树 
	{
		return v == val ? -1 : v > val;
	}
	int chsiz(int x) // 返回子树的大小
	{
		return ch[x] == nullptr ? 0 : ch[x]->siz;
	}
}
```

那么我们旋转的代码为

```cpp
void rotate(node*& cur, bool type)
{
	node *rt = cur->ch[type];
	cur->ch[type] = rt->ch[!type], rt->ch[!type] = cur, cur = rt;
	cur->ch[!type]->pushup(), cur->pushup();
}
```

注意两个 `pushup` 的顺序不要写反，千万**不要写反**。
## 提根
如我
