## 相关链接：
- 题卷查看：[第10场题卷](../题卷/第10场.pdf)
- 题目链接： [K-Team Formation_2026牛客暑期多校训练营10](https://ac.nowcoder.com/acm/contest/133885/K)

## 评价：
状态压缩(二进制数位优化) dp

## 题目大意：

![](../../../../../截图文件/Pasted%20image%2020260820171422.png)

### 题目解析：

有 $3n$ 名学生，需要将他们恰好分成 $n$ 个三元组（每组 3 人）。每两个人之间有一个兼容度得分 $res[i][j]$。一个三元组 $(i, j, k)$ 的得分为三名成员两两之间兼容度之和：

$$\text{Score}(i, j, k) = res[i][j] + res[j][k] + res[k][i]$$

要求找到一种分组方案，使得所有三元组的得分之和最大。

## 思路 :

### 记忆化搜索写法：

- **状态定义**：
    
    - `dfs(mask)` 函数表示：**当已被选走的学生集合状态为 `mask` 时，剩余尚未分配的学生后续还能凑出的最大总得分**。
        
    - 二进制位 `0` 表示学生还在，`1` 表示已被选走。
        
- **状态转移**：
    
    - 找到 `mask` 中从低到高**第一个为 `0` 的学生**（记为 `first`）。强行锁定他来发起当前三元组，消除组合顺序的重复。
        
    - 从剩余未分配（位为 `0`）的学生中枚举另外两人 `j` 和 `k`（满足 $\text{first} < j < k$）。
        
    - 转移方程：
        
        $$\text{dfs}(\text{mask}) = \max_{(j, k)} \left( \text{res}[\text{first}][j] + \text{res}[j][k] + \text{res}[k][\text{first}] + \text{dfs}(\text{next\_mask}) \right)$$
        
        其中 $\text{next\_mask} = \text{mask} \mid (1 \ll \text{first}) \mid (1 \ll j) \mid (1 \ll k)$。
        
- **递归边界**：
    
    - 当 `mask == (1 << m) - 1`（所有二进制位全为 `1`，即所有人均已分配完毕）时，后续无法再获得分数，直接返回 `0`。

### dp写法：

-  **状态定义**：
    
    使用二进制数 `mask` 表示每个学生的分配状态：
    
    - **`0`**：表示该学生**尚未被分配**。
        
    - **`1`**：表示该学生**已经被选走**并加入某个队伍。
        
    - **`dp[mask]`**：当已被选走的学生集合状态为 `mask` 时，当前能获得的**最大累积得分**。
        
- **状态转移（刷表法）**：
    
    - **起点**：初始时没有人被选走，即 `dp[0] = 0`，其余状态初始化为 $-\infty$。
        
    - **去重策略**：为了彻底消除不同三元组组合顺序导致的重复计算，我们在扩展新队伍时，**强行锁定当前尚未被分配的学生中编号最小的那一位**（即 `mask` 中从低位到高位遇到的第一个 `0`，记为 `first`）。
        
    - **转移方程**：在 `mask` 中寻找另外两个未分配的学生 `j` 和 `k`（满足 $\text{first} < j < k$），将他们三人选中并置为 `1`，得到新状态 `next_mask`：
        
        $$\text{next\_mask} = \text{mask} \mid (1 \ll \text{first}) \mid (1 \ll j) \mid (1 \ll k)$$
        
        $$\text{dp}[\text{next\_mask}] = \max(\text{dp}[\text{next\_mask}], \text{dp}[\text{mask}] + \text{res}[\text{first}][j] + \text{res}[j][k] + \text{res}[k][\text{first}])$$
        
    - **终点**：所有人均被选走（二进制位全为 `1`），答案即为 `dp[(1 << m) - 1]`。



## 复杂度分析：

- **时间复杂度**：$O(n \cdot 2^{3n})$。通过锁定当前编号最小的 `0`，大幅剪去了冗余组合，当 $n \le 8$ ($3n \le 24$) 时，有效运算量在 $10^7 \sim 10^8$ 级别，可在 **0.2 秒** 内高效通过。

- **空间复杂度**：$O(2^{3n})$，需开辟 $2^{24} \times 8 \text{ B} \approx 134 \text{ MB}$ 的 DP 数组，符合内存限制。



## 代码如下：

### 记忆化搜索：
```c++
#include <bits/stdc++.h>
using namespace std;
#define int long long
const int MI = -1e18;

void solve() {
    int n;
    if (!(cin >> n)) return;
    int m = 3 * n;
    vector<vector<int>> res(m, vector<int>(m));
    for (int i = 0; i < m; i++) {
        for (int j = 0; j < m; j++) {
            cin >> res[i][j];
        }
    }

    // memo[mask] 记录当前被选中的点状态为 mask 时的【后续最大能获得的追加收益】
    // 初始化为 -1 代表未访问过
    vector<int> memo(1 << m, MI);

    function<int(int)> dfs = [&](int mask) -> int {
        // 所有元素分配完毕，不再增加额外收益
        if (mask == (1 << m) - 1) {
            return 0;
        }

        // 命中记忆化缓存，直接返回
        if (memo[mask] != MI) {
            return memo[mask];
        }

        // 强行锁定当前【编号最小且未被使用】的点，消除组合顺序重复
        int i = 0;
        while (i < m && (mask & (1 << i))) {
            i++;
        }

        int mx = -1e18; // 记录当前状态下的最优子解

        // 枚举另外两个未使用的点 j 和 k (保证 i < j < k 避免内部重复)
        for (int j = i + 1; j < m; j++) {
            if (mask & (1 << j)) continue;
            for (int k = j + 1; k < m; k++) {
                if (mask & (1 << k)) continue;

                // 算上 1-based 偏移的权值 (因为输入是 1 到 m)
                int add = res[i][j] + res[j][k] + res[k][i];
                int next_mask = mask | (1 << i) | (1 << j) | (1 << k);

                mx = max(mx, add + dfs(next_mask));
            }
        }

        return memo[mask] = mx; // 记录答案并返回
    };

    cout << dfs(0) << '\n';
}

signed main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);
    cout.tie(0);
    int T = 1;
    // cin >> T;
    while (T--) {
        solve();
    }
    return 0;
}
```


### 递推dp:
```c++
#include <bits/stdc++.h>
using namespace std;
#define int long long

void solve() {
    int n;
    if (!(cin >> n)) return;
    int m = 3 * n;
    vector<vector<int>> res(m, vector<int>(m));
    for (int i = 0; i < m; i++) {
        for (int j = 0; j < m; j++) {
            cin >> res[i][j];
        }
    }

    // dp[mask] 表示已被选择的元素集合状态为 mask 时的最大收益
    vector<int> dp(1 << m, -1e18);
    dp[0] = 0; // 初始状态：一个人都没选，收益为 0

    // 从状态 0 递推到全集 (1 << m) - 1
    for (int mask = 0; mask < (1 << m); mask++) {
        if (dp[mask] == -1e18) continue; // 不可达状态跳过

        // 强行锁定当前未使用的最小编号，彻底消除重复搜索
        int first = -1;
        for (int i = 0; i < m; i++) {
            if (!(mask & (1 << i))) {
                first = i;
                break;
            }
        }

        if (first == -1) continue; // 全部元素已分配完毕

        // 从剩下的未使用元素中选取 j 和 k
        for (int j = first + 1; j < m; j++) {
            if (mask & (1 << j)) continue;
            for (int k = j + 1; k < m; k++) {
                if (mask & (1 << k)) continue;

                int next_mask = mask | (1 << first) | (1 << j) | (1 << k);
                int add = res[first][j] + res[j][k] + res[k][first];
                
                // 状态转移方程
                dp[next_mask] = max(dp[next_mask], dp[mask] + add);
            }
        }
    }

    cout << dp[(1 << m) - 1] << '\n';
}

signed main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);
    cout.tie(0);
    int T = 1;
    while (T--) {
        solve();
    }
    return 0;
}
```