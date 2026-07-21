# 启发思考：寻找子问题
![配图](../../../截图文件/Pasted%20image%2020260526164457.png)

# 二、递归怎么写：状态定义与状态转移方程

**1.定义 dfs(k,i,j) 表示从 i 到 j 的最短路长度，并且这条最短路的中间节点编号都 ≤k。注意中间节点不包含 i 和 j。

**2.根据上面的讨论：
不选 k，那么中间节点的编号都 ≤k−1，即 dfs(k,i,j)=dfs(k−1,i,j)。选 k，问题分解成从 i 到 k 的最短路，以及从 k 到 j 的最短路。由于这两条最短路的中间节点都不包含 k，所以中间节点编号都 ≤k−1，故得到 dfs(k,i,j)=dfs(k−1,i,k)+dfs(k−1,k,j)。这两种情况取最小值，就得到了 dfs(k,i,j)。
		
		写成式子就是：dfs(k,i,j)=min(dfs(k−1,i,j),dfs(k−1,i,k)+dfs(k−1,k,j))
	
**递归边界：dfs(−1,i,j)=`w[i][j]`。k=−1 表示 i 和 j 之间没有任何中间节点，此时最短路长度只能是连接 i 和 j 的边的边权，即 `w[i][j]`。如果没有连接 i 和 j 的边，则 `w[i][j]` = ∞。

**递归入口：dfs(n−1,i,j)，表示从 i 到 j 的最短路长度。k=n−1 是因为本题节点编号为 0 到 n−1，任意最短路的中间节点编号都 ≤n−1。



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

**超时原因：
在递归计算中，`dfs[k][i][j]`在每个节点计算和其他节点的最短路中的整个过程中，不同的节点可能会对`dfs[k][i][j]`这个状态计算重复多次；
这里就可以用f数组来储存对应`dfs[k][i][j]`的状态情况；
(同时这里的`f[k][i][j]`和`dfs[k][i][j]`的定义相同,都是表示在i到j节点中，中间节点的编号<= k)

**注意：
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
    
-----------------------------------------------------------------------------------
    
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
**观察上面的状态转移方程，在计算 f[k+1] 时，只会用到 f[k]，不会用到（第一个维度的）下标小于 k 的状态。
能不能像 0-1 背包那样，把第一个维度去掉呢？也就是
`f[i][j]=min(f[i][j],f[i][k]+f[k][j])`
不幸的是，我们无法确定 k 和 i,j 的大小关系，上式中的` f[i][k] `的值可能是` f[k][i][k]`，也有可能被覆盖成了` f[k+1][i][k]`。同样的，`f[k][j] `的值也可能被覆盖成了 `f[k+1][k][j]`

**但是！从状态的定义来看：
`f[k+1][i][k]` 表示从 i 到 k 的最短路长度，并且这条最短路的中间节点编号都 ≤k。由于终点是 k，那么中间节点必然不包含 k，所以中间节点编号都 ≤k−1，所以 `f[k+1][i][k]=f[k][i][k]`！
同理，`f[k+1][k][j]=f[k][k][j]`。
因为值没变，所以不用担心被覆盖，则我们不再需要在开一层二维数组来完成递推，在原基础上修改即可。

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
    
-----------------------------------------------------------------------------------
	
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