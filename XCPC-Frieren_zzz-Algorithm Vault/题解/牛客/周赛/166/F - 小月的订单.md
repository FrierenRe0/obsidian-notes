## 相关链接 ：
- 题目链接 ：[F-小月的订单_牛客周赛 Round 166](https://ac.nowcoder.com/acm/contest/141877/F)


## 评价 ：dp处理 + 二分
	
- 但是这里需要注意内存开销问题，需要采用坐标压缩解决（该题是一个3维压缩）


## 题目 ： 

![](../../../../截图文件/Pasted%20image%2020261019115850.png)
## 题目信息 ：

1. **输入规模**：$n \le 4 \times 10^3$，$s \le 10^{15}$，$a_i, b_i \le 10^9$。
    
2. **批次划分**：
    
    - 分为两批，每批各选 $k$ 个订单。
        
    - **严格先后顺序**：第一批所有被选订单的编号，必须全部小于第二批所有被选订单的编号（即中间有一个分界点 $i$，前 $i$ 个订单选第一批，后 $i+1 \sim n$ 个订单选第二批）。
        
3. **互斥限制**：
    
    - 任意两份被处理的订单编号之差绝对值不能为 $1$。
        
    - 这意味着：**不能选择相邻的订单**。不仅批次内部不能相邻，**跨批次**（第一批最后一个选的订单与第二批第一个选的订单）也不能相邻！如果第一批选了第 $i$ 个，第二批最早只能从 $i + 2$ 开始选。
        
4. **目标**：求满足总费用 $\le s$ 的**最大非负整数 $k$**。




## 一、 核心思路 ：

### 1. 二分答案 + 判定性问题 (Binary Search)

- 由于 $k$ 具有单调性（若能选 $k$ 个，则更小的数也能满足；反之亦然），我们可以对结果 $k$ 进行**二分答案**。
    
- 二分范围：$l = 0, r = \lfloor n / 2 \rfloor$。
    
- 对于每一个中间值 `mid`，我们需要写一个判定函数 `check(mid)`：**判断是否存在一种选法，使得两批各选 `mid` 个订单，且总费用不超过 $s$。**
    

### 2. 动态规划设计 (DP State Definition)

在 `check(mid)` 内部，我们需要求出在选定个数为 `mid` 的前提下的**最小总费用**。由于第一批和第二批有严格的前后分界，我们可以分别预处理前缀和后缀的 DP：

- **`pre[i][k][0/1]`**：
    
    - 含义：考虑前 $i$ 个订单，第一批选了 $k$ 个，且第 $i$ 个订单不选（0）/ 选（1）的最小费用。
        
    - 状态转移：
        
        - 不选：`pre[i][k][0] = min(pre[i-1][k][0], pre[i-1][k][1])`
            
        - 选：`pre[i][k][1] = pre[i-1][k-1][0] + a[i]` （注意：因为不能选相邻，所以必须从 `i-1` 不选的状态转移过来）
            
- **`back[i][k][0/1]`**：
    
    - 含义：考虑从后往前第 $i \dots n$ 个订单，第二批选了 $k$ 个，且第 $i$ 个订单不选（0）/ 选（1）的最小费用。
        
    - 状态转移：
        
        - 不选：`back[i][k][0] = min(back[i+1][k][0], back[i+1][k][1])`
            
        - 选：`back[i][k][1] = back[i+1][k-1][0] + b[i]`


### 3. 二分答案完成状态合并

当我们有了 `pre` 和 `back` 表之后，枚举第一批的结束分界点 $i$（从 $1$ 到 $n$）：

1. 若第 $i$ 个订单被第一批选中（`pre[i][mid][1]`）：
    
    - 此时的最小费用为：`pre[i][mid][1] + back[i + 1][mid][0]`
    
2. 若第 $i$ 个订单没被第一批选中（`pre[i][mid][0]`）：
    
    - 此时的最小费用为：`pre[i][mid][0] + min(back[i+1][mid][0], back[i+1][mid][1])`

如果在所有合法的分界点 $i$ 中，存在一个方案的总费用 $\le s$，则说明当前 `mid` 可行，维护最小开销比对，合法返回 `true`，反之`false`。



## 二、 空间优化与细节处理 (Space Optimization)

由于 $n \le 4 \times 10^3$，如果直接使用三维 `vector<vector<vector<int>>>`：

- 内存占用会达到数百万个独立动态对象，导致 **MLE（内存超限）**。
    
- **优化策略**：采用**一维数组扁平化（Flattening / 空间降维）**，把多维下标映射到一段连续的一维内存中：
    
    $$\text{Index} = (i \times \text{dim\_k} + k) \times 2 + \text{state}$$
    
- 这样不仅消除了海量指针和控制块的内存碎片开销，还保证了极致的连续内存访问速度。



## 代码如下：
```c++
#include <bits/stdc++.h>
using namespace std;
#define int long long
#define endl '\n'
#define ctz __builtin_ctzll         // 二进制表示中末尾连续0的个数
#define clz __builtin_clzll         // 二进制表示中先导0的个数
#define count1 __builtin_popcountll // 二进制表示中1的个数

const int mod = 1e9 + 7;
const int MOD = 998244353;
const int N = 2e5 + 7;
const int INF = 1e18;

void solve() {
    int n, s;
    cin >> n >> s;
    vector<int> a(n + 1), b(n + 1);
    for (int i = 1; i <= n; i++) cin >> a[i];
    for (int i = 1; i <= n; i++) cin >> b[i];
    
    int m = (n + 1) / 2;
    int mx = m / 2;
    
    //3维下标压缩 防止内存爆炸
    // 寻址公式: idx(i, k, state) = (i * (mx + 1) + k) * 2 + state
    int dim_k = mx + 1;
    int dim_state = 2;
    auto get_idx = [&](int i, int k, int state) {  // based -> 0;
        return (i * dim_k + k) * dim_state + state;
    };
    
    int total_size = (n + 3) * dim_k * dim_state;
    vector<int> pre(total_size, INF);
    vector<int> back(total_size, INF);
    
    // pre 预处理
    pre[get_idx(0, 0, 0)] = 0;
    for (int i = 1; i <= n; i++) {
        for (int k = 0; k <= mx; k++) {
            // no choose :      
            pre[get_idx(i, k, 0)] = min(pre[get_idx(i - 1, k, 0)], pre[get_idx(i - 1, k, 1)]);
            // choose :
            if (k >= 1) {
                pre[get_idx(i, k, 1)] = pre[get_idx(i - 1, k - 1, 0)] + a[i];
            }
        }
    }
        
    // back 预处理
    back[get_idx(n + 1, 0, 0)] = 0;
    for (int i = n; i >= 1; i--) {
        for (int k = 0; k <= mx; k++) {
            // no choose :
            back[get_idx(i, k, 0)] = min(back[get_idx(i + 1, k, 0)], back[get_idx(i + 1, k, 1)]);
            // choose :
            if (k >= 1) {
                back[get_idx(i, k, 1)] = back[get_idx(i + 1, k - 1, 0)] + b[i];
            }
        }
    }
    
    auto check = [&](int mid) -> bool {
        if (mid == 0) return true;
        int mi = INF;
        for (int i = mid * 2 - 1; i < n; i++) {
            int cur = min(pre[get_idx(i, mid, 1)] + back[get_idx(i + 1, mid, 0)], 
                          pre[get_idx(i, mid, 0)] + min(back[get_idx(i + 1, mid, 1)], back[get_idx(i + 1, mid, 0)]));
            mi = min(mi, cur);
        }
        return mi <= s;
    };
    
    int l = 0, r = mx;
    int ans = 0;
    while (l <= r) {
        int mid = (l + r) >> 1;
        if (check(mid)) {
            ans = mid;
            l = mid + 1;
        } else {
            r = mid - 1;
        }
    }
    cout << ans;
    return;
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