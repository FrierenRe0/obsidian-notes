## 相关链接：
	
- **题目链接 ：**
	- [P17515 [ECUSTPC 2026 Fall] 一分为二 2 - 洛谷](https://www.luogu.com.cn/problem/P17515?contestId=356798)
	- [F-小月的订单_牛客周赛 Round 166](https://ac.nowcoder.com/acm/contest/141877/F) (具体查看“题解” - 牛客周赛166)



## 评价：二分 + dp


## 题目概述：

给定两个长度为 $n$ 的数组 $a$ 和 $b$。

你可以任选 $k \in [0,n]$，将 $a$ 的前 $k$ 个元素和 $b$ 的后 $n-k$ 个元素拼接成一个长度为 $n$ 的新数组。

请从新数组中选出恰好 $m$ 个互不相邻的元素。记其中最小的元素为 $p$ ，求 $p$ 的最大值。

## 输入格式

第一行输入一个整数 $T$ ($1 \le T \le 10^4$)，表示测试数据的数量。

每组测试数据第一行输入两个整数 $n$ 和 $m$ ($1 \le m \le \lceil \frac{n}{2} \rceil \le n \le 2 \times 10^5$)，分别表示数组长度和需要选择的元素数量。

第二行输入 $n$ 个整数 $a_1, a_2, \ldots, a_n$ ($-10^9 \le a_i \le 10^9$)，表示数组 $a$。

第三行输入 $n$ 个整数 $b_1, b_2, \ldots, b_n$ ($-10^9 \le b_i \le 10^9$)，表示数组 $b$。

保证所有测试数据的 $\sum n \le 2 \times 10^5$。

## 输出格式

对于每组测试数据，输出一行一个整数，表示 $p$ 的最大值。

## 输入输出样例 #1

### 输入 #1

```
2
5 2
5 4 3 2 1
1 2 3 4 5
9 5
-1 1 -4 -5 -1 4 -1 -9 1
-9 -9 -8 2 4 -4 -3 -5 3
```

### 输出 #1

```
5
-4
```

## 说明/提示

对于第 $1$ 组测试数据，当 $k = 4$ 时，新数组为 $[5, 4, 3, 2, 5]$，选择第 $1,5$ 个元素，最小元素为 $5$，可以证明这是最大值。



## 题目分析：

- **二分答案**：
    
    题目要求最大化所选 $m$ 个互不相邻元素中的最小值 $p$。因为满足单调性（如果某个值 $X$ 可以作为最小值达成目标，那么任何小于 $X$ 的值也可以），我们可以对 $p$ 的可能取值（即数组 $a$ 和 $b$ 中出现的所有元素）进行**二分查找**。
    
- **动态规划检验（Check 函数）**：
    
    对于二分出的一个固定目标值 $X$：
    
    - 我们需要判断是否存在某个分割点 $k \in [0, n]$，使得将 $a$ 的前 $k$ 个元素与 $b$ 的后 $n-k$ 个元素拼接成的新数组中，能够选出 $\ge m$ 个**互不相邻**且**全部 $\ge X$** 的元素。
        
    - **前缀 DP（针对数组 $a$）**：预处理出对于每个 $k$，前 $k$ 个元素中满足条件的最大独立集。区分两种状态：
        
        - $A0[k]$：第 $k$ 个元素**不选**时的最大独立集。
            
        - $A1[k]$：第 $k$ 个元素**选**时的最大独立集（若 $a_k < X$ 则设为 $-\infty$）。
            
    - **后缀 DP（针对数组 $b$）**：预处理出从位置 $j$ 到 $n$ 的后缀中满足条件的最大独立集：
        
        - $B0[j]$：第 $j$ 个元素**不选**时的最大独立集。
            
        - $B1[j]$：第 $j$ 个元素**选**时的最大独立集（若 $b_j < X$ 则设为 $-\infty$）。
            
    - **边界拼接**：
        
        当分割点为 $k$ 时，数组的交界处是 $a_k$（来自 $a$ 的末尾）和 $b_{k+1}$（来自 $b$ 的开头）。由于它们在拼接后的数组中是**相邻**的，因此**不能同时选择**。
        
        对于每个 $k$，组合后的最大可选元素数量为：$$\max \Big( A0[k] + \max(B0[k+1], B1[k+1]), \max(A0[k], A1[k]) + B0[k+1] \Big)$$


## 代码展示：

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
    int n,m;
    cin >> n >> m;
    vector<int> a(n + 1), b(n + 1);
    vector<int> vals;
    vals.reserve(2 * n);
    
    for (int i = 1; i <= n; ++i) {
        cin >> a[i];
        vals.push_back(a[i]);
    }
    for (int i = 1; i <= n; ++i) {
        cin >> b[i];
        vals.push_back(b[i]);
    }

    // 离散化
    sort(vals.begin(), vals.end());
    vals.erase(unique(vals.begin(), vals.end()), vals.end());

    vector<int> A0(n + 1), A1(n + 1);
    vector<int> B0(n + 2), B1(n + 2);

    auto check = [&](int X) -> bool {
        //数组 a 的前缀 DP
        A0[0] = 0;
        A1[0] = INT_MIN;
        for (int i = 1; i <= n; ++i) {
            A0[i] = max(A0[i - 1], A1[i - 1]);
            if (a[i] >= X) {
                A1[i] = A0[i - 1] + 1;
            } else {
                A1[i] = INT_MIN;
            }
        }

        //数组 b 的后缀 DP
        B0[n + 1] = 0;
        B1[n + 1] = INT_MIN;
        for (int i = n; i >= 1; --i) {
            B0[i] = max(B0[i + 1], B1[i + 1]);
            if (b[i] >= X) {
                B1[i] = B0[i + 1] + 1;
            } else {
                B1[i] = INT_MIN;
            }
        }

        for (int k = 0; k <= n; ++k) {
            int j = k + 1;
            int max_mis = max(A0[k] + max(B0[j], B1[j]),max(A0[k], A1[k]) + B0[j]);
            if (max_mis >= m) return true;
        }
        return false;
    };

    int l = 0, r = (int)vals.size() - 1;
    int ans = vals[0];
    
    while (l <= r) {
        int mid = l + (r - l) / 2;
        if (check(vals[mid])) {
            ans = vals[mid];
            l = mid + 1; 
        } else {
            r = mid - 1; 
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
    cin >> T;
    while (T--) {
        solve();
    }
    return 0;
}
```