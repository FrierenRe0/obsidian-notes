## 相关链接 ：

- 题目链接 ：[Problem - D - Codeforces](https://codeforces.com/contest/2285/problem/D)

## 题目 ：

You are given positive integers $n$ and $m$.

A partition is an array $p$ of $m$ positive integers such that $p_1 + p_2 + \ldots + p_m = n.$

For any array $b$ of length $n$ and a partition $p$, define $f(b, p)$ as an array $c$ of length $m$ where:

-   $c_1$ is the sum of the first $p_1$ elements of $b$,
-   $c_2$ is the sum of the next $p_2$ elements of $b$,
    
    ...
    
-   $c_m$ is the sum of the last $p_m$ elements of $b$.

For an array $d$ of length $n$, let $e$ be the array obtained by reversing $d$. A partition $p$ is a special partition of $d$ if $f(d, p) = f(e, p)$.

You are given an array $a$ of length $n$. You must answer $q$ **independent** queries. For each query, you are given two integers $i$ and $v$. Set $a_i := a_i + v$, and then output the number of special partitions of $a$. Since the answer may be large, output it modulo $10^9 + 7$.

Note that the queries are **independent**, meaning that after each query, $a$ reverts to its original value.

**Input**
Each test contains multiple test cases. The first line contains the number of test cases $t$ ($1 \le t \le 10^4$). The description of the test cases follows.

The first line of each test case contains three integers $n$, $m$, and $q$ ($1 \le m \le n \le 2 \cdot 10^5, 1 \le q \le 2 \cdot 10^5$) – the length of $a$, the length of the partition, and the number of queries respectively.

The second line of each test case contains $n$ integers – $a_1, a_2, \ldots, a_n$ ($|a_i| \le 10^9$).

The next $q$ lines of each test case contain two integers $i$ and $v$ ($1 \le i \le n, |v| \le 10^9$).

It is guaranteed that the sum of $n$ and the sum of $q$ over all test cases do not exceed $2 \cdot 10^5$.



## 思路 ：
### 题目模型转化

题目的核心要求是将数组分为 $m$ 段，满足正向划分和反向（逆序）划分的对应段和相等。 仔细思考这个条件，**特殊划分的本质是“前缀和与后缀和的匹配”**。 要将数组分为 $m$ 段，实际上等价于在数组中插入 $m-1$ 个“分割点”。对于任意一个合法的分割点 $x$（表示前缀包含 $a[0] \dots a[x]$），必须满足：

$$\text{前缀和}(0 \dots x) = \text{后缀和}(n-1-x \dots n-1)$$

我们定义一个差值变量：

$$\Delta_x = \text{后缀和}(n-1-x \dots n-1) - \text{前缀和}(0 \dots x)$$

那么，**一个合法的分割点 $x$ 必须满足 $\Delta_x = 0$。** 如果有 $ans$ 个合法的分割点，我们需要从中选出 $m-1$ 个作为最终的划分，方案数即为组合数 $C(ans, m-1)$。

### 2. 单点修改

题目要求处理独立查询：将 $a[i]$ 增加 $v$，求方案数。 如果不加优化，每次修改重新计算 $\Delta_x$ 显然会超时。但我们观察修改 $a[i]$ 对 $\Delta_x$ 的影响，会发现变化是有规律且分段的。

设修改的位置为 $i$，我们令 $l = \min(i, n - 1 - i)$，$r = \max(i, n - 1 - i)$。 随着分割点 $x$ 的增长（$x \in [0, n-2]$），前缀和后缀不断向中间延伸。修改 $a[i]$ 会将所有的分割点严格划分为三个区间：

- **区间一：$x \in [0, l - 1]$（未触及区）** 此时前缀和后缀都很短，都没有包含修改点 $a[i]$。**影响**：$\Delta_x$ 不变。我们需要原本 $\Delta_x = 0$ 的点。
    
- **区间二：$x \in [l, r - 1]$（单侧包含区 / 失衡区）** 此时前缀和后缀中，**恰好有一个**包含了修改点 $a[i]$。
    
    - 若修改在左半边（$i \le n - 1 - i$）：只有前缀增加了 $v$，导致差值 $\Delta_x$ 减少了 $v$。为了修改后 $\Delta_x = 0$，**原数组中必须有 $\Delta_x = v$**。
        
    - 若修改在右半边（$i > n - 1 - i$）：只有后缀增加了 $v$，导致差值 $\Delta_x$ 增加了 $v$。为了修改后 $\Delta_x = 0$，**原数组中必须有 $\Delta_x = -v$**。
    
- **区间三：$x \in [r, n - 2]$（双侧包含区 / 抵消区 -> 等价于区间一）** 此时前缀和后缀足够长，两者都包含了修改点 $a[i]$。前缀和后缀同时增加 $v$，差值抵消！ **影响**：$\Delta_x$ 变化量为 $0$。我们需要原本 $\Delta_x = 0$ 的点。


### 3. 数据维护 ：如何去做到查询任意一个区间中等于`x`的个数 , 做到时间复杂度`log(n)`以下 ？

- **预处理**：遍历原数组，计算每个分割点 $x$ 对应的 $\Delta_x$。使用 `std::map<int, vector<int>>`（或者哈希表），将相同的 $\Delta_x$ 的下标 $x$ 收集到同一个数组中。由于是从左到右遍历，这些下标天然是**严格递增**的。
    
- **查询**：在对应的下标数组中，使用二分查找（`std::lower_bound` 和 `std::upper_bound`），能在 $O(\log N)$ 的时间内求出区间内的合法点数量。

### 4. 最后统计 

**拼接答案**：每次查询 $a[i] + v$，分别统计区间一（查 $0$）、区间二（查 $v$ 或 $-v$）、区间三（查 $0$）的合法点个数，求和得到总有效分割点数 $ans$，最终结果合理分情况输出。



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

int fac[N], inv[N];
int ksm(int n, int m){
    int res = 1;
    n %= mod;
    while(m){
        if(m & 1){
            res = res * n % mod;
        }
        n = n * n % mod;
        m = m >> 1;
    }
    return res;
}

int init = [](){
    fac[0] = inv[0] = 1;
    for(int i = 1; i < N; i++){
        fac[i] = fac[i - 1] * i % mod;
        inv[i] = ksm(fac[i], mod - 2);
    }
    return 0;
}();

int C(int n, int m){
    if(n < m) return 0;
    return fac[n] * inv[n - m] % mod * inv[m] % mod;
}

void solve() { 
    int n, m, q;
    cin >> n >> m >> q;
    vector<int> a(n);
    for(auto& x : a) cin >> x;
    
    int pre = 0;
    int back = 0;
    map<int,vector<int>> idx;
    for(int i = 0;i < n;i++) {
        pre += a[i];
        back += a[n - 1 - i];
        int x = back - pre;
        idx[x].push_back(i);
    }

    auto query = [&](int l, int r, int v) -> int {
        if(l > r) return 0;
        auto it = idx.find(v);
        if (it == idx.end()) return 0;
        
        auto& nums = it->second;
        
        auto l_it = std::lower_bound(nums.begin(), nums.end(), l);
        auto r_it = std::upper_bound(nums.begin(), nums.end(), r);
        
        return std::distance(l_it, r_it);
    };

    while(q--) {
        int i, v;
        cin >> i >> v;
        i--; 
        
        //特判 ：
        if(m == 1) {
            cout << 1 << endl;
            continue;
        }
        
        int l = min(i, n - 1 - i);
        int r = max(i, n - 1 - i);
        
        int ans = 0;
        ans += query(0, l - 1, 0);
        ans += query(r, n - 2, 0); 

        v = (i <= n - 1 - i) ? v : -v;
        ans += query(l, r - 1, v);
		//达不到分割要求（ans < m - 1）cout << 0 ; 否则输出 C(ans, m - 1)；
        cout << (ans < m - 1 ? 0 : C(ans, m - 1)) << endl;
    }
}

signed main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);
    cout.tie(0);
    int T = 1;
    cin >> T;
    while (T--) {
        solve();
    }
    return 0;
}
```