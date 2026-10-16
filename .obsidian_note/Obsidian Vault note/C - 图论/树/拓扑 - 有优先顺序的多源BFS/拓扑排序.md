## 核心概念 :
**拓扑排序（Topological Sort）** 是一种针对 *有向无环图*（DAG, Directed Acyclic Graph）的节点线性排序算法。

在图论中，如果存在一条从节点 u 到节点 v 的有向边 (u → v)，那么在拓扑排序的结果序列中，u 必须出现在 v 的前面。

- **适用条件**：目标图必须是**有向无环图（DAG）**。如果图中有环，节点之间会形成循环依赖，无法确定绝对的前后顺序。
    
- **结果唯一性**：拓扑排序的结果**不一定唯一**。只要满足所有依赖约束，任何顺序都是合法的。
	**注意：** 但是如果需要构造出一个（如字典序最小）从而找到一个最小排序的情况，在结果上可能存在唯一性；

## 1. Kahn 算法（基于入度的广度优先搜索 / BFS）

- **思想**：依次找出没有任何前置依赖（入度为 0）的节点，将其放入结果集中，并从图中删除该节点及其发出的边（即更新邻接节点的入度）。
    
- **步骤**：
    
    - 计算图中所有节点的入度（In-degree）。
        
    - 将所有入度为 0 的节点放入队列中。
        
	- 当队列不为空时：
        
        - 弹出队首节点 $u$，加入拓扑序列。
            
        - 遍历 $u$ 的所有邻接点 $v$，将 $v$ 的入度减 1。
            
        - 若 $v$ 的入度变为 0，则将 $v$ 入队。
            
    - 若最终输出的节点数等于图的总节点数，则说明排序成功；否则说明图中存在**环**。

### 代码展示：
```cpp
vector<int> topologicalSort(int n, vector<vector<int>>& edges) {
	//返回(DAG)有向无环图的其中一个拓排序,无则return空列表
	//节点编号1 ~ n;
	vector<vector<int>> graph(n + 1);
	vector<int> in(n + 1);
	for(auto& e : edges) {
		int u = e[0],v = e[1];
		graph[u].push_back(v);
		in[v]++;
	}
	queue<int> q;
	for(int i = 1;i <= n;i++) {
		if(!in[i]) q.push(i);
	}
	
	vector<int> topo_order;
	topo_order.reserve(n);
	//这里如果题目不需要我们去记录拓排序，而是只是通过topo_ordersize()判断一下是否存在环，cnt统计一下即可；
	while(!q.empty()) {
		auto x = q.front();
		q.pop();
		topo_order.push_back(x);
		for(auto&next : graph[x]) {
			in[next]--;                  //去边
			if(!in[next]) q.push(next);  //入度为0时，加入队列
		}
	}
	if(topo_order.size() < n) return {};
	else return topo_order;
}
```

## 变形：构造最小字典序拓排序
当然如果存在这种在拓排序中是这种唯一性构造最字典序拓排序（小/大）情况，则是把`queue`换成`prioity_queue`（小根堆/大根堆）从而优先处理（小下标/大下标）即可；

### 优化思路代码如下`priority_queue`：
```cpp
vector<int> topologicalSort(int n, vector<vector<int>>& edges) {
	//返回(DAG)有向无环图的其中一个最小字典序拓排序,无则return空列表
	//节点编号1 ~ n;
	vector<vector<int>> graph(n + 1);
	vector<int> in(n + 1);
	for(auto& e : edges) {
		int u = e[0],v = e[1];
		graph[u].push_back(v);
		in[v]++;
	}
	priority_queue<int,vector<int>,greater<int>> pq;
	for(int i = 1;i <= n;i++) {
		if(!in[i]) pq.push(i);
	}
	
	vector<int> topo_order;
	topo_order.reserve(n);
	//这里如果题目不需要我们去记录拓排序，而是只是通过topo_ordersize()判断一下是否存在环，cnt统计一下即可；
	while(!pq.empty()) {
		auto x = pq.top();
		pq.pop();
		topo_order.push_back(x);
		for(auto&next : graph[x]) {
			in[next]--;                  //去边
			if(!in[next]) pq.push(next);  //入度为0时，加入队列
		}
	}
	if(topo_order.size() < n) return {};
	else return topo_order;
}
```

