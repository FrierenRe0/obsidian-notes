--并查集基本模板:
--
```c++
vector<int> p;       // p数组用于记录1到n(对应下标化)对应的父节点
vector<int> sz;      // 集合大小
int count = maxn;    // count代表连通块的数量

int Find(int x) {
    if(p[x] == x) {
        return x;
    } else return p[x] = Find(p[x]);
}

void Union(vector<int>& p,int from ,int to,int& count) {
    int x = Find(from);
    int y = Find(to);

    if(x == y) return;
    p[x] = y;
    sz[y] += sz[x];
    count--;
    return;

}

void solve() {
    int n;
    cin >> n;
    count = n;
    p.resize(n + 1);  //based - 1
    sz.resize(n + 1); //based - 1
    
    for(int i = 0;i <= maxn;i++) p[i] = i;
    
    ....
    ....
}

```

## p数组

```c++
//p数组是用来记录每个节点的根节点；
int p[maxn]= {......};
//maxn可以个人根据题目的数据来设定
```

## find(查操作)


```c++
/find操作：查找x属于哪一个集合（找根节点）
   int find(int x) {
       if(p[x] == x) {
           return x;
       }
       else {
           return p[x] = find(p[x]);
           //递归回溯的时候把路径上的节点指向根节点（路径压缩）
           //好处：是在下次在查找路径上的元素时间复杂度是o(1);
       }
   }
```

## union(并操作)

基础操作:
```c++
//union合并操作：
   void Union(int x,int y) {
       int rootx = find(x);
       int rooty = find(y);
       
       if(rootx != rooty) {
           p[rootx] = rooty;
           //可能会增加合并后的高度：
       }    
   }
```


优化操作
1.按高度合并:

```c++
int h[maxn] = {.....}; //初始化为1；
   void Union(int x,int y) {
       int rootx = find(x);
       int rooty = find(y);
       
       if(h[rootx] <= h[rooty]) {
           p[rootx] = rooty;
           if(h[rootx] == h[rooty]) h[rooty]++;
       } else {
           p[rooty] = rootx;
       }  
   }
```

2.按大小合并：

```c++
int s[maxn] = {.....};//初始化为1；
   void Union(int x,int y) {
       int rootx = find(x);
       int rooty = find(y);

       if(s[rootx] <= s[rooty]) {
           p[rootx] = rooty;
           s[rooty] += s[rootx];
       } else {
           p[rooty] = rootx;
           s[rootx] += s[rooty];
       }
   }
```
