![](../../../../../截图文件/statement_20018.pdf)
## 题目大意：

这道题的核心是利用**拓扑排序**结合**字典序最小/贪心策略**来求解。

#### 1. 条件转化（建图）
对于给定的限制 $p_{q_{i,1}} < p_{q_{i,2}} < \dots < p_{q_{i,k}}$，我们可以转化为有向边：
由 $q_{i,j}$ 向 $q_{i,j+1}$ 连一条有向边 $(q_{i,j} \to q_{i,j+1})$，表示在最终排序中，$q_{i,j}$ 的**相对大小值**（或在拓扑序列中的位置）必须小于 $q_{i,j+1}$，像这种需要满足优先顺序排列的情况可以用拓扑排序。

#### 2. 最小化逆序对的贪心策略
我们要让构造的排列 $p$ 的逆序对尽量少：
- 逆序对是指 $x < y$ 且 $p_x > p_y$。
- 如果我们让拓扑序列中**位置靠前的下标分配较小的值 $1, 2, \dots, n$**，那么对于任意两个下标 $x < y$，为了**避免产生逆序对（即希望 $p_x < p_y$）**，我们应该**尽量让下标较小的 $x$ 在拓扑序中排在前面**，则就是构造最小字典序拓排序。
- 因此，只要我们在求解拓扑排序时，**优先选择下标小的节点**（即使用小根堆 `priority_queue<int, vector<int>, greater<int>>` ，就能保证在满足所有边依赖的前提下，下标较小的节点尽可能拿到更小的值 $p_x$，从而最大程度减少逆序对。

#### 3. 算法步骤
1. **建图与统计入度**：根据输入的所有限制，建立有向图 `graph` 并维护入度数组 `in`。
2. **拓扑排序**：
   - 将所有入度为 0 的节点放入小根堆 `pq` 中。
   - 依次取出堆顶元素 $u$（即当前入度为 0 且下标最小的节点），并且从`val == 1`开始哈希映射赋值。
   - 遍历 $u$ 的邻接点 $v$，将 $in[v]$ 减 1；若 $in[v] == 0$，则压入 `pq`。
3. **环检测与答案构造**：
   - 如果拓扑序列长度小于 $n$，说明图中存在环，无解，输出 `-1`。
   - 否则，假设拓扑序列为 $T_1, T_2, \dots, T_n$，索引hash表`hash[T] == p`,最终输出 $p_1, p_2, \dots, p_n$。

---

### C++ AC 代码

```cpp
#include <bits/stdc++.h>
using namespace std;
#define int long long
#define ctz __builtin_ctzll         // 二进制表示中末尾连续0的个数
#define clz __builtin_clzll         // 二进驻表示中先导0的个数
#define count1 __builtin_popcountll // 二进制表示中1的个数
const int mod = 1e9 + 7;
const int MOD = 998244353;
const int N = 2e5 + 7;

void solve()
{
    int n, m;
    cin >> n >> m;
    // idx -> 接收数量;
    vector<vector<int>> graph(n + 1);
    vector<int> in(n + 1);
    int ci_mi = INT_MAX;
    for (int i = 0; i < m; i++)
    {
        int l, r;
        cin >> l >> r;
        int before_idx;
        for (int j = 0; j < r - l + 1; j++)
        {
            int idx;
            cin >> idx;
            if (j)
            {
                in[idx]++;
                graph[before_idx].push_back(idx);
            }
            before_idx = idx;
        }
    }
    
    //
    priority_queue<int, vector<int>, greater<>> pq;
    for (int i = 1; i <= n; i++)
    {
        if (!in[i]) pq.push(i);
    }
  
    //
    int val = 1;
    unordered_map<int, int> mp;
    // 出队数量
    int cnt = 0;
    while (!pq.empty())
    {
        auto x = pq.top();
        pq.pop();
        mp[x] = val++;
        cnt++;
        for (auto &next : graph[x])
        {
            in[next]--;
            if (!in[next]) pq.push(next);
        }
    }
    
    //
    if (cnt < n) cout << -1 << endl;
    else
    {
        for (int i = 1; i <= n; i++)
        {
            cout << mp[i] << " ";
        }
        cout << endl;
    }
    return;
}

signed main()
{
    ios::sync_with_stdio(false);
    cin.tie(nullptr);
    cout.tie(0);
    int T = 1;
    cin >> T;
    while (T--)
    {
        solve();
    }
    return 0;
}