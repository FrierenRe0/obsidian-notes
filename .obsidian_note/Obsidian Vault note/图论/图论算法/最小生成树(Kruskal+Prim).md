# 一、最小生成树是干嘛的？

给你一个**无向带权图**

选出 **n-1 条边**，把所有点连起来，**总权值最小**，且**没有环**

这就是 **最小生成树 MST**

---

# Kruskal 算法  -> 从边的角度找最小生成树

## 核心思想：**从小到大选边，不环就加！**

步骤：
1. 把所有边**按权值从小到大排序**
2. 依次拿边
3. 用**并查集**判断两点是否连通
4. **不连通就加入生成树**
5. 选够 n-1 条边结束

## Kruskal 算法代码

```c++
#include <bits/stdc++.h>
using namespace std;

// 边结构体：from, to, weight
struct Edge {
    int u, v, w;
    // 排序：从小到大
    bool operator<(const Edge& other) const {
        return w < other.w;
    }
};

// 并查集（判断是否连通）
vector<int> fa;

int find(int x) {
    if (fa[x] != x) fa[x] = find(fa[x]);
    return fa[x];
}
void Union(int a,int b) {
	fa[a] = b;
}
// Kruskal 最小生成树，返回总权值
int kruskal(int n, vector<Edge>& edges) {
	//排特例
	if(n <= 1) return 0;

    sort(edges.begin(), edges.end()); // 边从小到大排序
    fa.resize(n);
    for (int i = 0; i < n; i++) fa[i] = i; // 初始化并查集

    int sum = 0;   // 总权值
    int cnt = 0;   // 已选边数

    for (auto& e : edges) {
        int u = e.u, v = e.v, w = e.w;
        int fu = find(u), fv = find(v);

        if (fu != fv) {        // 不连通 → 加入
            Union(fu,fv);
            sum += w;
            cnt++;
            if (cnt == n-1) break; // 够 n-1 条边就结束
        }
    }
    return sum;
}

```

**Kruskal 只负责在原图的连通分量内部构造最小生成树**：
1. 如果**原图本身是连通图**：最终所有节点一定连通，选出恰好 (n-1) 条边，构成整图的最小生成树。
2. 如果**原图是不连通图（存在多个连通分量）**：算法只会给**每个连通分量**各自生成一棵最小生成森林，**整体无法全部连通**。

---

# Prim 算法  -> 从点的角度找最小生成树

## 核心思想:
****从一个起点开始，每次选「离当前树最近的点」加入树，一点点扩大，直到连完所有点。****

你可以把它理解成：
**像水波纹一样，从一个点慢慢扩散到全图，每次只走最近的路！**

## 超简单步骤（一看就懂）

1. 选**任意一个起点**（比如 0 号点），把它放进 **树集合**
2. 找一条**最短的边**：

    - 一端 **在树里**
    - 一端 **不在树里**

3. 把这个**不在树里的点**拉进树
4. 重复 **n 次**（因为要连 n 个点）
5. 结束 → 得到最小生成树

# 四、Prim 算法代码（基础暴力贪心）

```c++
#include <bits/stdc++.h>
using namespace std;

// graph 是邻接矩阵：graph[x][y] 表示 x到y 的边权
int prim(int n, vector<vector<int>>& graph) {//这里graph为邻接矩阵
    // 1. 定义两个关键数组
    vector<int> dis(n, INT_MAX);    // 每个点 到 生成树 的最短距离
    vector<bool> inTree(n, false);   // 点是否已经在树里
    dis[0] = 0;                     // 起点 0 到树的距离为 0
    int totalWeight = 0;            // 最小生成树总权重

    // 2. 必须选 n 个点
    for (int i = 0; i < n; i++) {
        // --------------------
        // 第一步：找 离树最近、且不在树里 的点 u
        // --------------------
        int u = -1;
        int minDist = INT_MAX;
        for (int j = 0; j < n; j++) {
            if (!inTree[j] && dis[j] < minDist) {
                minDist = dis[j];
                u = j;
            }
        }

        // --------------------
        // 第二步：把 u 加入树
        // --------------------
        inTree[u] = true;
        totalWeight += minDist;

        // --------------------
        // 第三步：用 u 更新邻居点到树的距离
        // --------------------
        for (int v = 0; v < n; v++) {
            if (!inTree[v] && graph[u][v] < dis[v]) {
                dis[v] = graph[u][v];
            }
        }
    }
    return totalWeight;
}

//样例主函数
//int main() {
//	int n = 4;
//	vector<vector<int>> g = {
//		{0,3,INT_MAX,INT_MAX},
//		{3,0,1,4},
//		{INT_MAX,1,0,1},
//		{INT_MAX,4,1,0}
//	};
//	cout << prim(n, g) << endl;
//}
```

## 优化：堆维护
prim最小生成树用堆维护，基本就是dijk算法
区别： prim中dis维护的是该节点距离生成树的最短距离 ，dijk中dis维护的是该节点距离起点的最短距离
```c++
int prim(int n, vector<vector<pair<int,int>>>& graph) { //这里graph为邻接表
    // 1. 定义两个关键数组
    vector<int> dis(n, INT_MAX);    // 每个点 到 生成树 的最短距离
    priority_queue<pair<int,int>,vector<pair<int,int>>,greater<>> pq;
    dis[0] = 0;                     // 起点 0 到树的距离为 0
    pq.push({0,0});
    int total = 0;            // 最小生成树总权重

    // 2. 必须选 n 个点
    while(!pq.empty()) {
        auto [dis_x,x] = pq.top();
        pq.pop();
		if(dis_x > dis[x]) continue;
        // --------------------
        // 第二步：把 x 加入树
        // --------------------
        dis[x] = 0;
        total += dis_x;

        // --------------------
        // 第三步：用 x 更新邻居点到树的距离
        // --------------------
        for(auto &[next,dis_next] : graph[x]) {
	        if(dis_next < dis[next]) {
		        dis[next] = dis_next;
		        pq.push({dis_next,next});
	        }
        }
    }
    return totalWeight;
}
```
