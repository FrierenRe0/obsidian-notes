## 基本板子
```c++
//要求：边不可以有负权
int solve() {
	//表示有n个节点 Based -> 0
	int n;
	cin >> n;

	//建邻接表
	vector<vector<pair<int,int>>> graph(n);
	int k;
	cin >> k;
	while(k--) {
		int u,v,w;
		cin >> u >> v >> w;
		graph[u].push_back({v,w});
		// graph[v].push_back({u,w}); // 无向边: add
	}
	//dijkstra算法(单源最短路)
	int start; //Based -> 0
	cin >> start;
	priority_queue<pair<int,int>> pq;
	vector<int> dis(n;LLONG_MAX);
	dis[start] = 0;
	pq.push({0,start});

	while(!pq.empty()) {
		auto &[dis_x,x] = pq.front();
		pq.pop();
		if(dis_x > dis[x]) {
			continue;
		}
		for(auto& [next,next_w] : graph[x]) {
			if(dis_x + next_w < dis[next]) {
				dis[next] = dis_x + next_w;
				pq.push({dis_x + next_w,next});
			}
		}
	}
}
```


## 题解：

**拓展**：
**如果存在某些特殊设定，如在按层展开中，如果一些特殊格子有冰冻期，需要等待这个格子过完这一段时间才可以访问**

**这种要优先顺序遍历 ->通常需要用到priority_queue数据结构->这种数据结构在图论中通常时dijkstra的底层基础思路**

**网格图的多源最短路 + 时间限制的 Dijkstra：[D-小L的扩展_2026牛客寒假算法基础集训营6](https://ac.nowcoder.com/acm/contest/120566/D)**
