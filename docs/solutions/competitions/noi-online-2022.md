# NOI Online 2022

### T1 丹钓战

大意：有 n $(1\leq n\leq 10^5)$ 个二元组 $(a_i,b_i)$。现在给出 q $(1\leq q\leq 10^5)$ 个询问，将从 l 到 r 的所有二元组顺次插入单调栈中，满足 b 值由栈底到栈顶下降，相邻两个元素的 a 值不相等。求有多少二元组在栈中单独出现过。

赛时思路：考虑模拟，对于每一个节点记录不冲突的前驱 $fa$。求多少二元组在栈中出现过，就等于求在 l 到 r 的范围内，有多少数的大小小于 l。具体用线段树或者树状数组维护即可。时间复杂度 $O(n \log{n})$（考虑 n，q 在同一级别）。

??? "赛时代码"
    ```cpp linenums="1"
    #include <iostream>
    #include <algorithm>
    #include <stack>
    using namespace std;
    const int maxn = 5e5 + 7;
    int n, m;
    int a[maxn], b[maxn];
    int s[maxn], scnt;
    pair<int, int> pre[maxn];
    int c[maxn];
    pair<pair<int, int>, int> q[maxn];
    int ans[maxn];
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
    int main()
    {
        freopen("stack.in", "r", stdin);
        freopen("stack.out", "w", stdout);
        scanf("%d%d", &n, &m);
        for (int i = 1; i <= n; i++)
            scanf("%d", &a[i]);
        for (int i = 1; i <= n; i++)
            scanf("%d", &b[i]);
        s[++scnt] = 0, b[0] = 1e7;
        for (int i = 1; i <= n; i++)
        {
            while (a[s[scnt]] == a[i] || b[s[scnt]] <= b[i])
                scnt--;
            pre[i] = make_pair(s[scnt], i);
            s[++scnt] = i;
        }
        sort(pre + 1, pre + n + 1);
        for (int i = 1; i <= m; i++)
        {
            scanf("%d%d", &q[i].first.first, &q[i].first.second);
            q[i].second = i;
        }
        sort(q + 1, q + m + 1);
        int p = 1;
        for (int i = 1; i <= m; i++)
        {
            while (p <= n && pre[p].first < q[i].first.first)
                add(pre[p].second, 1), p++;
            ans[q[i].second] = ask(q[i].first.second) - ask(q[i].first.first - 1);
        }
        for (int i = 1; i <= m; i++)
        {
            printf("%d\n", ans[i]);
        }
    }
    ```

### T2 讨论

大意：给定 n $(1\leq n\leq 10^6)$ 个集合，求是否存在交叉关系。

赛时思路：使用并查集，按集合的大小从小到大进行添加，并维护每个连通块的大小。可以证明，每个集合的子集总是在它之前被添加。如果某一次添加发现连通块的大小大于该点的度数，则可以判定存在交叉关系。时间复杂度 $O(n\operatorname{\alpha}(n))$

