# 区间DP总结


## 概念

区间DP，常用与处理区间类的一些算贡献的问题，比如石子合并这类问题就可以用区间DP解决，但是DP和贪心的区别是贪心是取眼前的最优策略，但是DP是考虑全局最优。

DP能解决的问题通常拥有以下几个条件：

**最优子结构**：每个大问题都一定可以拆成多个小问题去解决，不会陷入无限拆解。

**无后效性**：每个值求出来的方法不会影响到后面的值求出来。

~~否则就是你不会的神秘贪心算法~~

## 例题：

### 石子合并(弱化版)

链接：[~~题解~~](https://vjudge.net/contest/746288#problem/A)

#### 思路：

就是石子合并板子题。

**状态**： $dp[i][j]$ 表示合并从 $i$ 到 $j$ 的石子的最小代价。

**答案**： $dp[1][n]$

~~怎么没有转移？~~

让我们先来推一下转移：

​	首先我们手玩一下样例：

​	![](https://cdn.luogu.com.cn/upload/image_hosting/08obi22t.png)

不难发现，每次合并一个大区间都是先把大区间分成两个小区间，然后再把小区间合并的代价算出来，最后加上这个大区间所有数的和就好了所以我们可以推出如下转移方程：

$$
dp[l][r] = \min_{k = l}^{r - 1}{dp[l][k] + dp[k + 1][r] + s[r] - s[l - 1]}
$$

然后，就可以编代码了。

#### code:

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
#define pf push_front
#define mp make_pair
#define s second
#define f first
const int inf = (1 << 30) - 1;
const ll INF = 1LL << 60LL;
const int mod = 999971;

using namespace std;

const int N = 310;
int n;
ll dp1[N][N], a[N], s[N];

int main() {
    scanf("%d", &n);
    rep(i, 1, n) scanf("%lld", &a[i]);
    rep(i, 1, n) fill(dp1[i] + 1, dp1[i] + n + 1, inf);
    rep(i, 1, n) s[i] = s[i - 1] + a[i], dp1[i][i] = 0;
    rep(d, 1, n - 1) rep(l, 1, n - d) {
        int r = l + d;
        rep(k, l, r - 1) dp1[l][r] = min(dp1[l][r], dp1[l][k] + dp1[k + 1][r] + s[r] - s[l - 1]);
        // printf("%d %d %d %d\n", l, r, dp1[l][r], dp2[l][r]);
    }
    printf("%d\n", dp1[1][n]);
}
```

~~由于是在F题的代码上面改的，所以叫dp1~~

**时间复杂度：** $O(n^3)$

### 石子合并

[~~题解~~](https://vjudge.net/contest/746288#problem/F)

#### 思路：

这道题与上一题的区别在于上道题是一条链，而这一题是一个环，这时我们就需要破环为链，也就是把数组再复制一遍。

**为什么要复制一遍**：

假设原数组长度为 $n$，环形意味着第 $n$ 个石子可以和第 $1$ 个石子合并。
但普通区间 DP只能处理线性区间，无法直接处理“跨头尾”的区间。

**复制一遍的作用：**

- 把 $a[1...n]$ 复制到 $a[n+1...2n]$，形成长度为 $2n$ 的数组。
- 这样，原环上的任意连续 $n$ 个石子都可以在 $a[1...2n]$ 的区间中表示出来。

然后再和上一个例题一样的做法就好了。

**答案**：在所有的 $dp[i][i + n - 1]$ 中取 $\min$ 或 $\max$ 就好了

#### 代码：

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
#define pf push_front
#define mp make_pair
#define s second
#define f first
const int inf = (1 << 30) - 1;
const ll INF = 1LL << 60LL;
const int mod = 999971;

using namespace std;

const int N = 210;
int n;
ll dp1[N][N], dp2[N][N], a[N], s[N];

int main() {
    scanf("%d", &n);
    rep(i, 1, n) scanf("%lld", &a[i]), a[i + n] = a[i];
    n *= 2;
    rep(i, 1, n) fill(dp1[i] + 1, dp1[i] + n + 1, inf), fill(dp2[i] + 1, dp2[i] + n + 1, -inf);
    rep(i, 1, n) s[i] = s[i - 1] + a[i], dp1[i][i] = dp2[i][i] = 0;
    rep(d, 1, n - 1) rep(l, 1, n - d) rep(r, l + d - 1, n) {
        // int r = l + d;
        rep(k, l, r - 1) {
            dp1[l][r] = min(dp1[l][r], dp1[l][k] + dp1[k + 1][r] + s[r] - s[l - 1]);
            dp2[l][r] = max(dp2[l][r], dp2[l][k] + dp2[k + 1][r] + s[r] - s[l - 1]);
        }
        // printf("%d %d %d %d\n", l, r, dp1[l][r], dp2[l][r]);
    }
    ll ans1 = INF, ans2 = -INF;
    rep(i, 1, n / 2) ans1 = min(ans1, dp1[i][i + n / 2 - 1]), ans2 = max(ans2, dp2[i][i + n / 2 - 1]);
    printf("%d\n%d\n", ans1, ans2);
}
```

**时间复杂度：** $O(n^3)$

### Zuma

#### 思路：

这是我一开始的思路：

先把这个大区间分成两个小区间，在将两个小区间的操作数相加，如果这个大区间本来就是回文串，就将这个大区间的操作数设为一，否则设为两个小区间相加。

但是，这是错的，因为有可能一开始不是回文串，但是后面删去中间一部分后剩下的部分又是一个回文串。

正解：

**状态：** $dp[l][r]$ 表示消除 $l, r$ 区间的操作数。

**转移：**

$$
if (a[l] == a[r]) dp[l][r] = min(dp[l][r], dp[l + 1][r - 1]);
$$

$$
dp[l][r] = min(dp[l][r], \min_{k - l}^{r}{dp[l][k] + dp[k + 1][r]});
$$

**答案：** 即为 $dp[1][n]$

#### code:

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
#define pf push_front
#define mp make_pair
#define s second
#define f first
const int inf = (1 << 30) - 1;
const ll INF = 1LL << 60LL;
const int mod = 999971;

using namespace std;

const int N = 510;
int n, a[N], dp[N][N];

// bool check(int l, int r) {
//     for (int i = l, j = r; i < j; i++, j--) if (a[i] != a[j]) return false;
//     return true;
// }

int main() {
    scanf("%d", &n);
    rep(i, 1, n) fill(dp[i] + 1, dp[i] + n + 1, inf);
    rep(i, 1, n) scanf("%d", &a[i]), dp[i][i] = 1;
    rep(i, 1, n) dp[i + 1][i] = 1;
    /*
    有可能下面的(a[l] == a[r])本来就是最优的，后面更新的时候有可能前面都没它优，
    但是实际上这个区间的操作次数还是要加一的，所以对于每个i, dp[i + 1][i] = 1; 
    */
    rep(d, 1, n) rep(l, 1, n - d) {
        int r = l + d;
        if (a[l] == a[r]) dp[l][r] = min(dp[l][r], dp[l + 1][r - 1]);
        rep(k, l, r) dp[l][r] = min(dp[l][r], dp[l][k] + dp[k + 1][r]);//枚举到r是因为有可能上面的更新就是最优的
        // printf("%d %d %d\n", l, r, dp[l][r]);
    }
    printf("%d\n", dp[1][n]);
}
```




**时间复杂度：** $O(n^3)$


