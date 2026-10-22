## 相关链接：
- 题目链接 : [余镜 - 题目 - QOJ.ac](https://qoj.ac/contest/4582/problem/22922)

## 评价 ：暴力 + 数学统计


## 题目 ：

![](../../../../截图文件/Pasted%20image%2020261022104954.png)

## 题目大意 :

给定一个长度为 $n$ 的序列 $a_1, a_2, \dots, a_n$ 和一个模数上限 $M$。 对于所有 $1 \le l < r \le n$ 和 $2 \le m \le M$，定义 $f(l, r, m) = 1$ 当且仅当子序列 $[a_l \pmod m, a_{l+1} \pmod m, \dots, a_r \pmod m]$ 是回文序列；否则 $f(l, r, m) = 0$。 求所有 $1 \le l < r \le n$ 与 $2 \le m \le M$ 下 $f(l, r, m)$ 的总和，即：

$$\sum_{1 \le l < r \le n} \sum_{2 \le m \le M} f(l, r, m)$$

## 核心思路 + 数学转化 ：

1. **回文判定条件**： 子序列 $[a_l \pmod m, \dots, a_r \pmod m]$ 是回文序列，等价于对于所有 $l \le i \le \frac{l+r}{2}$，均满足：
    
    $$a_i \equiv a_{r+l-i} \pmod m$$
    
    这也就是说，模数 $m$ 必须能够整除所有对称位置差值的绝对值 $\vert{}a_i - a_{r+l-i}\vert{}$。
    
2. **区间 $\gcd$ 转化**： 令 $g(l, r)$ 表示区间 $[l, r]$ 中所有对称位置差值的最大公约数：
    
    $$g(l, r) = \gcd_{l \le i \le \lfloor \frac{l+r}{2} \rfloor} \vert{}a_i - a_{r+l-i}\vert{}$$
    
    - 如果 $g(l, r) > 0$，那么满足条件的 $m$ 就是 $g(l, r)$ 在 $[2, M]$ 范围内的**因数个数**。
        
    - 如果 $g(l, r) = 0$（即所有对称位置的差值全为 $0$，也就是 $a_i = a_{r+l-i}$ 恒成立），此时任何 $m \ge 2$ 都能满足条件，满足条件的 $m$ 共有 $M - 1$ 个。


## 状态转移与动态规划设计

我们可以通过区间长度从小到大进行区间 DP：

- 定义 `vis[l][r]` 表示区间 $[l, r]$ 的所有对称位置差值的 $\gcd$（若全为 0 则记为 0）。
    
- **状态转移方程**：
    
    $$g(l, r) = \gcd(g(l+1, r-1), \vert{}a_l - a_r\vert{})$$
    
    结合左右端点 $a_l$、 $a_r$ 与内部区间已有的 $\gcd$（即 `vis[l+1][r-1]`），我们可以高效地递推计算出当前区间的 $\gcd$。
    

## 预处理与复杂度分析

1. **因数预处理**： 使用类似埃氏筛（调和级数）的方法，预处理出每个数在 $[2, M]$ 范围内的因数个数，时间复杂度为 $O(M \log M)$。
    
2. **区间 DP**： 区间总共有 $\frac{n(n-1)}{2} = O(n^2)$ 个。每次转移仅需进行一次 $\gcd$ 运算，时间复杂度为 $O(\log(\max a))$。
    
3. **总复杂度**：
    
    - **时间复杂度**：$O(n^2 \log (\max a) + M \log M)$。在 $n \le 8000, M \le 10^6$ 的数据范围下，总计算量约为 $3.2 \times 10^7$ 次基本操作，完全可以在 2.0 秒的时间限制内轻松通过。
        
    - **空间复杂度**：$O(n^2 + M)$（主要用于存储 `vis` 二维数组和因数统计数组）。



## 代码展示 ：
```c++
#include <bits/stdc++.h>
using namespace std;
#define int long long
#define endl '\n'
#define ctz __builtin_ctzll         // 二进制表示中末尾连续0的个数
#define clz __builtin_clzll         // 二进驻表示中先导0的个数
#define count1 __builtin_popcountll // 二进制表示中1的个数
const int mod = 1e9 + 7;
const int MOD = 998244353;
const int N = 2e5 + 7;

void solve() { 
    int n, m; 
    cin >> n >> m;
    vector<int> a(n + 1);
    for(int i = 1; i <= n; i++) cin >> a[i];
    
    vector<vector<int>> vis(n + 1, vector<int>(n + 1));
    
    //类似于埃式筛 ：去统计【2，m】范围中，i有多少个因子个数
    vector<int> p(m + 1, 0);
    p[0] = m - 1; 
    for(int i = 2; i <= m; i++) {
        for(int j = i; j <= m; j += i) {
            p[j]++;
        }
    }
    
    //暴力
    int ans = 0;
    for(int len = 2; len <= n; len++) {
        for(int l = 1; l <= n - len + 1; l++) {
            int r = l + len - 1;
            int before = vis[l + 1][r - 1];
            
            if(!before) {
                if(a[l] == a[r]) {
                    ans += m - 1;
                    vis[l][r] = 0;
                }
                else {
                    int x1 = abs(a[l] - a[r]); 
                    vis[l][r] = x1;
                    ans += p[x1];
                }
            }
            else {
                if(a[l] == a[r]) {
                    ans += p[before];
                    vis[l][r] = before;
                }
                else {
                    vis[l][r] = gcd(abs(a[l] - a[r]), before);
                    ans += p[vis[l][r]];
                }
            }
        }
    }
    
    cout << ans << endl;
    return;
}

signed main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);
    cout.tie(0);
    int T = 1;
    //cin >> T;
    while (T--) {
        solve();
    }
    return 0;
}
```

