## 相关题目链接：
- 模板题：
	[543. 二叉树的直径](https://leetcode.cn/problems/diameter-of-binary-tree/)
- 变形题：
	[124. 二叉树中的最大路径和](https://leetcode.cn/problems/binary-tree-maximum-path-sum/)
	[687. 最长同值路径](https://leetcode.cn/problems/longest-univalue-path/)
	[2385. 感染二叉树需要的总时间](https://leetcode.cn/problems/amount-of-time-for-binary-tree-to-be-infected/)（这题可以留个心，有不同的思考方式）

## 树的直径本质就是树形dp :

**解释**：
1. 其实树的直径本质上就是在递归遍历树中的各个节点时，整合子节点所传递的信息（这里在计算树的直径时，常常与树中节点最大深度有关/子树的最长链），（完成后更新答案 -> 全局变量），在向父节点所需向上传递的信息。会发现对于树的直径问题本质其实就是树形dp的思想
	
	所以，对于该问题就归类到树形dp大类，具体查看“E - 动态规划 记忆搜索 - dp”章节 - “树形dp”
	
1. 注意这里有个误区 ：树的直径并不是一定会过根节点；
