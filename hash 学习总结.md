# hash 学习总结

**注：本章所有代码都涉及到STL，空间不好计算，所以不予空间计算。**

## 前置知识：集合、映射

### 集合：

集合，就是把一些元素汇集到一起的一个整体。

集合中的元素具有确定性、无序性、互异性。

比如你要表示 $[1, 10000]$ 的实数，可以这样表示：$S = \{ x \in \mathbb{R} \mid 1 \leq x \leq 10000 \}$ 。

而集合什么元素也没有记作 $ \varnothing $ 。

### 映射：

给定两个集合 $S, T$，如果有一种法则 $\sigma$ 能将 $S$ 中的元素一一对应到 $T$ 中的元素，则称 $\sigma$ 为 $S$ 到 $T$ 的一个映射。

记作 $\sigma：S \to T$ 。

这里 $\sigma$ 为小写的 $\Sigma$

## 主要知识 ： hash

给你一个只包含小写字母的字符串 $s$ 和 $t$ (保证 $\left | s \right | > \left | t \right |$)，让你求出 $t$ 在 $s$ 中出现了多少次。

数据范围：$\left |s \right | \leq 5 * 10^5$，$\left | t \right | \leq 5 * 10^5$。

首先暴力就是枚举 $s$ 的每一个位置，然后暴力判是否相同，但是时间复杂度 $O(nm)$。

然后就可以用字符串hash优化。

但是不能保证完全正确，原因后面会讲。

首先直接的字符串比较瓶颈在于比较，而比较是不是可以用一些其他的东西代替呢？

我们想到是不是可以把字符串转换成数字或者更好比较的东西，然后再去比较呢？

字符串 hash 就诞生了。

字符串 hash 可以看为从字符集合到数字集合的一个映射。

如果两个字符串的哈希值是一样的，那么我们也认为这两个字符串是一样的，当然有时会被卡。



**为什么模数要选好**

老师上课给我们讲过这么一个例子：一个字符串就当它只含小写字母，那么每一位都有26种可能，如果字符串长度为 $5 * 10^5$

那么就会有 $26 ^ {5 * 10^5}$ 种可能。

就算你的模数是 $2 ^ {62}$ 级别的，根据鸽巢原理~~（好像是人都能看出来）~~一定会有两个不相同的字符串但是哈希值一样，此时便是哈希冲突。



**减小误判的方法**

第一个是

不过也有方法解决哈希冲突，就是多哈希，这样可以大幅度的减少你判错的可能。

假设当前单哈希的冲突概率为 $x$，那么如果你使用了 $y$ 哈希，那么你的冲突概率将会从 $x$ 变为 $x^y$

然后就是哈希函数设计。

你需要一个base，还需要一个

哈希函数的话一般是转 $base$ 进制然后取模。



代码：

```cpp
template<ull base, ull mod> //为了多哈希方便
struct hash {
    int len;
    std::vector<ull> ha, pw;//ha为哈希值数组，pw为base的次方数组
    void init(const std::string &s) {//初始求出字符串的哈希值
        len = s.size();
        ha.resize(len + 1), pw.resize(len + 1);
        pw[0] = 1;
        rep(i, 1, len) {
            ha[i] = add(mul(ha[i - 1], base), s[i]);
            pw[i] = mul(pw[i - 1], base);
        }
    }
    ull add(ull a, ull b) {//重载加
        a += b;
        if (a >= mod) a -= mod;//这里需要保证a, b均小于mod
        return a;
    }
    ull mul(ull a, ull b) {return i128(a) * b % mod;}//重载乘
    ull query(int l, int r) {return add(ha[r], mod - mul(ha[l - 1], pw[r - l + 1]));}//访问从l到r的哈希值
};
```

我这里是封装好的。

**注意：这里base是一定要大于字符集的，否则就会要很频繁的哈希冲突。**

**mod一般取质数且很大，这样才能尽量避免冲突**

## 例题

### 例题一 ：[字符串哈希](https://vjudge.net/contest/766880#problem/B)

**题意**

给你n个字符串，求出有多少个不相同的字符串。

**思路**

哈希模板题，思路前面已经讲过，其实就是将字符串的哈希值存下再去重，最后输出长度。

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
const int inf = 1e9;
const ll INF = 1LL << 60LL;
const ull mod = 998244353;
const ull base = 911;

