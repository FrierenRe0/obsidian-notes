![[Pasted image 20260514115706.png]]

**提问：
- 在什么情况下我们可以用到树状数组，也就是说树状数组可以解决一个什么问题；
- 在去理解树状数组的基本算法原理；
	 关键是去理解 ：
     - 树状数组中每个点对应的关键区间表示的是个什么范围；
     - 如何完成对于在单点数据更改中，如何把只涉及该点的关键区间更新维护；
     - 同时完成对于区间范围查询时，如何统计该范围的数据；
     d
[307. 区域和检索 - 数组可修改 - 力扣（LeetCode）](https://leetcode.cn/problems/range-sum-query-mutable/solutions/2524481/dai-ni-fa-ming-shu-zhuang-shu-zu-fu-shu-lyfll/)

**树状数组的模板：
```c++
vector<int> tree;
void update(int i,int add) {
	for(;i < tree.size();i += i & -i) {
		tree[i] += add; //这里要根据题目的要求维护对应所需要的数据
	}
}
int pre(int i) {
	int sum = 0;
	for(;i > 0;i &= (i - 1)) {
		sum += tree[i]; // 同上方注释
	}
	return sum;
}
int query(int l,int r) { //注意：l,r based->1
	return pre(r) - pre(l - 1); //同上方注释
}

void solve() {
	....;
	tree.resize(n + 1);
	//根据题目完单点更新，区间查询
}
```
