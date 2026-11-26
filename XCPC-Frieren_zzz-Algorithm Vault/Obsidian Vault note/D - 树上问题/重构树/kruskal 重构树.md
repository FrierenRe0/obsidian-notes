这段代码实现的是 **Kruskal 重构树（Kruskal Reconstruction Tree）**。它将经典求最小生成树的 Kruskal 算法扩展，把无向图转化成一棵节点数最多为 $2n - 1$ 的森林，从而将“图上路径瓶颈”问题转化为“树上 LCA”问题。

**这里我增加一些我个人对于kruskal重构树的初步了解以及应用场景和优势：**

- 首先kruskal重构树是一个用于解决“图上路径瓶颈”问题 -> 这个是kruskal重构树的典型问题标志
	
- 其次我们对于这种“图上路径瓶颈”问题，通过将边看成虚拟节点重构树的方式巧妙优化，效率上将单次的查询降至`logm`，可以完成多次询问；
	
- 时间上的开销分析：
		
	- 在 重构树+建立st表 的时间复杂度级为 o(`mlogm + nlogn`)
		  
	- 同时对于多次查询 (这里我就设定为查询`a`次) 时间复杂度级为 o(`alogm`) -> (这里可以做到`loga`级别是因为转成了`LCA`问题，采用了st表二进制跳跃)；
	
- 下面是对于kruskal重构树的具体分析；


## 需要的前置知识：

- 并查集
	
- 最小生成树 - kruskal
	
- 最近公共祖先 - LCA


## 核心性质

重构树将原图的**边权转换为新树上的点权**，具备以下三大核心性质：

1. **结构构成**：
    
    - **叶子节点**（编号 $1 \sim n$）：对应原图中的 $n$ 个真实顶点，点权为 $0$。
        
    - **非叶节点**（编号 $n+1 \sim 2n-1$）：合并连通块时创建的虚拟父节点，点权等于当时合并那条边的边权（代码中的 `weight[node]`）。
        
2. **小根堆性质**：
    
    - 因为边是按权值升序排序后合并的，从任意叶子节点向上到根节点的路径上，节点的 `weight` 值单调递增。
        
3. **瓶颈路径等价性**：
    
    - 原图中任意两点 $u, v$ 之间所有可行路径中“最大边权”的最小值，恰好等于它们在重构树上最近公共祖先的点权，即 **`weight[Lca(u, v)]`**。
        

## 代码流程与结构解析

结合你代码中的函数，算法具体分为 4 个关键阶段：

```

 原图边集                 Kruskal重构树                   倍增LCA查询
[Edges] --(排序+并查集)-->  (新节点+连边) --(DFS预处理)--> [st倍增表] --> 输出

weight[Lca(u,v)]
```

### 1. 边的排序与虚拟节点创建 (`rebuildkruskalTree`)

- 按照边权 `w` 升序排序。
    
- 遍历每条边 $(u, v, w)$，用 `find(u)` 和 `find(v)` 检查连通性。
    
- 若不连通，新建虚拟节点 `node`（从 $n+1$ 开始），将 `weight[node]` 赋为 $w$。
    
- 将 $u, v$ 当前连通块的根节点的父指针 `fa` 都指向 `node`，并通过 `addedge` 从 `node` 向它们连有向边，完成重构树子树的构建。


### 2. 链式前向星建树 (`addedge`)

- 代码采用链式前向星记录重构树的父子关系。由于每次合并均由父节点 `node` 连向子节点 $u, v$，因此构造成了一棵自顶向下的有向树（或森林）。


### 3. 倍增预处理与 LCA (`dfs` + `Lca`)

- 对所有重构树的根节点（满足 `fa[i] == i`）执行 `dfs(i, 0)`，计算树深度 `deep` 并初始化倍增数组 `st[20][N]`。
    
- `Lca(x, y)` 利用 $O(\log n)$ 的倍增跳跃算法找到 $x$ 和 $y$ 在重构树上的公共祖先。
    

### 4. 在线查询 (`solve`)

- 对于查询 $(u, v)$：
    
    - 若 `find(u) != find(v)`，说明原图中两点不连通，直接输出 `"impossible"`。
        
    - 若连通，输出 `weight[Lca(u, v)]` 即为两点间瓶颈边权。
        


## 5.补充：代码中`node` 和 `tot_edge` 两个变量表示的含义：

这两个变量分别用来维护“点”**和**“边”两个完全不同的维度：

|**变量名**|**维护的对象**|**对应索引的数组**|**变化规律**|
|---|---|---|---|
|**`node`**|**重构树的“节点编号”**|`weight[node]`, `fa[node]`, `deep[node]`, `st[k][node]`|初始为 $n$。每合并两个连通块，新建 **1 个** 节点（$n+1, n+2, \dots$），`node` 加 1。|
|**`tot_edge`**|**链式前向星的“边内存索引”**|`to[tot_edge]`, `nxt[tot_edge]`|初始为 1。每调用一次 `addedge` 添加 **1 条** 连向子节点的边，`tot_edge` 加 1。|

### 为什么链式前向星下不能把它们合并成一个？

因为**增加的节奏不同**：

1. **新建节点（`node`）**：合并一次产生 **1 个** 新节点（例如节点 $n+1$）。
    
