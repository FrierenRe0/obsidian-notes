## 相关链接 ：

 - 题目链接：[问题 - G2 - Codeforces --- Problem - G2 - Codeforces](https://codeforces.com/contest/2290/problem/G2)


## 题目 ：

**This is the hard version of the problem. The two versions differ only in the constraints on $n$, on the sum of $n$ over all test cases, and on the number of test cases.**

A circular trail around a lake consists of $n$ sections numbered from $1$ to $n$ along the trail. The height of the $i$\-th section is $h_i$. Sections $i$ and $i+1$ are adjacent for each $i$ from $1$ to $n-1$, and sections $n$ and $1$ are adjacent as well.

The steepness of the trail is the maximum absolute difference between the heights of two adjacent sections. In other words, the steepness is $\max_{1 \le i \le n} \left|h_i - h_{i+1}\right|$, where $h_{n+1}$ should be considered to exist and to be equal to $h_1$.

You have $k$ units of soil. One unit of soil increases the height of one section by $1$; several units may be used on the same section. Soil cannot be removed, so the heights can only increase.

Find the minimum possible steepness of the trail after using at most $k$ units of soil.

**Input**

Each test contains multiple test cases. The first line contains the number of test cases $t$ ($1 \le t \le 10^4$). The description of the test cases follows.

The first line of each test case contains two integers $n$ and $k$ ($3 \le n \le 10^6$, $0 \le k \le 10^{18}$) — the number of sections and the number of units of soil.

The second line contains $n$ integers $h_1, h_2, \ldots, h_n$ ($1 \le h_i \le 10^9$) — the heights of the sections.

It is guaranteed that the sum of $n$ over all test cases does not exceed $10^6$.



## 分析 ：

解决本题的核心思维是**二分答案**。题目要求找到在消耗不超过 $k$ 单位泥土的情况下，能达到的**最小的最大相邻高度差**。 

因为“最大高度差”具有明显的单调性：要求的高度差越大，需要填的土越少；要求的高度差越小，需要填的土越多。因此，我们可以二分枚举目标高度差 $mid$，然后编写一个 `check(mid)` 函数，计算为了让全图相邻高度差都不超过 $mid$，最少需要填多少土。

### 现在关键就是对于`check`函数这么去处理 ？ 
在 `check(mid)` 函数中，我们需要根据限制条件计算出每个点最终的最低高度 $lim[i]$。 限制条件为：对于任意相邻的 $x$ 和 $y$，必须满足 $\vert{}lim[x] - lim[y]\vert{} \le mid$。 转化为单向约束即为：**$lim[y] \ge lim[x] - mid$**。 这意味着，如果某一座山峰很高，它的邻居不能太矮，至少要达到“当前山峰高度减去 $mid$”的高度。高度要求会像波浪一样向四周传递。



## 思路：
### 方法一：优先队列 / Dijkstra 扩散法

**逻辑解析：** 将高度传递的过程视作图论中的**多源最长路问题**（或者类似 Dijkstra 的松弛过程）。

1. 初始时，所有点的最低要求就是它们原本的高度 $h[i]$。
    
2. 我们用一个大根堆（优先队列）来维护当前“对周围要求最高”的山峰。
    
3. 每次从队列中取出高度最大的点，尝试用它去“拔高”它左右两侧的邻居：如果 $lim[\text{neighbor}] < lim[\text{curr}] - mid$，说明邻居太矮了，违背了坡度限制，就将邻居强行拔高至 $lim[\text{curr}] - mid$，并把更新后的邻居重新推入队列。
    
4. 直到队列为空，说明所有点的高度要求都传递完毕，此时全图满足陡峭度不超过 $mid$。

**效果与局限：** 这种方法的逻辑非常直观，它完全模拟了“沙子从高处向低处自然滑落”的物理过程。由于使用了优先队列，它天然地按照高度从大到小的顺序处理，保证了每次更新都是最优且合法的。 但在 G2 版本中，由于数据规模 $\sum n \le 10^6$，二分查找需要调用约 30 次 `check` 函数。优先队列的单次时间复杂度为 $O(N \log N)$，并且存在极其庞大的常数开销（动态内存分配、树形结构调整、Cache 命中率低）。如果出题人构造了锯齿状的极端数组，优先队列会被卡满，从而导致 Time Limit Exceeded (TLE)。


#### 代码展示 ：
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
    int n,k;
    cin >> n >> k;
    vector<int> a(n);
    priority_queue<pair<int,int>> q;
    int mx = -1;
    for(int i = 0;i < n;i++) {
        cin >> a[i];
        q.push({a[i],i});
        int nxt = (n + 1) % n;
    }

    auto check = [&](int mid) -> bool {
        auto pq = q;
        vector<int> lim = a;
        while(!pq.empty()) {
            auto [v,i] = pq.top();
            //cout << v << " " << i << endl;
            pq.pop();
            if(lim[i] > v) continue;
            lim[i] = v;
            //
            int x = (i - 1 + n) % n;
            if(lim[x] < v - mid) {
                lim[x] = v - mid;
                pq.push({lim[x], x});
            }
            int y = (i + 1) % n;
            if(lim[y] < v - mid) {
                lim[y] = v - mid;
                pq.push({lim[y], y});
            }
        }
        //
        int sum = 0;
        for(int i = 0;i < n;i++) {
            sum += lim[i] - a[i];
            if(sum > k) return false;
        }
        
        // cout << mid << " : " << sum << endl;
        // for(auto& x : lim) cout << x << " ";
        // cout << endl;

        return sum <= k;
    };

    //
    int l = 0,r = 0;
    for(int i = 0;i < n;i++) {
        r = max(r,abs(a[i] - a[(i + 1) % n]));
    }
    int ans = r;
    while(l <= r) {
        int mid = (l + r) >> 1;
        if(check(mid)) {
            ans = mid;
            r = mid - 1;
        }   
        else l = mid + 1;
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





### 方法二：正反线性扫描 / DP 传递法

**逻辑解析：** 因为整个地形只有一条**环形的一维路径**，且高度传递的衰减代价是恒定的 $mid$，我们根本不需要复杂的图论算法来动态寻找“最高点”。 高度的传递只可能有两个绝对的方向：**顺时针（向右）**和**逆时针（向左）**。

1. **顺时针扫描**：只考虑高度向右传递的影响。遍历数组，如果左边的点传递过来的要求 $lim[i-1] - mid$ 比当前点 $lim[i]$ 还要高，就把 $lim[i]$ 更新。
    
2. **逆时针扫描**：只考虑高度向左传递的影响。从右向左遍历数组，执行类似的更新。
    
3. **为什么是 $2N$ 次循环？** 因为地形是一个**环**。如果有一座极高的山峰位于数组末尾 $N-1$ 处，它向右传递高度时，需要跨过边界影响到 $0, 1, 2 \dots$ 等位置。跑两圈（$2N$ 次）在数学上可以绝对保证，任何一个最高点的约束都能绕着环传透整座山脉。


#### 代码展示 ：
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
    int n, k;
    cin >> n >> k;
    vector<int> a(n);
    for(auto& x : a) cin >> x;

    auto check = [&](int mid) -> bool {
        vector<int> lim = a;
        
        for(int i = 0; i < 2 * n; i++) {
            int curr = (i >= n) ? i - n : i;
            int prev = (curr == 0) ? n - 1 : curr - 1;
            
            if(lim[prev] - mid > lim[curr]) {
                lim[curr] = lim[prev] - mid;
            }
        }
        
        for(int i = 2 * n - 1; i >= 0; i--) {
            int curr = (i >= n) ? i - n : i;
            int next = (curr == n - 1) ? 0 : curr + 1;
            
            if(lim[next] - mid > lim[curr]) {
                lim[curr] = lim[next] - mid;
            }
        }

        //
        int sum = 0;
        for(int i = 0; i < n; i++) {
            sum += lim[i] - a[i];
            if(sum > k) return false;
        }
        return true;
    };

    int l = 0,r = 0;
    for(int i = 0;i < n;i++) {
        r = max(r,abs(a[i] - a[(i + 1) % n]));
    }
    int ans = r;
    while(l <= r) {
        int mid = (l + r) >> 1;
        if(check(mid)) {
            ans = mid;
            r = mid - 1;
        }   
        else l = mid + 1;
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