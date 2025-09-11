# $tarjan$ 学习总结

~~太监~~ $tarjan$ 算法就是一种 $dfs$ 找割点以及个边或点双或边双等的算法，

## 知识点一：联通分量

联通分量也称极大联通子图，说通俗点就是连通块，但是极大连通子图和最大联通子图有不一样，有的时候，最大联通子图一定是极大连通子图，而极大连通子图不一定是最大联通子图。

我们画图举例：

![](https://cdn.luogu.com.cn/upload/image_hosting/58abzfsm.png)

这样不就可以很好的显示出它们之间的区别了。

## 知识点二：搜索树

搜索树就是用图的 $dfs$ 序建树，同样我们画图举例：

![](https://cdn.luogu.com.cn/upload/image_hosting/74o4nk6o.png)

这个图的搜索树是这样的：

![](https://cdn.luogu.com.cn/upload/image_hosting/94m2ntw7.png)

其中红色为返祖边，黑边为搜索树上的边，返祖边就是能够走回祖先的边。

## 知识点三：割点

如果从无向图中删去一个点之后，图里的连同块数量增加，删去的这个点就叫做割点。

如果我们依次考虑删除每个点，再数出连通分量的数量，复杂度就会来到$O(nm)$

我们一般使用$tarjan$算法来找割点：

### 流程：

$tarjan$ 算法其实就是在 $dfs$ 的过程中维护 $dfn$ 和 $low$ 两个信息，

其中 $low$ 是在不经过（搜索树中）父节点的情况下能到达的最小的时间戳。

如下图：

![](https://cdn.luogu.com.cn/upload/image_hosting/6kx53g6m.png)

他的搜索树如下：

![](https://cdn.luogu.com.cn/upload/image_hosting/de5az6ah.png)

 

如果对于一个点 $u$，它至少存在一个儿子 $v$ 不能回到 $u$ 之前的点，那么 $u$ 就是割点

因为删掉 $u$ 之后，$v$ 所在的子树没有返祖边可以连回来。

我们可以用 $low$ 和 $dfn$ 快速判断是否能连回来，$low[v] >= dfn[u]$ 就表示不能连回来，

**只在非根节点时成立**

**根节点需要单独考虑**

如果根节点在**搜索树(不是图！！！)**上有两个或更多子节点，根节点是割点。

否则如果只有一个子节点，则不是割点。

## 知识点四：割边

和割点的定义类似，如果从图中删去一个边之后，图里的连通分量数量增加，删去的这个边就叫做割边。 

如果没有重边，只需要把求割点的过程中的 $low[v] >= dfn[u]$  改为 $>$ 即可，

如果有重边，那么重边们一定不是割边。

## 例题一：割点（割顶）

tarjan模板题，思路上面讲了，直接放代码。

```cpp
#include <bits/stdc++.h>

using ll = long long;
using ull = unsigned long long;
using i128 = __int128;
using db = double;
#define rep(i, a, b) for (int i = a; i <= b; i++)
#define frep(i, b, a) for (int i = b; i >= a; i--)
#define all(x) (x).begin(), (x).end()
#define pb push_back
#define mp make_pair
#define s second
#define f first
const int inf = (1 << 30) - 1;
const ll INF = (1LL << 62LL) - 1;
const int mod = 998244353;

using namespace std;

const int N = 2e4 + 10;
int n, m, tot, dfn[N], low[N];
vector<int> ans, g[N];

void dfs(int u, int fa) {
    low[u] = dfn[u] = ++tot;
    bool ok = 0; 
    int cnt = 0;
    for (auto v : g[u]) {
        if (fa == v) continue;
        if (dfn[v]) low[u] = min(low[u], dfn[v]);
        else {
            cnt++;
            dfs(v, u);
            low[u] = min(low[u], low[v]);
            if (fa != u && !ok && low[v] >= dfn[u]) ans.pb(u), ok = true;
        }
    }
    if (!ok && fa == u && cnt > 1) ans.pb(u);
}

int main() {
    scanf("%d%d", &n, &m);
    rep(i, 1, m) {
        int u, v;
        scanf("%d%d", &u, &v);
        g[u].pb(v);
        g[v].pb(u);
    }
    rep(i, 1, n) if (!dfn[i]) dfs(i, i);
    sort(all(ans));
    printf("%d\n", ans.size());
    for (auto v : ans) printf("%d ", v);
}
```

~~赛时被卡原因：没看输出。~~

## 例题二：点双连通分量

点双连通分量，简称点双

删去一个点后连通性不变的连通分量就是点双连通分量

​	● 两个点双之间最多只有一个公共点，这个点是割点。

​	● 对于一个点双，它在搜索树中 $dfn$ 最小的点一定是割点或根节点

所以，我们用栈记录搜索过的点，在搜索到割点之后的段，把点双统计出来即可。

### 代码：

```cpp
#include <bits/stdc++.h>

using ll = long long;
using ull = unsigned long long;
using i128 = __int128;
using db = double;
#define rep(i, a, b) for (int i = a; i <= b; i++)
#define frep(i, b, a) for (int i = b; i >= a; i--)
#define all(x) (x).begin(), (x).end()
#define pb push_back
#define mp make_pair
#define s second
#define f first
const int inf = (1 << 30) - 1;
const ll INF = (1LL << 62LL) - 1;
const int mod = 998244353;

using namespace std;

const int N = 5e5 + 10;
int n, m, tot, dfn[N], low[N], deg[N];
vector<int> g[N];
stack<int> st;
vector<vector<int>> bdcc;

void dfs(int u, int fa) {
	low[u] = dfn[u] = ++tot;
	st.push(u);
	for (auto v : g[u]) {
		if (v == fa) continue;
		if (dfn[v]) low[u] = min(low[u], dfn[v]);
		else {
			dfs(v, u);
			low[u] = min(low[u], low[v]);
			if (low[v] >= dfn[u]) {
				vector<int> vv{u};
				while (vv.back() != v) {
					vv.pb(st.top());
					deg[st.top()]++;
					st.pop();
				}
				deg[u]++;
				bdcc.pb(vv);
			}			
		}
	}
}


int main() {
    scanf("%d%d", &n, &m);
    rep(i, 1, m) {
        int u, v;
        scanf("%d%d", &u, &v);
        g[u].pb(v);
        g[v].pb(u);
    }
    rep(i, 1, n) if (!dfn[i]) dfs(i, i);
    // sort(all(ans));
    // printf("%d\n", ans.size());
    // for (auto v : ans) printf("%d ", v);
    rep(i, 1, n) if (deg[i] == 0) {
        vector<int> v;
        v.pb(i);
        bdcc.pb(v);
    }
    printf("%d\n", bdcc.size());
    for (auto v : bdcc) {
        printf("%d ", v.size());
        for (auto y : v) printf("%d ", y);
        puts("");
    }
}
```



~~赛是被卡原因：没判单点~~