const int N = 1e4 + 10;
template<ull base, ull mod> 
struct hash {
    int len;
    std::vector<ull> ha, pw;
    void init(const std::string &s) {
        len = s.size();
        ha.resize(len + 1), pw.resize(len + 1);
        pw[0] = 1;
        rep(i, 1, len) {
            ha[i] = add(mul(ha[i - 1], base), s[i]);
            pw[i] = mul(pw[i - 1], base);
        }
    }
    ull add(ull a, ull b) {
        a += b;
        if (a >= mod) a -= mod;
        return a;
    }
    ull mul(ull a, ull b) {return i128(a) * b % mod;}
    ull query(int l, int r) {return add(ha[r], mod - mul(ha[l - 1], pw[r - l + 1]));}
};
int n, m;
std::string s, t;
std::vector<ull> b;

int main() {
    std::ios::sync_with_stdio(false);
    std::cin.tie(nullptr), std::cout.tie(nullptr);
    std::cin >> n;
    rep(i, 1, n) {
        std::string s;
        std::cin >> s;
        s = ' ' + s;
        hash<base, mod> a;
        a.init(s);
        b.pb(a.query(1, s.size() - 1));
    }
    sort(all(b));
    b.erase(unique(all(b)), b.end());//离散化
    std::cout << b.size() << '\n';
}
```

**时间复杂度：** $O(NM)$

### 例题二：[匹配统计](https://vjudge.net/contest/766880#problem/G)

**题意**

给你两个字符串，让你求出第一个字符串的后缀与第二个字符串的匹配长度一样的有几个。

**思路**

他要我们求匹配长度，首先我们可以想到怎么去看匹配的长度，先从暴力出发。

暴力就是先枚举位置，然后枚举匹配长度，再暴力判是否匹配，但是这样时间复杂度显然很高，为 $O(nm^2)$ 。

开始优化，首先判相不相同是可以用哈希判断的，时间复杂度来到了 $O(nm)$ 。

此时瓶颈在于枚举匹配长度，如果长度越小是不是就越容易匹配，因为长度为一只有26种字符串，而长度为二就有 $26^2$种了。

后面更是指数级增长。于是就发现有单调性，可以二分匹配长度，此时时间复杂度来到了 $O(n\log{m})$

**代码**

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
const int inf = 1e9;
const ll INF = 1LL << 60LL;
const ull mod = (1ull << 61) - 1;
const ull base = 911;

const int N = 200010;
template<ull base, ull mod> 
struct hash {
    int len;
    std::vector<ull> ha, pw;
    void init(const std::string &s) {
        len = s.size();
        ha.resize(len + 1), pw.resize(len + 1);
        pw[0] = 1;
        rep(i, 1, len) {
            ha[i] = add(mul(ha[i - 1], base), s[i]);
            pw[i] = mul(pw[i - 1], base);
        }
    }
    ull add(ull a, ull b) {
        a += b;
        if (a >= mod) a -= mod;
        return a;
    }
    ull mul(ull a, ull b) {return i128(a) * b % mod;}
    ull query(int l, int r) {return add(ha[r], mod - mul(ha[l - 1], pw[r - l + 1]));}
};
int n, m, q, ans[N];
std::string s, t;

int main() {
    std::ios::sync_with_stdio(false);
    std::cin.tie(nullptr), std::cout.tie(nullptr);
    std::cin >> n >> m >> q;
    std::cin >> s >> t;
	s = ' ' + s, t = ' ' + t;
    hash<base, mod> a, b;
    a.init(s), b.init(t);
	rep(i, 1, n) {
		int l = 0, r = m + 1;
		while (l + 1 < r) {
			int mid = l + r >> 1;
            // if (i + mid - 1 > n) continue;
			if (a.query(i, i + mid - 1) == b.query(1, mid)) l = mid;
			else r = mid;
		}
		ans[l]++;
        // std::cout << i << '\n';
	}
	while (q--) {
		int x;
        std::cin >> x;
        std::cout << ans[x] << '\n';
	}
}
```


字符串哈希是一种很好用的算法，后面应该还会有更难的。