??? "赛时代码"
    ```cpp linenums="1"
    #include <iostream>
    #include <algorithm>
    #include <cstring>
    using namespace std;
    const int maxn = 2e6 + 7, maxm = 4e6 + 7;
    int head[maxn], nxt[maxm], to[maxm], ecnt;
    void add(int u, int v)
    {
        nxt[++ecnt] = head[u];
        head[u] = ecnt;
        to[ecnt] = v;
    }
    int t, n;
    int k[maxn];
    int fa[maxn], siz[maxn];
    int get_fa(int x)
    {
        if (x == fa[x])
            return x;
        return fa[x] = get_fa(fa[x]);
    }
    void merge(int a, int b)
    {
        int f_a = get_fa(a), f_b = get_fa(b);
        if (f_a != f_b)
        {
            siz[f_a] += siz[f_b];
            fa[f_b] = f_a;
        }
    }
    int ord[maxn];
    int mark[maxn];
    bool comp(int a, int b)
    {
        return k[a] < k[b];
    }
    int main()
    {
        freopen("discuss.in", "r", stdin);
        freopen("discuss.out", "w", stdout);
        int _t;
        scanf("%d", &t);
        while (t--)
        {
            for (int i = 1; i <= n; i++)
                head[i] = 0;
            scanf("%d", &n);
            bool flag = false;
            int ansa, ansb;
            for (int i = 1; i <= n; i++)
            {
                scanf("%d", &k[i]);
                for (int j = 1; j <= k[i]; j++)
                {
                    scanf("%d", &_t);
                    add(i, n + _t);
                }
            }
            for (int i = 1; i <= n * 2 + 2; i++)
            {
                fa[i] = i;
                siz[i] = 1;
            }
            for (int i = 1; i <= n; i++)
            {
                ord[i] = i;
                siz[i] = 0;
            }
            sort(ord + 1, ord + n + 1, comp);
            for (int i = 1; i <= n; i++)
            {
                int c = ord[i];
                for (int j = head[c]; j; j = nxt[j])
                    merge(c, to[j]);
                if (siz[get_fa(c)] > k[c])
                {
                    flag = true;
                    ansa = c;
                    for (int j = head[c]; j; j = nxt[j])
                        mark[to[j]] = (t + 1) << 1;
                    for (int j = n + 1; j <= n * 2; j++)
                        if (get_fa(j) == get_fa(c) && mark[j] != ((t + 1) << 1))
                            mark[j] = ((t + 1) << 1) | 1;
                    bool flaga = 0, flagb = 0;
                    for (int j = 1; j <= n; j++)
                    {
                        flaga = 0, flagb = 0;
                        if (get_fa(c) == get_fa(j))
                            for (int l = head[j]; l; l = nxt[l])
                                if (mark[to[l]] == ((t + 1) << 1))
                                    flaga = 1;
                                else if (mark[to[l]] == (((t + 1) << 1) | 1))
                                    flagb = 1;
                        if (flaga && flagb)
                        {
                            ansb = j;
                            break;
                        }
                    }
                    break;
                }
            }
            if (flag == false)
            {
                printf("NO\n");
            }
            else
            {
                printf("YES\n");
                printf("%d %d\n", ansa, ansb);
            }
        }
    }
    ```

### T3 如何正确地排序

大意：给定一个 $m\times n$ 的数组 $a_{i,j}$，其中 $m\leq4,n\leq 2\times 10^5$，求 $\sum_{i=1}^{n}{\sum_{j-1}^{n}{\min_{k=1}^{m}{(a_{k,i}+a_{k,j})}+\max_{k=1}^{m}{(a_{k,i}+a_{k,j})}}}$ 的值。

赛时思路：暴力得部分分。

??? "赛时代码"
    ```cpp linenums="1"
    #include <iostream>
    #include <algorithm>
    using namespace std;
    const int maxm = 5;
    const int maxn = 2e5 + 7;
    int m, n;
    int a[maxm][maxn];
    long long ans;
    int main()
    {
        freopen("sort.in", "r", stdin);
        freopen("sort.out", "w", stdout);
        scanf("%d%d", &m, &n);
        for (int i = 1; i <= m; i++)
            for (int j = 1; j <= n; j++)
                scanf("%d", &a[i][j]);
        if (m == 2)
        {
            for (int i = 1; i <= n; i++)
                ans += a[1][i] + a[2][i];
            ans *= 2LL * n;
            printf("%lld\n", ans);
        }
        else
        {
            int c1, c2;
            for (int i = 1; i <= n; i++)
                for (int j = 1; j <= n; j++)
                {
                    c1 = 0x7fffffff, c2 = 0;
                    for (int k = 1; k <= m; k++)
                        c1 = min(c1, a[k][i] + a[k][j]), c2 = max(c2, a[k][i] + a[k][j]);
                    ans += c1 + c2;
                }
            printf("%lld\n", ans);
        }
    }
    ```