**对于单点更新（非区间更新）线段树

**//这里我先定义一下：
每个用来维护对应区间的节点，我把它叫做区间节点

**再初始化线段树的时候，如果n是2的幂，逻辑上为满二叉树
如果n不是2的幂，在逻辑上则不是满二叉树，为完全二叉树

**为啥实在逻辑上为 ---：
因为在逻辑上线段树是用数组来维护，每一个区间节点的编号以数组中的i下标来代替表示


```c++
vector<int> tree;
int merge_val(int a,int b) {
	//根据题目情况具体分析维护数据;
	return max(a,b);
}

void maintain(int node) {
	tree[node] = merge_val(tree[node * 2],tree[node * 2 + 1]);
}

void build(const vector<int>& a,int l,int r,int cur) {
	if(l == r) {
		tree[cur] = a[l];
		return;
	} 
	int mid = (l + r) >> 1;
	build(a,l,mid,cur * 2);
	build(a,mid + 1,r,cur * 2 + 1);
	maintain(node);
}

//单点更新
void update(int cur,int l,int r,int idx,int val) {
	if(l == r) {
		tree[node] = merge_val(tree[node],val);
		return;
	}
	int mid = (l + r) >> 1;
	if(idx <= mid) {
		update(node * 2,l,mid,idx,val);
	}
	else {
		update(node * 2 + 1,mid + 1,r,idx,val);
	}
	maintain(node);
} 
//维护区间[L,R]数据
int query(int cur,int l,int r,int L,int R) {
	if(L <= l && r <= R) {
		return tree[cur];
	}
	int mid = (l + r) >> 1;
	if(L <= mid) {
		int a = query(cur * 2,l,mid,L,R);
	} 
	if(R > mid) {
		int b = query(cur * 2,mid + 1,r,L,R);
	} 
	return merge_val(a,b);
}

// 返回区间 [0,R] 中 <= val 的最靠左的位置，不存在时返回 -1
int findFirst(int cur, int l, int r, int R, int val) {
	if (tree[cur] > val) {
		return -1; // 整个区间的元素值都大于 val
	}
	if (l == r) {
		return l;
	}
	int m = (l + r) / 2;
	if (tree[cur * 2] <= val) {
		return findFirst(cur * 2, l, m, R, val);
	}
	if (R > m) {
		return findFirst(cur * 2 + 1, m + 1, r, R, val);
	}
	return -1;
}

solve() {
	int n;
	cin >> n;
	vector<int> a(n);
	for(int& i : a) cin >> i;
	//初始化
	build(a,0,n - 1,cur);
	//tree数组的空间大小开n * 4可以存下所有的区间节点
}
```

**注释：
- **线段树下标是based -> 1；
-  **把任意区间用 O(logn) 个区间表示，线段树的每个节点记录对应区间的信息。
	- **询问：把询问区间拆分成 O(logn) 个区间，对应着线段树的 O(logn) 个节点，把这 O(logn) 节点的信息合并，即为答案。
	- **单点更新：有 O(logn) 个区间包含被修改的位置，需要更新 O(logn) 个节点的信息。