2. **添加边（`tot_edge`）**：重构树里这个新节点需要分别连向两个子节点 $u$ 和 $v$，因此需要调用两次 `addedge`，产生 **2 条** 有向边。
    

#### 举个例子：

假设初始有 $n = 3$ 个点：

1. 第一次合并 $1$ 和 $2$：
    
    - 生成新节点 $n+1 = 4$（`node` 变成 4）；
        
    - 添加边 $4 \to 1$，占据前向星第 1 存储位（`tot_edge` 变成 2）；
        
    - 添加边 $4 \to 2$，占据前向星第 2 存储位（`tot_edge` 变成 3）。
        

- node :  一个图中的节点计数；
- tot_edge ：添加边的记号，和链式前向星的编号记号等价；


## 代码展示：

对应模板题 ：[P2245 星际导航 - 洛谷](https://www.luogu.com.cn/problem/P2245)

```
#include <bits/stdc++.h>
using namespace std;
#define int long long
#define endl '\n'
#define ctz __builtin_ctzll         // 二进制表示中末尾连续0的个数
#define clz __builtin_clzll         // 二进制表示中先导0的个数
#define count1 __builtin_popcountll // 二进制表示中1的个数
const int mod = 1e9 + 7;
const int MOD = 998244353;
const int N = 2e5 + 7; // 若原图 n 最大 2e5，重构树点数最多 2n-1，此处需开到 4e5+7


struct edge {
    int u, v, w;
    bool operator<(const edge& other) const {
        return w < other.w;
    }
};

// 链式前向星 -> 图
int head[N];
int nxt[N];
int to[N];
int weight[N];
int node_edge; // 用于链式前向星 ：边的编号
int node;      // 用于重构树节点的编号

// 倍增与并查集
int fa[N];
int deep[N];
int st[20][N];

// 边集
vector<edge> edges;

int find(int x) {
    if (fa[x] == x) return x;
    return fa[x] = find(fa[x]);
}

void addedge(int u, int v) {
    to[node_edge] = v;
    nxt[node_edge] = head[u];
    head[u] = node_edge++; 
} 

void dfs(int x, int p) {
    deep[x] = deep[p] + 1;
    st[0][x] = p;
    for (int i = 1; i < 20; i++) {
        st[i][x] = st[i - 1][st[i - 1][x]];
    }
    for (int e = head[x]; e; e = nxt[e]) {
        dfs(to[e], x);
    }
}

int Lca(int x, int y) {
    if (deep[x] > deep[y]) swap(x, y);

    // 将 x 和 y 调到同一深度
    for (int i = 19; i >= 0; i--) {
        if (deep[st[i][y]] >= deep[x]) {
            y = st[i][y];
        } 
    }
    if (x == y) return x;

    for (int i = 19; i >= 0; i--) {
        int dx = st[i][x];
        int dy = st[i][y];
        if (dx != dy) {
            x = dx;
            y = dy;
        }
    }
    return st[0][x];
}

void rebuildkruskalTree() {
    sort(edges.begin(), edges.end());
    for (auto e : edges) {
        int u = e.u;
        int v = e.v;
        int w = e.w;
        u = find(u), v = find(v);
        if (u == v) continue;
        weight[++node] = w;
        fa[u] = fa[v] = node;
        fa[node] = node; 

        addedge(node, u);
        addedge(node, v);
    }
}

void solve() { 
    int n, m;
    cin >> n >> m;
    edges.resize(m);
    for (int i = 0; i < m; i++) {
        cin >> edges[i].u >> edges[i].v >> edges[i].w;
    }

    auto init = [&]() {
        node_edge = 1; 
        for (int i = 1; i <= 2 * n; i++) {
            head[i] = 0;
            fa[i] = i;
        }
    };
    init();

    node = n; 
    rebuildkruskalTree();   
    
    for (int i = 1; i <= node; i++) {
        if (fa[i] == i) {
            dfs(i, 0);
        }
    } 

    int q;
    cin >> q;
    while (q--) {
        int u, v;
        cin >> u >> v;
        if (find(u) != find(v)) {
            cout << "impossible" << endl;
        } else {
            cout << weight[Lca(u, v)] << endl;
        }
    }
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




## 复杂度与应用场景

| **维度**    | **复杂度**                  | **说明**                                         |
| --------- | ------------------------ | ---------------------------------------------- |
| **时间复杂度** | $O(m \log m + q \log n)$ | 瓶颈在于边的排序 $O(m \log m)$ 与每次查询的 LCA $O(\log n)$。 |
| **空间复杂度** | $O(n \log n + m)$        | 需要开辟 $2n$ 级别的节点数组及 $20 \times 2n$ 的倍增数组。       |

**常见考查场景：**

1. **两点连通路径瓶颈**：求 $u \to v$ 所有路径中“最大边权最小值”（升序建树）或“最小边权最大值”（降序建树）。
    
2. **子树可达性查询**：查询从点 $u$ 出发，只经过边权 $\le W$ 的边能到达的所有点。通过倍增向上找到深度最浅且 `weight[ancestor] <= W` 的祖先节点，该祖先子树内的所有叶子节点即为答案。