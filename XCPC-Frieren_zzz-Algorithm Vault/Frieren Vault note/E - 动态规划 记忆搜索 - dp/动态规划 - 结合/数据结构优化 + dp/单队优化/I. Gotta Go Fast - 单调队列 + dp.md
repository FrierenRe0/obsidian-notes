## 相关链接 ：

- 题目链接 ：[Problem - I - Codeforces](https://codeforces.com/gym/106551/problem/I)


## 题目 ：

Have you ever played Eggy Party? inf loves playing Eggy Party.

Four eggies are stacked into a four-story eggy tower, with the bottom initially at coordinate $0$. From bottom to top, they are sequentially labeled A, B, C, and D, where D is the topmost eggy.

The total length of the track is $l$, which consists of $l$ segments of length $1$. Segment $i$ connects coordinates $i-1$ and $i$ $(1 \le i \le l)$.

The terrain of the $i$\-th segment is given by the character $S_i$. If the eggy tower walks from coordinate $i-1$ to coordinate $i$, the cost is:

-   Flat ground `0`: $t_0$ seconds;
-   Mud pit `1`: $t_1$ seconds;
-   Speed pad `2`: $t_2$ seconds.

The eggy tower can perform two types of operations:

-   **Normal Advance:** If the current eggy tower is at coordinate $x \lt l$, it can advance forward to coordinate $x+1$. The time cost of this advance **depends only on the terrain of segment $x+1$**.
-   **Super Start:** If there is an eggy tower containing $j$ eggies at coordinate $x$ $(j \ge 2)$, the bottommost eggy can instantly throw the $j-1$ eggies above it to coordinate $\min(l, x+d)$, where $d$ is a given positive integer. This operation does not trigger any terrain time cost, i.e., it takes $0$ seconds. After the operation, the bottommost eggy at the original position is eliminated, leaving that position empty; a new eggy tower containing $j-1$ eggies is formed at coordinate $\min(l, x+d)$, and the vertical order of the remaining eggies stays the same.

Please compute the minimum time required for the initially topmost eggy D to reach the destination coordinate $l$.


**Input**

The first line contains two positive integers $l$ and $d$ ($1 \le d \lt l \le 2 \times 10^5$).

The second line contains three positive integers $t_0$, $t_1$, $t_2$ ($1 \le t_2 \lt t_0 \lt t_1 \le 10^9$).

The third line contains a string $S$ of length $l$, consisting only of the characters `0`, `1`, and `2`.


**Output**

Output a single integer representing the minimum total time.


## 思路 ：

这份代码采用的是**DP + 单调队列优化**。整体思路非常标准：将“剩余蛋仔数量”作为阶段维度，利用单调队列维护长度为 $d$ 的滑动窗口最小值，将每次跳跃转移的复杂度从 $O(d)$ 降至均摊 $O(1)$。

### 一、 状态定义与转移模型

#### 1. 状态表示

设 `dp[i][j]` 表示当前蛋仔塔中还剩 $i$ 个蛋仔（$1 \le i \le 4$），且位于坐标 $j$（$0 \le j \le n$）时的最小耗时。

- **初始状态**：起点 $0$ 处有 $4$ 个蛋仔，故 `dp[4][0] = 0`，其余状态初始化为 `LLONG_MAX / 2`（防加法溢出）。
    
- **最终答案**：只要顶层蛋仔 $D$ 抵达终点 $n$ 即可，此时塔内可能剩余 $1 \sim 4$ 个蛋仔，取最小值 $\min_{1 \le i \le 4} dp[i][n]$。
    

#### 2. 状态转移方程

到达状态 $(i, j)$ 有两种决策：

1. **普通前进**：蛋仔数不变，从 $j - 1$ 走过第 $j$ 段地形（下标 `j - 1`）：
    
    $$dp[i][j] = \min\Big(dp[i][j],\ dp[i][j - 1] + val[id(S_{j-1})]\Big)$$
    
2. **超级起步（滑动窗口模型）**：消耗底层 $1$ 个蛋仔（从 $i + 1$ 变为 $i$，$i \le 3$），花费 $0$ 秒从距离不超过 $d$ 的前方位置 $k \in [\max(0, j - d),\ j - 1]$ 跳跃过来：
    
    $$dp[i][j] = \min\left(dp[i][j],\ \min_{\max(0, j - d) \le k \le j - 1} dp[i + 1][k]\right)$$
    

### 二、 单调队列优化的核心实现细节

代码中用 `vector<deque<int>> q(5)` 为第 $2, 3, 4$ 层分别维护了一个**队首到队尾 `dp` 值单调递增**的双端队列，存储的是候选起跳坐标下标 $k$。其中有四个非常关键的工程细节：

1. **遍历顺序保证转移合法性（先查询、后入队）** 外层循环按坐标 $j$ 从 $1$ 到 $n$ 推进，内层循环按剩余蛋仔数 $i$ 从 $1$ 到 $4$ 枚举：
    
    - 计算 `dp[i][j]` 时，直接取 `q[i + 1].front()`。
        
    - 由于 $i$ 从小到大遍历，此时第 $i + 1$ 层的当前坐标 $j$ **尚未入队**，`q[i + 1]` 中保存的严格是 $[j - d, j - 1]$ 内的历史坐标。这天然杜绝了在同一坐标 $j$ 连续多次消耗蛋仔的非法转移。
        
2. **初始全层入队避免判空** 初始化时执行了 `q[2].push_back(0); q[3].push_back(0); q[4].push_back(0);`。虽然 `dp[2][0]` 和 `dp[3][0]` 均为无穷大，但提前放入下标 `0` 可以保证后续 `j = 1` 访问 `q[i + 1].front()` 时队列永不为空，省去了繁琐的判空分支。
    
3. **队尾维护单调性（去劣）** 当算完 `x = dp[i][j]`（$i \ge 2$）准备将其作为后续坐标的起跳点时：
    
    C++
    
    ```
    while (!dq.empty() && dp[i][dq.back()] >= x) {
        dq.pop_back();
    }
    dq.push_back(j);
    ```
    
    如果队列尾部旧坐标的耗时大于等于当前 `dp[i][j]`，由于 `j` 更靠右（存活时间更长）且耗时更少，旧坐标永远不可能成为最优解，直接弹出。
    
4. **队首滑动窗口过期淘汰**
    
    C++
    
    ```
    if (dq.front() == j - d) dq.pop_front();
    ```
    
    在坐标 $j$ 的末尾淘汰下标 `j - d`，使得队列中保留的下标区间恰好收束为 $[j - d + 1, j]$。当外层循环进入下一步 $j' = j + 1$ 时，队列内的候选区间正好对应 $[j' - d, j' - 1]$，与转移方程完全对齐。
    

### 三、 复杂度分析

- **时间复杂度**：共有 $4 \times l$ 个状态，每个坐标 $j$ 在每层最多入队、出队各一次，单次状态转移与队列维护均为均摊 $O(1)$，总时间复杂度为 $O(l)$。
    
- **空间复杂度**：`dp` 表与三个双端队列最多同时存储 $O(l)$ 个元素，空间复杂度为 $O(l)$。


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
    int l,d;
    cin >> l >> d;
    vector<int> val(3);
    for(int i = 0;i < 3;i++) {
        cin >> val[i];
    }
    string s;
    cin >> s;
    int n = s.size();
    vector<vector<int>> dp(6,vector<int>(n + 1,LLONG_MAX / 2));
    auto id = [&](char c) -> int {
        return c - '0';
    };
    vector<deque<int>> q(5);
    dp[4][0] = 0;
    q[2].push_back(0);
    q[3].push_back(0);
    q[4].push_back(0);

    for(int j = 1;j <= n;j++) {
        for(int i = 1;i <= 4;i++) {
            // butong
            dp[i][j] = min(dp[i][j],dp[i][j - 1] + val[id(s[j - 1])]);
            // super
            if(i <= 3) {
                dp[i][j] = min(dp[i][j],dp[i + 1][q[i + 1].front()]);
            }
        
            //维护deque;
            if(i >= 2) {
                auto x = dp[i][j];
                auto& dq = q[i];
                while(!dq.empty() && dp[i][dq.back()] >= x) {
                    dq.pop_back();
                }
                dq.push_back(j);
                if(dq.front() == j - d) dq.pop_front();
            }
        }
    }
 
    int ans = LLONG_MAX / 2;
    for(int i = 1;i <= 4;i++) {
        ans = min(ans,dp[i][n]);
    }
    cout << ans;
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

666