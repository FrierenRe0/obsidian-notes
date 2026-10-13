## 相关链接：
- 题目链接：[Problem - E - Codeforces](https://codeforces.com/contest/1748/problem/E)

## 评价：
- 笛卡尔树建立数组中的所有元素的相对大小关系，可以理解为这里的笛卡尔树对于数组时一个镜像关系；
- 在去通过树形dp完成对于方案数的统计；


## 题目大意：

给定一个长度为 $n$ 的数组 $a$，求有多少个长度为 $n$ 的整数数组 $b$，满足：

1. $1 \le b_i \le m$ （所有元素在 $1$ 到 $m$ 之间）。
    
2. 对于任意区间 $[l, r]$，数组 $b$ 在该区间上的**左侧最大值位置**与数组 $a$ 在该区间上的**左侧最大值位置**完全相同。
    
    输出方案数对 $10^9 + 7$ 取模的结果。


## 思路：

### 一、 核心转化：笛卡尔树同构

区间左侧最大值的位置完全一致，在数学上等价于：**数组 $b$ 的大根堆笛卡尔树结构必须与数组 $a$ 的笛卡尔树完全相同**。

因此，我们可以将原问题拆解为两步：

1. **建树**：通过单调栈，以 $a$ 为基础构建一棵大根堆笛卡尔树。
    
2. **树形 DP**：在固定的树形结构上，计算有多少种给每个节点赋权值（$1 \sim m$）的方案，使得满足大根堆性质。


### 二、 细节处理：左侧最大值的不对称限制

因为题目要求的是“左侧最大值”（即当数值相同时，靠左的元素优先成为最大值），所以在构建笛卡尔树和树形 DP 时，左右子树的权值限制是不对称的：

- **相等元素处理**：在单调栈建树时，遇到相同值的元素**不弹栈**（使用严格小于 `<` 弹栈），这保证了**先出现的元素（靠左）会成为后出现元素（靠右）的祖先**。
    
- **左子树限制**：左子树的所有节点必须**严格小于**当前节点。因为如果左子树存在 $\ge$ 当前节点的数，由于它在左侧，会抢占“左侧最大值”的位置。
    
- **右子树限制**：右子树的所有节点可以**小于或等于**当前节点。因为右子树在右侧，即使值相同，由于“左侧优先”原则，当前节点依然是赢家。

**当然，这里我来个变形，如果题目说这里是当数值相同时，靠右的元素优先成为最大值；反知在建树的时候就是  $\le$  的情况要弹栈了。**



### 三、 动态规划设计

设 $dp[u][v]$ 表示：**以节点 $u$ 为根的子树中，节点 $u$ 的权值 $\le v$ 时的方案总数**。

#### 1. 状态转移方程

当节点 $u$ 的权值恰好为 $v$ 时，根据乘法原理，左右子树的填数方案是独立的：

- 左子树由于要求严格小于 $v$，其最大值上限为 $v - 1$，方案数为：
    
    $$\text{left\_ways} = \begin{cases} dp[\text{left}[u]][v - 1], & \text{若左子树存在} \\ 1, & \text{若无左子树} \end{cases}$$
    
- 右子树由于要求小于等于 $v$，其最大值上限为 $v$，方案数为：
    
    $$\text{right\_ways} = \begin{cases} dp[\text{right}[u]][v], & \text{若右子树存在} \\ 1, & \text{若无右子树} \end{cases}$$
    

当节点 $u$ 权值恰好为 $v$ 时的总方案数：

$$\text{ways}(v) = (\text{left\_ways} \times \text{right\_ways}) \pmod{10^9+7}$$

由于 $dp[u][v]$ 要求的是权值 $\le v$ 的累加和，因此可以利用前缀和思想进行滚动累加：

$$dp[u][v] = (dp[u][v - 1] + \text{ways}(v)) \pmod{10^9+7}$$



## 代码如下：
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
    vector<int> a(n + 1);
    for(int i = 1;i <= n;i++) cin >> a[i];
    //
    vector<int> left(n + 1),right(n + 1);
    vector<int> st(n + 1);
    int top = 0;
    auto build = [&]() -> void {
        for(int i = 1;i <= n;i++) {
            int p = top;
            int before = 0;
            while(p > 0 && a[st[p]] < a[i]) {
                before = st[p];
                p--;
            }
            if(p) {
                right[st[p]] = i;
            }
            if(before) {
                left[i] = before;
            }
            top = p + 1;
            st[top] = i;
        }
    };
    build();
    int fa = st[1];

    //树形dp
    vector<vector<int>> dp(n + 1,vector<int>(m + 1));
    auto dfs = [&](this auto&& dfs,int x) -> void {
        if(x == 0) return;
        if(left[x]) dfs(left[x]);
        if(right[x]) dfs(right[x]);
        
        for(int i = 1;i <= m;i++) {
            int cnt = 1;
            if(left[x]) cnt = cnt * dp[left[x]][i - 1] % mod;
            if(right[x]) cnt = cnt * dp[right[x]][i] % mod;

            dp[x][i] = (dp[x][i - 1] + cnt) % mod; 
        } 
    };
    dfs(fa);
    cout << dp[fa][m] << endl;
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