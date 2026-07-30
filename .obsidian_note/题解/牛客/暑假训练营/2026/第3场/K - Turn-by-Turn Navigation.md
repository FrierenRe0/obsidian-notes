## 相关链接
题卷：[2026牛客暑期多校训练营_3.pdf](file:///C:/Users/l'x/Downloads/2026%E7%89%9B%E5%AE%A2%E6%9A%91%E6%9C%9F%E5%A4%9A%E6%A0%A1%E8%AE%AD%E7%BB%83%E8%90%A5_3.pdf)
题目链接：[K-Turn-by-Turn Navigation_2026牛客暑期多校训练营3](https://ac.nowcoder.com/acm/contest/133878/K)

## 题目大意：

![](../../../../../截图文件/Pasted%20image%2020260730153408.png)

平面直线之间的位置关系判断，一段经典的叉乘问题：(模板题)
--> 不了解叉乘的可以看我算法章中数论叉乘章节

## 代码如下：
```c++
#include <bits/stdc++.h>  
using namespace std;  
#define int long long
#define endl '\n'
const int mod = 1e9 + 7;
const int MOD = 998244353;

void ap() {
	int n;
    cin >> n;
    vector<array<int,2>> a(n);
    for(int i = 0;i < n;i++) {
        cin >> a[i][0] >> a[i][1];
    }
    //这里的(x,y)表示第一段向量
    int x = a[1][0] - a[0][0];
    int y = a[1][1] - a[0][1];
    
    //从第3个点出发(作为第二段向量)，依次看相邻两个向量的位置关系
    for(int i = 2;i < n;i++) {
        
        //(x1,y1)建立当前向量
        int x1 = a[i][0] - a[i - 1][0];
        int y1 = a[i][1] - a[i - 1][1];
        
        //叉乘判断当前相邻向量位置关系
        if(x * y1 - y * x1 > 0) cout << "LEFT";
        else if(x * y1 - y * x1 == 0) cout << "STRAIGHT";
        else cout << "RIGHT";
        cout << " ";
        
        //旧向量更新
        x = x1;
        y = y1;
    }
    cout << endl;
	return ;
}

signed main() {
	ios::sync_with_stdio(false);
	cin.tie(nullptr);
	int T = 1;
	cin >> T;
	while (T--) {
		ap();
	} 
    return 0;
}
```
