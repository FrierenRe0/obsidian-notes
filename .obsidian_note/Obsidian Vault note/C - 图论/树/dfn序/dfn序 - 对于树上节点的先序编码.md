# DFN 序（深度优先搜索序）核心概念

> **DFN（Depth-First Numbering / DFS Sequence）** 是树上算法中极其强大的一种思想。它的核心本质是：**将复杂的树形结构“拍扁”成一维的线性数组**，从而把许多原本需要在树上跳跃、递归的区间问题，转化为我们熟知的一维数组上的区间操作。

## 一、 什么是 DFN 序？

在对一棵树进行 **DFS（深度优先搜索）** 时，按照访问节点的先后顺序，给每个节点打上一个编号，这个编号就是该节点的 **DFN 序**。

通常在遍历时，我们会记录两个关键数组：

1. **`dfn[u]`**：节点 $u$ 在 DFS 遍历中被**第一次访问到的顺序/时间戳**。
    
2. **`sz[u]`**：以节点 $u$ 为根的子树中**节点的总数**。


### 核心性质：子树的连续性

这是 DFN 序最神奇、最重要的性质：

> **在一棵树的先序遍历（Pre-order）中，以任意节点 $u$ 为根的子树，在 DFN 序中占据的必然是一个连续的闭区间：**
> 
> $$\mathbf{[dfn[u], dfn[u] + sz[u] - 1]}$$

这意味着，只要我们知道了某个子树的根节点 $u$ 及其子树大小 $sz[u]$，我们就能在 $O(1)$ 时间内定位出这棵子树包含的所有节点在扁平化数组中的位置。



## 二、 通过例题来强化对于dfn序的理解：

### 相关链接：

- 题目链接：[2458. 移除子树后的二叉树高度 - 力扣（LeetCode）](https://leetcode.cn/problems/height-of-binary-tree-after-subtree-removal-queries/submissions/)

### 题目：

![](../../../../截图文件/Pasted%20image%2020261016151038.png)

### 思路：

在刚才做的题目中（删除某个子树，求剩余节点的最大深度），传统的做法每次删除子树都要去遍历树、重新计算，非常耗时。而结合 **DFN 序 + 前后缀最大值**，可以将复杂度降到极致：

1. **扁平化建树**：
    
    我们通过一次 DFS，把整棵树的节点按照访问顺序排成一条“直线”（对应数组下标 $1$ 到 $n$）。同时记录下每一个节点的深度 `deep`、在直线上的位置 `dfn` 以及子树大小 `sz`。
    
2. **子树转化为区间**：
    
    当题目要求“删除以节点 `x` 为根的子树”时，在 DFN 直线数组中，这等价于挖掉了区间：
    
    $$\mathbf{[dfn[x], dfn[x] + sz[dfn[x]] - 1]}$$
    
3. **利用前后缀极值快速查询**：
    
    挖掉中间一个区间后，剩余的有效节点被分成了两部分：
    
    - 左半部分区间：$[1, dfn[x] - 1]$
        
    - 右半部分区间：$[dfn[x] + sz[dfn[x]], n]$
        
    
    通过预处理**前缀最大值数组（`pre`）**和**后缀最大值数组（`back`）**，我们就可以在 **$O(1)$** 的时间里直接获取这两部分的最大深度，两者取个 `max` 就是答案。


### 代码展示：

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
const int N = 1e5 + 1;
class Solution {
public:
    int dfn[N];
    int sz[N];  // idx -> dfn;
    int deep[N]; // idx -> dfn; 
    vector<int> treeQueries(TreeNode* root, vector<int>& queries) {
        int n = 0;
        int dep = 0;
        //先序编码，将树型结构庄转成线性结构
        auto dfs = [&](this auto&& dfs,TreeNode* x) -> int {
            if(!x) return 0;
            int val = x->val;
            int node = ++n;
            dfn[val] = node;
            deep[node] = dep;
            dep++;
            int size = 1;
            size += dfs(x->left);
            size += dfs(x->right);
            sz[node] = size;
            dep--;
            return size;
        };
        dfs(root);
        
        //将 树型结构 - 树 转成 线性结构 - 数组 后，即可前后缀维护；
        vector<int> pre(n + 1),back(n + 2);
        int mx = -1;
        for(int i = 1;i <= n;i++) {
            mx = max(mx,deep[i]);
            pre[i] = mx;
        }
        mx = -1;
        for(int i = n;i >= 1;i--) {
            mx = max(mx,deep[i]);
            back[i] = mx;
        }
        
        vector<int> ans;
        ans.reserve((int)queries.size());
        //每次o(1)query:
        for(auto& x : queries) {
            int node = dfn[x];
            int size = sz[node];
            ans.push_back(max(pre[node - 1],back[node + size]));
        }
        return ans;
    }
};
```