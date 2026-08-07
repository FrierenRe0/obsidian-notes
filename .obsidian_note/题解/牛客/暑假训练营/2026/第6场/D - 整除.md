## 相关链接：
- 题卷查看：[contest-57488-zh.pdf](https://uploadfiles.nowcoder.com/files/20260805/999991351_1785901556367/contest-57488-zh.pdf)
- 题目链接：[D-Divisibility_2026牛客暑期多校训练营6](https://ac.nowcoder.com/acm/contest/133881/D)

## 评价：
数学思维题 + 图论：**二分图 BFS**（求到每个点的最短偶数长度路径与奇数长度路径）实现

## 题目大意：

![](../../../../../截图文件/Pasted%20image%2020260807152923.png)

### 题目解析：

给定一个包含 $n$ 个顶点、$m$ 条边的无向图（可能包含自环和重边）和一个正整数 $k$。

对于每个顶点 $u$，求满足以下条件的**最小非负整数路径长度 $d$**：

1. $d$ 是 $k$ 的倍数（即 $k \mid d$）；
    
2. 存在一条从顶点 $1$ 到顶点 $u$ 且长度恰好为 $d$ 的路径（路径可重复经过点和边）。

如果不存在这样的 $d$，则 $f(u) = -1$。求所有 $1 \le u \le n$ 的 $f(u)$。

## 思路：

#### 1. 路径长度的扩展特性（奇偶性）

在无向图中，如果我们找到了一条从 $1$ 到 $u$ 长度为 $L$ 的路径，只要 $u$ 存在邻接边，我们就可以通过在邻接边上来回走把路径长度变为 $L + 2, L + 4, L + 6, \dots$。

这意味着：

- **偶数长度的路径**可以延伸为任意大于等于初始值的**偶数**；
    
- **奇数长度的路径**可以延伸为任意大于等于初始值的**奇数**。


因此，我们需要用 **拆点 BFS** 分别求出从 $1$ 到每个点 $u$ 的：

- `dist[u][0]`：最短**偶数**路径长度；
    
- `dist[u][1]`：最短**奇数**路径长度。

### 实现过程：

- **BFS 求解奇偶最短路**：
    
    - 使用 BFS 拆点求出 $dist[u][0]$（从 1 到 $u$ 的最短偶数长路径）和 $dist[u][1]$（从 1 到 $u$ 的最短奇数长路径）。
    
- **奇偶性与 $k$ 的倍数推导**：
    
    - 我们希望找一个 $d = c \cdot k \ge \text{dist}$，且 $d \pmod 2 = \text{parity}$。
        
    - **情况 1： $k$ 为偶数**
        
        - $c \cdot k$ 一定是偶数。因此只能匹配 $dist[u][0]$。
            
        - 最小的 $c$ 为 $\lceil dist[u][0] / k \rceil$。答案即为 $c \cdot k$。
        
    - **情况 2： $k$ 为奇数**
        
        - 若匹配 $dist[u][0]$（偶数路径），则 $c \cdot k$ 必须是偶数 $\implies c$ 必须是偶数。
            
            - $c = \lceil dist[u][0] / k \rceil$，若 $c$ 为奇数则 $c \leftarrow c + 1$。
            
        - 若匹配 $dist[u][1]$（奇数路径），则 $c \cdot k$ 必须是奇数 $\implies c$ 必须是奇数。
            
            - $c = \lceil dist[u][1] / k \rceil$，若 $c$ 为偶数则 $c \leftarrow c + 1$。
            
        - 取这两者计算出的 $c \cdot k$ 的最小值。
		 
	- **情况3：其他形式这表示对于该点找不到被k整除的情况，输出-1；**


## 代码如下：

```c++
#include<bits/stdc++.h>
using namespace std;
const long long INF = 1e18;

void solve() {
    int n, m;
    long long k;
    cin >> n >> m >> k;

    vector<vector<int>> graph(n + 1);
    for (int i = 0; i < m; ++i) {
        int u, v;
        cin >> u >> v;
        graph[u].push_back(v);
        graph[v].push_back(u); 
    }

    vector<vector<long long>> dist(n + 1, vector<long long>(2, INF));
    queue<pair<int, int>> q;

    dist[1][0] = 0;
    q.push({1, 0});

    while (!q.empty()) {
        auto [u, p] = q.front();
        q.pop();

        for (int v : graph[u]) {
            int np = p ^ 1;
            if (dist[v][np] == INF) {
                dist[v][np] = dist[u][p] + 1;
                q.push({v, np});
            }
        }
    }

    for (int u = 1; u <= n; ++u) {
        long long ans = -1;

        if (k % 2 == 0) {
            if (dist[u][0] != INF) {
                long long c = (dist[u][0] + k - 1) / k;
                ans = c * k;
            }
        } else {
            long long min_c = INF;

          
            if (dist[u][0] != INF) {
                long long c = (dist[u][0] + k - 1) / k;
                if (c % 2 != 0) c++; 
                min_c = min(min_c, c);
            }

            if (dist[u][1] != INF) {
                long long c = (dist[u][1] + k - 1) / k;
                if (c % 2 == 0) c++; 
                min_c = min(min_c, c);
            }

            if (min_c != INF) {
                ans = min_c * k;
            }
        }

        cout << ans << " ";
    }
    cout << endl;
}

int main() {
    ios_base::sync_with_stdio(false);
    cin.tie(NULL);
    int t = 1;
    cin >> t;
    while (t--) {
        solve();
    }
    return 0;
}
```