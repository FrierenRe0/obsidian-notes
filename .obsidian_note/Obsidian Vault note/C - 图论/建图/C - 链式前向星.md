## c++代码：

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


链式前向星 -> 个人代码展示：

//这里我给定我这里给你：
// n 个节点
// m 条边 -> 这里有向和无向逻辑一样，无向就是一条边的两方向我都要加一次；
// 这里我还给出每一条边的带权值

//因为这里我在建立链式前向星时，对于空间开销的上限是一个固定的，这里开一个静态数组即可
const int N = 1e5 + 1;
const int M = 2e5 + 1;
//编号我这里 -> based -> 1; 
int head[N + 1];       //   head: 下标表示i节点编号，对应head[i]为i节点的头号位置的边编号
int nxt[2 * M + 1];    //   next: 下标表示i边编号，对应next[i]为i边的下一位边的编号
int to[2 * M + 1];     //     to: 下标表示i边编号，对应to[i]为i边所链接去往的节点编号
int weight[2 * M + 1]; // weight: 下标表示i边编号，对应to[i]为i边所对应的权值

int cnt = 1;           //    cnt: 这里cnt用于边的编号；


// 逻辑上等价于链表的头插法
unordered_map<int,pair<int,int>> jilu;

void addedges(int u,int v,int w) {
    weight[cnt] = w;
    to[cnt] = v;
    jilu[cnt] = make_pair(u,v);
    //数组模拟链表头插
    nxt[cnt] = head[u];
    head[u] = cnt++;
}

void init_clear() {
    cnt = 1;
    fill(head, head + N + 1, 0);
}

void write(int n) {
    for(int i = 1;i <= n;i++) {
        cout << "node-" << i << " : ";

        int node_e = head[i];
        while(node_e) {
            cout << "( ";
            cout << node_e << "," << to[node_e];
            cout << " ) ";
            node_e = nxt[node_e];
            if(node_e) cout << "-> ";
        }
        cout << endl;
    }
}

void solve() { 
    int n,m;
    cin >> n >> m;
    for(int i = 0;i < m;i++) {
        int u,v,val;
        cin >> u >> v >> val;
        addedges(u,v,val);
        addedges(v,u,val);
    }
    
    //这我把每条边对应的编号输出方便观察：
    cout << "egdes : " << endl;
    for(int i = 1;i < cnt;i++) {
        cout << i << " : " << "( " << jilu[i].first << "," << jilu[i].second << " )";
        cout << endl;
    }
    
    cout << endl;

    //这里遍历所有每个节点所链接的边以及对应每条边所去往的节点
    write(n);
    
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


### 三种常用存图方式对比：

|**存储方式**|**空间复杂度**|**遍历节点 u 所有出边**|**检查是否存在边 (u,v)**|**适用场景 / 特点**|
|---|---|---|---|---|
|**邻接矩阵** (`g[u][v]`)|$\mathcal{O}(N^2)$|$\mathcal{O}(N)$|$\mathcal{O}(1)$|稠密图（$M \approx N^2$）、点数较小（$N \le 2500$）|
|**`vector` 邻接表**|$\mathcal{O}(N + M)$|$\mathcal{O}(\text{deg}(u))$|$\mathcal{O}(\text{deg}(u))$|通用性最高，代码易读，绝大多数场景首选|
|**链式前向星**|$\mathcal{O}(N + M)$|$\mathcal{O}(\text{deg}(u))$|$\mathcal{O}(\text{deg}(u))$|稀疏图、高并发/超限时图论竞赛题、极致性能需求|