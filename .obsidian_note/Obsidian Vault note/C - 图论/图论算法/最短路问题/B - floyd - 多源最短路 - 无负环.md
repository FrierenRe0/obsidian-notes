# 启发思考：寻找子问题
![配图](../../../../截图文件/Pasted%20image%2020260526164457.png)

# 二、递归怎么写：状态定义与状态转移方程

**1.定义 dfs(k,i,j) 表示从 i 到 j 的最短路长度，并且这条最短路的中间节点编号都 ≤k。注意中间节点不包含 i 和 j。**

**2.根据上面的讨论**：
不选 k，那么中间节点的编号都 ≤k−1，即 dfs(k,i,j)=dfs(k−1,i,j)。选 k，问题分解成从 i 到 k 的最短路，以及从 k 到 j 的最短路。由于这两条最短路的中间节点都不包含 k，所以中间节点编号都 ≤k−1，故得到 dfs(k,i,j)=dfs(k−1,i,k)+dfs(k−1,k,j)。这两种情况取最小值，就得到了 dfs(k,i,j)。

		写成式子就是：dfs(k,i,j)=min(dfs(k−1,i,j),dfs(k−1,i,k)+dfs(k−1,k,j))

**递归边界：dfs(−1,i,j)=`w[i][j]`。k=−1 表示 i 和 j 之间没有任何中间节点，此时最短路长度只能是连接 i 和 j 的边的边权，即 `w[i][j]`。如果没有连接 i 和 j 的边，则 `w[i][j]` = ∞。**

**递归入口：dfs(n−1,i,j)，表示从 i 到 j 的最短路长度。k=n−1 是因为本题节点编号为 0 到 n−1，任意最短路的中间节点编号都 ≤n−1。**


