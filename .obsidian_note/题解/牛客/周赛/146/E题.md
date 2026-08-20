## 评价：本质上是贪心 -> 因为是做最优解

## 博弈题
![配图](../../../../截图文件/Pasted%20image%2020260602185609.png)
**1.在本题中，因为两人的选择方式一一定是按照非递减方式去选择的

**这里我用例子描述**：
如在 ： 5 5  4 4  3 3 3  2 2  11
2.因为这里先手的想要赢，他在最后会是一个怎么局面，也就是推到最后一组（最大）的第一个数字时，只有当两者操作总次数为奇数时，这样red就会赢

2.怎么才可以在保证red至少存在一个总操作次数为奇数的情况？
将所有数高低分类
思考，大值在逻辑树上等价于叶子节点，如果red胜，改层的操作数量是因该为要是奇数；
分类讨论：
如果该层是偶数，如果red在第一次是怎么上面选的，fang一定会让总操作次数为偶数；
如果为奇数，red一定会让总操作次数为奇数

**总结：所以我们找到了从下到上的逻辑树路径查找过程 -> 从大值到小值看数量奇偶性如果当层的数字数量时奇数时，red肯定时会赢的；否则就往后看，直到发现当前层时奇数；如果看完了还没有奇数，那red必输**

## 代码参考如下：
```c++
#include <bits/stdc++.h>
using namespace std;
#define int long long
const int mod = 1e9 + 7;
const int N = 3e5 + 7;

int fast_io = []()
{
    ios::sync_with_stdio(false);
    cin.tie(nullptr);
    cout.tie(0);
    return 0;
}();

int init = []()
{
    return 0;
}();

void solve() {
    int n;
    cin >> n;
    map<int,int> mp;
    for(int i = 0;i < n;i++) {
        int a;
        cin >> a;
        mp[a]++;
    }
    for(auto it = mp.rbegin();it != mp.rend();it++) {
        if(it->second & 1) {
            cout << "red" << endl;
            return;
        }
    }
    cout << "fang" << endl;
    return;
}
signed main() {
    int T = 1;
    cin >> T;
    while (T--) {
        solve();
    }
    return 0;
}
```
