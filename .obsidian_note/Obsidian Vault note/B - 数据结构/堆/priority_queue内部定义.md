## 1.对于优先队列的自定义排序:
```c++
#define pp pair<string,int>
//#using pp = pair<string,int>;两种写法任意
struct cmp {
	bool operator()(const pp& a,const pp& b) const {
		//.....写排序逻辑
	}
};
```

但是注意：
`priority_queue`的底层排序逻辑是大根堆，也就是说
```c++
struct cmp {
	bool operator()(int a, int b) {
		return a < b;
	}
};
```

**含义：a 优先级比 b 低 → a 往下沉 → 大的留在上面、

 **同样的 cmp 比较函数：效果完相反！**
- `sort` () → 从小到大
- `priority_queue` → 从大到小（大顶堆）

例如:[2-2 天梯赛的赛场安排 - 26天梯热身赛4](https://pintia.cn/problem-sets/2038803646327758848/exam/problems/type/7?problemSetProblemId=2038804504110510081)

问题：
**为什么 `priority_queue` 必须用 `struct` 写 `cmp`，不能像 `sort` 那样直接写函数？**
核心结论
- **`sort` 用的是函数**，实在调用变量名
- **`priority_queue` 用的是** **类型（type）**

**为什么 sort 可以直接用函数？
```c++
sort(a, a+n, cmp);
```
这里的 `cmp` 是一个**变量、一个函数名、一个可调用的东西**。

```c++
priority_queue<pp, vector<pp>, cmp>
```
**尖括号 <> 里面必须放 类型 type
必须是**类型
如：
- `int` → **类型
- `vector<int>` → **类型
- `struct cmp` → **类型
- **函数名 → 不是类型！**
所以你**必须用 struct /class 包一层**，让它变成一个**类型**！