[1334. 阈值距离内邻居最少的城市 - 力扣（LeetCode）](https://leetcode.cn/problems/find-the-city-with-the-smallest-number-of-neighbors-at-a-threshold-distance/solutions/2525946/dai-ni-fa-ming-floyd-suan-fa-cong-ji-yi-m8s51/)

## 递归版floyd(超时)

时间复杂度：O(n^2*3^n)。
空间复杂度：O(n)。
```c++
int dfs(const vector<vector<int>>& g,int k,int i,int j) {
	if(k < 0) {
		return g[i][j];
	}
	return min(dfs(g,k - 1,i,j),dfs(g,k - 1,i,k) + dfs(g,k - 1,k,j));
}

int findTheCity(int n, vector<vector<int>>& edges, int distanceThreshold) {
	vector<vector<int>> g(n,vector<int>(n,INT_MAX >> 1));
	for(auto& e : edges) {
		int x = e[0],y = e[1],w = e[2];
		g[x][y] = g[y][x] = w;
	}
	int ans = 0;
	int min_cnt = n;
	for(int i = 0;i < n;i++) {
		int cnt = 0;
		for(int j = 0;j < n;j++) {
			if(j != i && dfs(g,n - 1;i,j) <= distanceThreshold) {
				cnt++;
			}
		}
		if(cnt <= min_cnt) {
			min_cnt = cnt;
			ans = i;
		}
	}
	return ans;
}
```

**超时原因**：
在递归计算中，`dfs[k][i][j]`在每个节点计算和其他节点的最短路中的整个过程中，不同的节点可能会对`dfs[k][i][j]`这个状态计算重复多次；
这里就可以用f数组来储存对应`dfs[k][i][j]`的状态情况；
(同时这里的`f[k][i][j]`和`dfs[k][i][j]`的定义相同,都是表示在i到j节点中，中间节点的编号<= k)

**注意**：
但是这里对于状态转移方程：`f[k][i][j]=min(f[k−1][i][j],f[k−1][i][k]+f[k−1][k][j])`

**k == -1(k < 0)这里存不下该状态；
所以只能在 f 数组前在加一个二维数组；k == 0 来表示 k == -1,其他的k值情况就依次后移1位；则状态转移方程就变为：`f[k+1][i][j]=min(f[k][i][j],f[k][i][k]+f[k][k][j])`
(`f[k+1][i][j]`表示从 i 到 j 的最短路长度，并且这条最短路的中间节点编号都 ≤k)


## 递推 + (数组储存状态)版

```c++
int findTheCity(int n, vector<vector<int>>& edges, int distanceThreshold) {
    const long long INF = INT_MAX >> 1;
    //初始化
    vector<vector<int>> g(n, vector<int>(n, INF));
	    //如果要考虑i到i为0的情况：
    for(int i = 0;i < n;i++) {
		g[i][i] = 0;
    }
    //建图
    for (auto& e : edges) {
        int x = e[0], y = e[1], w = e[2];
        g[x][y] = min(g[x][y],w);//有重边，维护min
        g[y][x] = min(g[y][x],w);//无向
    }

    //递推 + 数组状态记录
    vector f(n + 1,vector(n,vector<int>(n)));
    f[0] = g;
    for(int k = 0;k < n;k++) {
        for(int i = 0;i < n;i++) {
            for(int j = 0;j < n;j++) {
	            f[k + 1][i][j] = min(f[k][i][j],f[k][i][k] + f[k][k][j]);
            }
        }
    }


    int ans = -1;//定义-1表示待定
    int min_cnt = n;
    for(int i = 0;i < n;i++) {
        int cnt = 0;
        for(int j = 0;j < n;j++) {
            if(i != j && f[n][i][j] <= distanceThreshold) {
                cnt++;
            }
        }
        if(cnt <= min_cnt) {
            min_cnt = cnt;
            ans = i;
        }
    }
    return ans;
}
```



## 空间优化

**观察上面的状态转移方程，在计算 f[k+1] 时，只会用到 f[k]，不会用到（第一个维度的）下标小于 k 的状态。**
能不能像 0-1 背包那样，把第一个维度去掉呢？也就是
`f[i][j]=min(f[i][j],f[i][k]+f[k][j])`
不幸的是，我们无法确定 k 和 i,j 的大小关系，上式中的` f[i][k] `的值可能是` f[k][i][k]`，也有可能被覆盖成了` f[k+1][i][k]`。同样的，`f[k][j] `的值也可能被覆盖成了 `f[k+1][k][j]`

**但是！从状态的定义来看**：
`f[k+1][i][k]` 表示从 i 到 k 的最短路长度，并且这条最短路的中间节点编号都 ≤k。由于终点是 k，那么中间节点必然不包含 k，所以中间节点编号都 ≤k−1，所以 `f[k+1][i][k]=f[k][i][k]`！
同理，`f[k+1][k][j]=f[k][k][j]`。
因为值没变，所以不用担心被覆盖，则我们不再需要在开一层二维数组来完成递推，在原基础上修改即可。




### 提问：在 k 这一轮循环，会不会 `f[i][k]` 或者 `f[k][j]` 被提前更新，变成本轮新值，加到公式里，破坏转移？

# 核心一句话

**在第 k 次外层循环里，`f[*][k]` 和 `f[k][*]` 永远不会被修改！** 所以它们一直保存的是 k-1 轮的旧值。只有 `f[i][j]`（i,j≠k）才会被修改。

## 为什么本轮 k 循环不会修改 `f[x][k]`、`f[k][x]`？

本轮 k 的松弛式子： `f[i][j] = min(f[i][j], f[i][k] + f[k][j]);`

我们看什么时候会修改 `f[x][k]`： 要更新 `f[x][k]`，要让 i=x, j=k： `f[x][k] = min(f[x][k], f[x][k] + f[k][k]);` 而 `f[k][k]=0`，式子等价于 `min(f[x][k], f[x][k])`，**不会发生任何变化**。

同理，想要更新 `f[k][y]`，令 i=k, j=y： `f[k][y] = min(f[k][y], f[k][k] + f[k][y]);` 同样 `f[k][k]=0`，等于 `min(f[k][y], f[k][y])`，**也不会变**。

👉 本轮 k 循环从头到尾： 所有 `f[*][k]`、`f[k][*]` 的值，**全程停留在上一轮（k-1）的旧值，不会被本轮改写**。

所以当我们计算 `f[i][k] + f[k][j]` 的时候，拿到的一定是 k-1 的旧数据，和三维版本完全一样！

# 对比 01 背包（帮你感受差别）

01 背包压缩二维： `dp[j] = min(dp[j], dp[j-w] + v)` 这里 `dp[j-w]` 有可能是**本轮刚刚更新过的值**，会出错，所以 j 逆序。 原因：**转移会修改式子里面被读取的变量**。

Floyd 完全不是这个情况： 式子 `f[i][k] + f[k][j]` 里面的两项，**在本轮 k 循环根本不会被修改**，读取的永远是旧值。 只有式子左边的 `f[i][j]` 才会被覆盖。






## 对上的空间优化版

```c++
int findTheCity(int n, vector<vector<int>>& edges, int distanceThreshold) {
    const long long INF = INT_MAX >> 1;
    //初始化
    vector<vector<int>> g(n, vector<int>(n, INF));
	    //如果要考虑i到i为0的情况：
    for(int i = 0;i < n;i++) {
		g[i][i] = 0;
    }
    //建图
    for (auto& e : edges) {
        int x = e[0], y = e[1], w = e[2];
        g[x][y] = min(g[x][y],w);//有重边，维护min
        g[y][x] = min(g[y][x],w);//无向
    }

    //递推 + 数组状态记录
    vector f(n,vector<int>(n)));
    f = g;
    for(int k = 0;k < n;k++) {
        for(int i = 0;i < n;i++) {
            for(int j = 0;j < n;j++) {
	            f[i][j] = min(f[i][j],f[i][k] + f[k][j]);
            }
        }
    }

---
	int ans = -1;//定义-1表示待定
    int min_cnt = n;
    for(int i = 0;i < n;i++) {
        int cnt = 0;
        for(int j = 0;j < n;j++) {
            if(i != j && f[n][i][j] <= distanceThreshold) {
                cnt++;
            }
        }
        if(cnt <= min_cnt) {
            min_cnt = cnt;
            ans = i;
        }
    }
    return ans;
}
```



### Floyd（Floyd-Warshall）限制条件

> **目标**：求**所有点对（多源）最短路**

---
### ✅ 允许条件

1. **支持负权边**：单条边权值为负数完全允许。
2. **支持图类型**：同时适用于**有向图**和**无向图**。
3. **适合稠密图**：通常采用邻接矩阵存储。

---
### ❌ 硬性限制（核心重点）

**图里不能存在负环（负权回路）**

* **负环定义**：环上所有边权之和 $< 0$。
* **影响**：一旦存在负环，路径只要能到达该负环，就可以无限绕圈使距离无限减小，因此**不存在最短路**，Floyd 计算出的结果无意义。
* **负环检测**：Floyd 可以用来**检测负环**。算法执行完毕后，只要存在 `dist[i][i] < 0`，即代表节点 `i` 处于负环上。

---
### ⏱️ 性能与规模限制

算法时间复杂度为 $O(n^3)$，**节点数量 $n$ 不能太大**：

* **$n \le 200$**（竞赛常用）：$200^3 = 8 \times 10^6$，计算速度极快。
* **$n = 300$**：$300^3 = 2.7 \times 10^7$，勉强可用。
* **$n > 400$**：$400^3 = 6.4 \times 10^7$，大概率超时（TLE），不建议使用 Floyd，应改用多次 Dijkstra、SPFA 或 Johnson 算法。

---
### 📌 代码与实现细节

1. **存储方式**：必须采用**邻接矩阵**存图，不适合稀疏图（稀疏图建议多次运行 Dijkstra / SPFA）。
2. **INF 溢出防范**：防范加法溢出，不能直接使用 `INT_MAX`（`INF + INF` 会导致溢出）。在 C++ 中通常取 `0x3f3f3f3f` 或 `1e18`（`long long`）。
3. **循环顺序限制**：**最外层循环必须是中转点 $k$**。若将 $k$ 放在内层会导致逻辑错误（无法保证阶段无后效性与重叠更新问题）。