## 相关链接：
- 题卷查看：[contest-57488-zh.pdf](https://uploadfiles.nowcoder.com/files/20260805/999991351_1785901556367/contest-57488-zh.pdf)
- 题目链接：[G-Game on a Graph_2026牛客暑期多校训练营6](https://ac.nowcoder.com/acm/contest/133881/G)

## 评价：
变形题：拓扑排序 （本质：多源 BFS）
- 变形的地方：加入学习队列的前提条件发生变化 : 对于当前节点，如果特殊节点遍历到该点的次数 $>= 2$ 该节点可以入队

## 题目大意：

![](../../../../../截图文件/Pasted%20image%2020260807151359.png)

### 题目解析：

在一个最大度数不超过 $3$ 的无向连通图中，有 $k$ 个特殊节点（目标点）。Alice 和 Bob 轮流行动，Alice 先手移动，Bob 后手删边。若 Alice 踏入特殊节点则 Alice 获胜；若 Alice 无路可走则 Bob 获胜。求 Alice **初始站立时能必胜**的所有非特殊节点。

## 思路：

由于 Alice **先手**，游戏存在两种不同的获胜机制：

1. **初始站立即一步获胜（起点优势）**：
    
    - 如果节点 $u$ 至少有 **1** 条边连接到“安全区域”（上一步已知 Alice 必胜的节点），且 Alice **初始就站在 $u$**，她可以在第 1 回合直接走过去获胜。此时 Bob 还没来得及删边，因此 **$u$ 是一个必胜起始点（Winning vertex）**。
    
2. **作为途径中转点（博弈对抗）**：
    
    - 如果 Alice 不是初始站在 $u$，而是游戏途中**经过** $u$，情况会截然不同：当 Alice 到达 $u$ 时，下一步轮到 Bob 删边。
        
    - 如果 $u$ 只有 1 条边通往安全区域，Bob 删掉这条边后 Alice 就会被困死。
        
    - 因此， $u$ 必须拥有 **至少 2 条边** 通往上一层的安全区域。这样哪怕 Bob 删掉 1 条边，Alice 依然可以走剩下的那条边。只有满足条件的 $u$ 才能作为**安全中转点**，继续向外扩展。
    

### 代码逻辑分析与运行流程

代码巧妙地利用了**分层多源 BFS** 来模拟这个倒推过程：

1. **初始化**：
    
    - 将 $k$ 个特殊节点放入 BFS 队列 `q`，标记 `vis[special] = true`。它们是距离为 $0$ 的终点。
    
2. **按层扩展**：
    
    - 使用 `len = q.size()` 严格控制逐层扩展。
        
    - 对于当前层（已确认的安全中转点）扩展到的邻居 `next`：
        
        - **第一次遇到（`cnt[next] == 0`）**：说明 $next$ 拥有了 **1 条** 通向上一层安全区域的边。Alice 如果初始站在 $next$，可以先手一步迈入获胜。因此将 $next$ 加入 `ans`。但此时它还不能作为中转点入队。
        
        - **第二次遇到（`cnt[next] > 0`）**：说明 $next$ 拥有了 **至少 2 条** 通向上一层安全区域的边，能够抵抗 Bob 删去 1 条边。此时将 $next$ 可以升级为特殊节点，标记 `vis[next] = true` 并放入队列 `q`，供下一层继续扩展。


## 复杂度分析

- **时间复杂度**： $\mathcal{O}(N + M)$
    
    图中的每个节点和每条边在多源 BFS 中最多被遍历常数次，运行效率极高。
    
- **空间复杂度**： $\mathcal{O}(N + M)$
    
    主要空间开销为邻接表 `graph` 以及辅助数组 `vis`、`cnt` 和队列 `q`。



## 代码如下：

```c++
#include <bits/stdc++.h>
using namespace std;
#define int long long
const int mod = 1e9 + 7;
const int MOD = 998244353;
const int N = 2e5 + 7;

void solve() { 
    int n,m,k;
    cin >> n >> m >> k;
    vector<vector<int>> graph(n + 1);
    for(int i = 0;i < m;i++) {
        int u,v;
        cin >> u >> v;
        graph[u].push_back(v);
        graph[v].push_back(u);
    }
    vector<bool> vis(n + 1,false);
    vector<int> cnt(n + 1);
    queue<int> q;
    for(int i = 0;i < k;i++) {
        int a;
        cin >> a;
        q.push(a);
        vis[a] = true;
    }
    vector<int> ans;
    while(!q.empty()) {
        int len = q.size();
        while(len--) {
            auto x = q.front();
            q.pop();
            for(auto& next : graph[x]) {
                if(vis[next]) continue;
        
                if(!cnt[next]) {
                    ans.push_back(next);
                    cnt[next]++;
                }
                else {
                    cnt[next]++;
                    q.push(next);
                    vis[next] = true;
                }
            }
        }
    }
    cout << ans.size() << endl;
    for(auto& x : ans) cout << x << " ";
    cout << endl;
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