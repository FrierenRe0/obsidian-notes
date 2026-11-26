## 相关链接：

- 题目链接：[2127. 参加会议的最多员工数 - 力扣（LeetCode）](https://leetcode.cn/problems/maximum-employees-to-be-invited-to-a-meeting/description/)

## 思路：

### 一、 什么是“基环树”？

在这道题中，每个人只有一个最喜欢的员工（`favorite[v] = u`），这意味着在由“人”和“最喜欢的人”构成的有向图中，**每个节点的出度（Out-degree）都正好等于 1**。

这种特殊的图结构被称为**基环树森林（Functional Graph）**：

- **结构特点**：整个图由若干个连通分量组成。每个连通分量内部都**有且仅有一个环**，而环的周围可以挂着许多棵有向树（树的边方向指向环内；或者说在该题目背景下，是一个内向基环树森林。
    
- **直观理解**：就像是一个个或者“带有尾巴的圈”：
    
    1. **尾巴（树枝部分）**：入度为 0 或逐步消去的节点，最终会汇聚到环上。
        
    2. **核心（环部分）**：环上的节点形成闭环（比如 A 喜欢 B，B 喜欢 C，C 喜欢 A）。
        

### 二、 题目本质与两种最大化方案

要让尽可能多的人坐在一起，且满足“每个人都坐在自己最喜欢的人旁边”，最终的座次安排只有两种可能的形式：

1. **大于 2 的环（大环）**：
    
    - 比如 $A \to B \to C \to D \to A$。这些人可以围坐成一个大圆桌。
        
    - 注意：**如果有多个这样的大环，我们只能选择其中最大的一个**，因为不同的独立大环无法合并到同一个圆桌里。
        
2. **大小为 2 的环（双人互喜欢，即互为最爱）**：
    
    - 比如 $A \leftrightarrow B$（$A$ 喜欢 $B$，$B$ 也喜欢 $A$）。
        
    - 这种情况下，不仅 $A$ 和 $B$ 可以相邻，所有挂在 $A$ 和 $B$ 身上的长链（树枝）都可以排成长队接在 $A$ 和 $B$ 的两侧。
        
    - 并且，**所有的 2-环及其挂载的长链都可以同时并存**（因为它们各自独立，可以拼成多张桌子或一条长龙），所以所有符合条件的 2-环贡献的的人数可以**累加**求和。
        

最终的答案就是：**所有独立 2-环及挂载链的总人数之和 (`sum_circle`)** 与 **任意一个最大大环的人数 (`big_circle`)** 中的较大值。



### 三、 时空复杂度分析：
	
- **时间复杂度分析：$O(n)$**
	
- **空间复杂度分析：$O(n)$**


## 代码详情：
```c++
class Solution {
public:
    int maximumInvitations(vector<int>& favorite) {
        int n = favorite.size();
        vector<bool> vis(n,false);
        vector<int> cnt(n);
        vector<vector<int>> g(n);
        vector<int> sz(n,1);
        for(int v = 0;v < n;v++) {
            int u = favorite[v];
            cnt[u]++;
            g[v].push_back(u);
        }   
        queue<pair<int,int>> q;
        for(int i = 0;i < n;i++) {
            if(!cnt[i]) {
                q.push({i,sz[i]});
                vis[i] = true;
            }
        }
        while(!q.empty()) {
            auto [x,cur] = q.front();
            q.pop();
            for(auto& next : g[x]) {
                cnt[next]--;
                sz[next] = max(sz[next],cur + 1);
        
                if(!cnt[next]) {
                    q.push({next,sz[next]});
                    vis[next] = true;
                }
            }
        }   
        int sum_circle = 0; //环大小: = 2
        int big_circle = 0; //环大小: > 2
        auto dfs = [&](this auto&& dfs,int x) -> int {
            if(vis[x]) return 0;
            vis[x] = true;
            int sum = 1;
            sum += dfs(favorite[x]);
            return sum;
        };
        for(int i = 0;i < n;i++) {
            if(vis[i]) continue;
            int size = dfs(i);
            if(size == 2) sum_circle += sz[i] + sz[favorite[i]];
            else big_circle = max(big_circle,size);
        }
        return max(sum_circle,big_circle);
    }
};
```