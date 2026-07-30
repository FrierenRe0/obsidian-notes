**ST表其实就是一个储存对应静态子区间的二维数组，采用了二分的思想将查询时间复杂度提高到（log(n)）级别，但是ST表一般是处理静态区间问题。**

**补充**：
不是 ST 表不能处理区间更新，而是：完全没必要！（相比树状数组和线段树的log(n)）；
理论上：
- 你**可以**每次更新后，**重新暴力构建一遍 ST 表**
- 但这样时间复杂度直接炸穿 → **O (n log n) 每次更新**
- 这和不用 ST 表没区别

**解释**：
- `bit_width((size_t)n)`
	- 这里一定要传无符号整数。
	- 作用：**求 x 二进制最高位是第几位**。
- 在st二维数组中，i表示在原数组中的下标，j表示在该列中对应区间的长度其2的幂次
- st表中就可以表示原数组中，所有可以维护任意以2的幂次级长度的子区间；
```c++
vector<vector<int>> st;
void init(vector<int>& nums) {
	int n = nums.size();
	int m = bit_width((unsigned)n);
	st.resize(n,vector<int>(m));
	for(int i = 0;i < n;i++) {
		st[i][0] = nums[i];
	}
	//我这里是维护子区间max;
	for(int j = 1;j < m;j++) {
		for(int i = 0;i + (1 << j) <= n;i++) {
			st[i][j] = max(st[i][j - 1],st[i + (1 << (j - 1))][j - 1]);
		}
	}
}
//这里定义区间[l,r) 左闭右开 --> 这里定义就不会忘记什么 r - (1 << k)还要减1
int query(int l,int r) {
	int k = bit_width((size_t)r - l) - 1;
	int mx = max(st[l][k],st[r - (1 << k)][k]);
	return mx;
}
```
