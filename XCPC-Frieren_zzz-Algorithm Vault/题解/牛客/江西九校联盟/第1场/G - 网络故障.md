## 相关链接：
- 题卷查看：[第1场题卷](../题卷/main-zh.pdf)
- 题目链接：[G-网络故障_2026江西八校联盟第一场联赛](https://ac.nowcoder.com/acm/contest/138533/G)

## 评价：
并查集  —> 逆向思考

### 这种多种查询，操作问题：一般正向思考操作存在超时问题，如果合理优化完成多次查询操作？
类似问题：
- 一些单向并查集删除
- 二维矩阵覆盖
### 思考：
如果正向思考存在大量重复操作，如果去规避这样的重复操作？
现在想象一下对于某一个当前查询操作，再结果上的输出取决于他后续的操作情况，而前者的操作会被后者的操作情况给覆盖，这样就避免了前面大量的无效重复重复操作，实现了优化；


## 题目大意：

![](../../../../截图文件/Pasted%20image%2020260818160104.png)

## 题目解析：

给定一个包含 $n$ 个点和 $m$ 条无向边的图，并按顺序执行 $q$ 次操作。操作分为两种：

1. **删除指定边 $(u, v)$**：将图中顶点 $u$ 与顶点 $v$ 之间的连边移除。
    
2. **查询连通块大小**：查询顶点 $u$ 当前所在连通块中包含的顶点数量。


## 思路：

由于在并查集（DSU）中，**合并操作（Merge）高效**，而**拆分/撤销操作（Split）成本高**，直接按时间正序处理边被删除的过程非常困难，时间复杂度大。

我们可以采用 **逆序处理（离线算法 Offline Processing）** 的技巧：
这里才用逆序处理就如我上方所说：
- 每一个当前查询操作结果上的输出取决于他后续的操作情况，而前者的操作会被后者的操作情况给覆盖，这样就避免了前面大量的无效重复重复操作；
- 合并操作（Merge）高效，从最后状态到当前状态过程是一个并查集merge -> 高效；

1. **记录被删除的边**：
    
    在正序读取所有 $q$ 次操作时，将其中所有的“删除边”操作记录下来，并在初始图数据中标记这      些边已经被移除。
    
2. **构建最终基底图**：
    
    将所有**未被删除**的原始边合并（`merge`）加入并查集中，形成经历所有删除操作后的最终图连通状态。
    
3. **逆序还原并回答询问**：
    
    从最后一次操作（第 $q$ 个）向前倒序处理：
    
    - **遇到“删除边 $(u, v)$”操作**：在倒序视角下，删边等价于**重新添加该边**，直接调用 `merge(u, v)` 将两个顶点所在连通块合并。
        
    - **遇到“查询点 $u$”操作**：直接通过并查集查询 $u$ 根节点的 `sz[find(u)]`，即为此时 $u$ 所在的连通块大小，并将答案压入数组。
    
1. **输出答案**：
    
    最后将记录的答案数组反转（`reverse`），按原始正序依次输出即可 / 或者直接stack存储。
    

## 关键点与坑点 (Key Points & Caveats)

1. **无向图边去重/匹配**：
    
    - 在使用 `map<pair<int,int>, int>` 存储边时，注意原图输入和删除操作输入的边顺序可能不同（例如原图给 $(u, v)$，删边给 $(v, u)$）。建议统一保持 $u < v$ 的顺序，或者确保映射逻辑能双向匹配。
    
2. **逆序压栈与反转**：
    
    - 只有在遇到查询操作（Type 2）时才保存结果，逆序处理完毕后必须 `reverse` 答案数组。
    

## 复杂度分析 (Complexity Analysis)

- **时间复杂度**：$\mathcal{O}((m + q) \log m + (n + q) \cdot \alpha(n))$
    
    - Map 维护边标记的复杂度为 $\mathcal{O}((m + q) \log m)$（若改用 Hash 或者排好序的 vector 可降至更低）。
        
    - 并查集带路径压缩的合并与查询均摊单次复杂度为 $\mathcal{O}(\alpha(n))$。
        
    - 总时间复杂度在限时内可以高效通过。
    
- **空间复杂度**：$\mathcal{O}(n + m + q)$，用于保存并查集状态、边集信息及离线的操作序列。



## 代码如下：
```c++
#include <bits/stdc++.h>
using namespace std;
#define int long long
const int mod = 1e9 + 7;
const int N = 3e5 + 7;
vector<int> fa;
vector<int> sz;
void init(int n) {
    fa.resize(n + 1);
    sz.resize(n + 1);
    for(int i = 1;i <= n;i++) {
        fa[i] = i;
        sz[i] = 1;
    }
}
int find(int x) {
    if(fa[x] == x) return x;
    else return fa[x] = find(fa[x]);
}
void merge(int x,int y) {
    x = find(x),y = find(y);
    if(x == y) return;
    fa[y] = x;
    sz[x] += sz[y];
}
void solve() {
    int n,m,q;
    cin >> n >> m >> q;
    init(n);
    map<pair<int,int>,int> edges;
    for(int i = 0;i < m;i++) {
        int u,v;
        cin >> u >> v;
        edges[make_pair(u,v)] = 0;
    }
    vector<tuple<int,int,int>> a;
    a.reserve(q);
    for(int i = 0;i < q;i++) {
        int vis;
        cin >> vis;
        if(vis == 1) {
            int u,v;
            cin >> u >> v;
            a.emplace_back(vis,u,v);
            edges[{u,v}] = 1;
        }
        else {
            int u;
            cin >> u;
            a.emplace_back(vis,u,-1);
        }
    }
    
    //final graph
    for(auto [it,v] : edges) {
        if(v == 1) continue;
        int x = it.first;
        int y = it.second;
        merge(x,y);
    }
    
    vector<int> ans;
    for(int i = q - 1;i >= 0;i--) {
        auto res = a[i];
        int v = get<2>(res);
        if(v == -1) {
            int f = find(get<1>(res));
            ans.push_back(sz[f]);
        }
        else {
            merge(get<1>(res),get<2>(res));
        }
    }
    reverse(ans.begin(),ans.end());
    for(auto& x : ans) cout << x << endl;
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