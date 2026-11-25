## 相关链接 ：

- 题目链接 ：[F-小月的取物_牛客周赛 Round 171](https://ac.nowcoder.com/acm/contest/144189/F)

## 思路 ：

本题属于**基于“操作不变量”状态压缩的线性 DP**。

直接模拟取物过程会产生 $2^{n-1}$ 种操作分支，但每次只能取走队首（第 $1$ 个）或队次（第 $2$ 个）物品这一规则，隐含着一个极强的结构性质：

> **操作不变量**：当我们把前 $i$ 件物品纳入考虑，并已经取走了其中 $i - 1$ 件后，**前 $i$ 件物品中必然只剩下 $1$ 件还留在队头**，其余未考虑的物品 $a_{i+1}, a_{i+2}, \dots, a_n$ 按原顺序紧随其后。

此外，在刚刚完成的第 $i - 1$ 次取物中，被取走的那件物品（即计算相邻差值 $\vert{}b_k - b_{k-1}\vert{}$ 所需的“前驱物品”），要么是**刚进入视野的 $a_i$**，要么是**之前留下的那件历史物品 $a_j$**（$1 \le j \le i - 1$）。

## 二、 动态规划设计

### 1. 状态定义

设阶段 $i$ 表示“已考虑前 $i$ 件物品，且取走了其中 $i - 1$ 件”。在 $1 \le j \le i - 1$ 的范围内， $\{a_j, a_i\}$ 这两件物品必然是一取一留，由此定义两类互斥状态：

- $dp[i][j][0]$：**刚取走 $a_i$**，把 $a_j$ 留在了队头时的最小代价（此时队列前端为 $[a_j,\ a_{i+1},\ \dots]$）；
    
- $dp[i][j][1]$：**刚取走 $a_j$**，把 $a_i$ 留在了队头时的最小代价（此时队列前端为 $[a_i,\ a_{i+1},\ \dots]$）。
    

> **空间优化技巧**：代码中利用滚动数组省去第一维 $i$，并用 `+ n` 偏移量将最后一维拍平——即用 `dp[now][j]` 表示 $dp[i][j][0]$，用 `dp[now][n + j]` 表示 $dp[i][j][1]$。

### 2. 初始化

- 当 $n = 1$ 时，直接输出 $0$；
    
- 当 $n = 2$ 时，无论先取哪一个，代价均为 $\vert{}a_1 - a_2\vert{}$；
    
- 当 $n \ge 3$ 时，从阶段 $i = 2$（已完成第 $1$ 次取物，初始代价为 $0$）开始递推：
    
    - 取走 $a_2$、留下 $a_1$ $\implies dp[2][1][0] = 0$（对应 `dp[now][1] = 0`）
        
    - 取走 $a_1$、留下 $a_2$ $\implies dp[2][1][1] = 0$（对应 `dp[now][n + 1] = 0`）


### 3. 状态转移方程

从阶段 $i - 1$ 推进到阶段 $i$ 时，引入新元素 $a_i$ 并执行第 $i - 1$ 次取物。此时上一阶段的历史遗留元素下标 $j$ 满足 $1 \le j \le i - 2$（因为 $a_{i-1}$ 已作为上一阶段的固定配对项出现，不能与自身重复配对）：

1. **从 $dp[i-1][j][0]$ 转移**（上一步刚取走 $a_{i-1}$，当前队列前端为 $[a_j,\ a_i,\ \dots]$）：
    
    - **取队首 $a_j$，留下 $a_i$**：
        $$dp[i][j][1] = \min\Big(dp[i][j][1],\ dp[i-1][j][0] + \vert{}a_{i-1} - a_j\vert{}\Big)$$
        
    - **取队次 $a_i$，留下 $a_j$**：
        $$dp[i][j][0] = \min\Big(dp[i][j][0],\ dp[i-1][j][0] + \vert{}a_{i-1} - a_i\vert{}\Big)$$
        
2. **从 $dp[i-1][j][1]$ 转移**（上一步刚取走 $a_j$，当前队列前端为 $[a_{i-1},\ a_i,\ \dots]$）：
    
    - **取队首 $a_{i-1}$，留下 $a_i$**：
        $$dp[i][i-1][1] = \min\Big(dp[i][i-1][1],\ dp[i-1][j][1] + \vert{}a_j - a_{i-1}\vert{}\Big)$$
        
    - **取队次 $a_i$，留下 $a_{i-1}$**：
        $$dp[i][i-1][0] = \min\Big(dp[i][i-1][0],\ dp[i-1][j][1] + \vert{}a_j - a_i\vert{}\Big)$$

### 4. 答案统计

循环结束于 $i = n$ 时，已取走 $n - 1$ 件物品，队列中仅剩最后 $1$ 件物品。无论剩下的是 $a_j$ 还是 $a_n$，最后一步的代价增量均为 $\vert{}a_n - a_j\vert{}$：

$$\text{ans} = \min_{1 \le j < n} \Big( \min(dp[n][j][0],\ dp[n][j][1]) + \vert{}a_n - a_j\vert{} \Big)$$

## 三、 复杂度分析

- **时间复杂度**：外层阶段枚举 $O(n)$，内层状态转移枚举 $O(n)$，总时间复杂度为 $O(n^2)$。
    
- **空间复杂度**：使用滚动数组压缩阶段维度，仅需维护大小为 $2 \times (2n + 1)$ 的数组，空间复杂度为 $O(n)$。



## 代码展示 ：

```c++
#include <bits/stdc++.h>
using namespace std;
#define int long long
#define endl '\n'
#define ctz __builtin_ctzll
#define clz __builtin_clzll
#define count1 __builtin_popcountll
const int mod = 1e9 + 7;
const int MOD = 998244353;
const int N = 2e5 + 7;
const int INF = 1e18;

// solve -> 2^(n - 1)次方的暴力写法，超时（赛时暴力代码）
// solve1 -> 线性dp写法

void solve() {
    int n;
    cin >> n;
    vector<int> a(n);
    for (int i = 0; i < n; i++) {
        cin >> a[i];
    }

    if (n == 1) {
        cout << 0 << endl;
        return;
    }
    if (n == 2) {
        cout << abs(a[0] - a[1]) << endl;
        return;
    }

    int ans = INF;
    vector<int> nxt(n, -1);
    for (int i = 0; i < n - 1; i++) {
        nxt[i] = i + 1;
    }

    auto dfs = [&](auto&& self, int fi, int se, int before, int val, int cen) -> void {
        if (cen == n - 2) {
            ans = min(ans, val + abs(a[fi] - a[se]) + min(abs(a[fi] - a[before]), abs(a[se] - a[before])));
            return;
        }

        if (cen) val += abs(a[fi] - a[before]);
        self(self, se, nxt[se], fi, val, cen + 1);
        if (cen) val -= abs(a[fi] - a[before]);

        if (cen) val += abs(a[se] - a[before]);
        self(self, fi, nxt[se], se, val, cen + 1);
        if (cen) val -= abs(a[se] - a[before]);
    };

    dfs(dfs, 0, 1, -1, 0, 0);
    cout << ans << endl;
}

void solve1() {
    int n;
    cin >> n;
    vector<int> a(n + 1);
    for(int i = 1; i <= n; i++) cin >> a[i];
    if(n == 1) {
        cout << 0 << endl;
        return;
    }
    if(n == 2) {
        cout << abs(a[1] - a[2]) << endl;
        return;
    }
    int len = 2 * n + 1;
    vector<vector<int>> dp(2, vector<int>(len, INF));
    int now = 0, pre = 1;
    dp[now][1] = dp[now][n + 1] = 0;
    for(int i = 3; i <= n; i++) {
        swap(now, pre);
        fill(dp[now].begin(), dp[now].end(), INF);
        
        // if(before_choose == [pre][j])
        for(int j = 1; j < i - 1; j++) {
            dp[now][n + j] = min(dp[now][n + j], dp[pre][j] + abs(a[j] - a[i - 1]));
            dp[now][j] = min(dp[now][j], dp[pre][j] + abs(a[i] - a[i - 1]));
        }
        // if(before_choose == [pre][n + j])
        for(int j = 1; j < i - 1; j++) {
            dp[now][n + i - 1] = min(dp[now][n + i - 1], dp[pre][n + j] + abs(a[j] - a[i - 1]));
            dp[now][i - 1] = min(dp[now][i - 1], dp[pre][n + j] + abs(a[j] - a[i]));
        }
    }
    int ans = INF;
    for(int i = 1; i < n; i++) {
        ans = min(ans, dp[now][i] + abs(a[i] - a[n]));
    }
    for(int i = 1; i < n; i++) {
        ans = min(ans, dp[now][i + n] + abs(a[i] - a[n]));
    }
    cout << ans << endl;
}

signed main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);
    cout.tie(0);
    int T = 1;
    // cin >> T;
    while (T--) {
        solve1();
    }
    return 0;
}
```