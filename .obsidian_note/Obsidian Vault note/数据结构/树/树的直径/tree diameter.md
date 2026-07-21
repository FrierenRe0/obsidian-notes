## 二叉树的直径：
模板题：
[543. 二叉树的直径](https://leetcode.cn/problems/diameter-of-binary-tree/)
变形题：
[124. 二叉树中的最大路径和](https://leetcode.cn/problems/binary-tree-maximum-path-sum/)
[687. 最长同值路径](https://leetcode.cn/problems/longest-univalue-path/)
[2385. 感染二叉树需要的总时间](https://leetcode.cn/problems/amount-of-time-for-binary-tree-to-be-infected/)（这题可以留个心，有不同的思考方式）
### 解释：
**1. 其实树的直径本质上就是在递归遍历树中的各个节点时，整合子节点所传递的信息（这里在计算树的直径时，常常与树中节点最大深度有关），（完成后更新答案-->全局变量），在return在当前节点所需向上传递的信息。

**2. 你会发现这里再计算树的直径时，思路和树形dp的想法很像，他们其实本质大差不差

**3.--> 注意这里有个误区：
树的直径并不是一定会过根节点；



## 例题1：
[543. 二叉树的直径](https://leetcode.cn/problems/diameter-of-binary-tree/)

![配图](../../../../截图文件/Pasted%20image%2020260708204006.png)
### 代码如下：
```c++
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode() : val(0), left(nullptr), right(nullptr) {}
 *     TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
 *     TreeNode(int x, TreeNode *left, TreeNode *right) : val(x), left(left), right(right) {}
 * };
 */
class Solution {
public:
    int mx = -1;
    int dfs(TreeNode* root) {
        if(!root) return -1; 
        int l = dfs(root->left);
        int r = dfs(root->right);
        mx = max(mx,l + r + 2);
        return max(l,r) + 1;
    }
    int diameterOfBinaryTree(TreeNode* root) {
        dfs(root);
        return mx;
    }
};
```





## 例题2：
[2385. 感染二叉树需要的总时间](https://leetcode.cn/problems/amount-of-time-for-binary-tree-to-be-infected/) **(这里有多解法)

![配图](../../../../截图文件/Pasted%20image%2020260708203843.png)

### 思想一：—> 邻接表 + bfs

**思路：这里把start作为根节点，去找树的最大深度，去建立一个邻接表完成建图，再bfs完成最大深度求解

**代码如下：
```c++
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode() : val(0), left(nullptr), right(nullptr) {}
 *     TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
 *     TreeNode(int x, TreeNode *left, TreeNode *right) : val(x), left(left), right(right) {}
 * };
 */
class Solution {
public:
    vector<int> graph[100001];
    //建表
    void dfs(TreeNode* root) {
        if(!root) return;
        int u = root->val;
        if(root->left) {
            int v = root->left->val;
            graph[u].push_back(v);
            graph[v].push_back(u);
            dfs(root->left);
        }
        if(root->right) {
            int v = root->right->val;
            graph[u].push_back(v);
            graph[v].push_back(u);
            dfs(root->right);
        }
        return;
    }
    //bfs遍历
    int bfs(int start) {
        queue<int> q;
        q.push(start);
        int ans = 0;
        bool vis[100001] = {false};
        vis[start] = true;
        while(!q.empty()) {
            int len = q.size();
            for(int i = 0;i < len;i++) {
                auto x = q.front();
                q.pop();
                for(auto& next : graph[x]) {
                    if(!vis[next]) {
                        vis[next] = true;
                        q.push(next);
                    }
                }
            }
            if(q.size()) ans++;
        }
        return ans;
    }
    int amountOfTime(TreeNode* root, int start) {
        //graph.resize(100001);
        dfs(root);
        return bfs(start);
    }
};
```

**但是这里我需要完成两遍遍历，如何去优化成一次遍历

### 思想二：—> 类似于树的直径思想

## 第一步：先拆解题目本质

**题目：起点节点同步向四周扩散，求整树感染完的时间 = 起点 start 到树上最远节点的距离。
放到二叉树里，两点之间路径只有两种形态：
- 路径**不拐弯**：起点向下走到自己子树的叶子（纯左 / 纯右分支）--> 这里则直接时处理子节点的最大深度更新答案； 
- 路径**拐弯**：起点往上走到某个祖先节点，再拐弯走到另一侧子树的叶子  --> 这里则是有点类似于树的直径，只不过一端的节点是受限的，这个节点为start这个确定的节点；

#### 这里再递归的时候可以分为之后的这4中情况：
**递归时，除了返回当前子树的最大链长加一，还需要返回一个布尔值，表示当前子树是否包含 start。
如果当前节点是空节点，返回 0 和 false。
设左子树的返回的链长为 lLen，右子树返回的链长为 rLen。
如果当前节点值等于 start，初始化答案为 max(lLen,rLen)，即子树 start 的最大深度。然后返回 1 和 true。
如果左右子树都不包含 start，返回 max(lLen,rLen)+1。
**如果左子树或右子树包含 start，像计算直径那样，用 lLen+rLen 更新答案的最大值。如果左子树包含 start，则返回 lLen 和 true，否则返回 rLen 和 true。这种返回方式可以保证 lLen+rLen 一定是端点为 start 的直径长度。**

**代码如下：
```c++
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode() : val(0), left(nullptr), right(nullptr) {}
 *     TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
 *     TreeNode(int x, TreeNode *left, TreeNode *right) : val(x), left(left), right(right) {}
 * };
 */
class Solution {
public:
    int ans = 0;
    int target;
    pair<int,bool> dfs(TreeNode* root) {
        if(!root) return {0,false};
        auto [l_len,l_bound] = dfs(root->left);
        auto [r_len,r_bound] = dfs(root->right);
        if(root->val == target) {
        
			// 计算子树 start 的最大深度
            ans = max(l_len,r_len);
            return {1,true}; //已经找到start
        }
        if(l_bound || r_bound) {
	    
			// 只有在左子树或右子树包含 start 时，才能更新答案
            ans = max(ans,l_len + r_len);
            // 保证start是直径的端点
            return {(l_bound ? l_len : r_len) + 1,true};
        }
        return {max(l_len,r_len) + 1,false};
    }
    int amountOfTime(TreeNode* root, int start) {
        target = start;
        dfs(root);
        return ans;
    }
};
```