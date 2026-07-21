## 逻辑推理题
![配图](../../../../截图文件/Pasted%20image%2020260602185609.png)
**1.在本题中，因为两人的选择方式一一定是按照非递减方式去选择的

**这里我用例子描述：
如在 ： 5 5  4 4  3 3 3  2 2  11
2.因为这里先手的想要赢，他在最后会是一个怎么局面，也就是推到最后一组（最大）的第一个数字时，只有当red为先手时，则组数字的数量为奇数时，或当fang为先手时，则组数字的数量为偶数时，这样red就会赢

**3.那我可不可以这样想，反正到最后的胜负局面我能描绘出他对应的情况，那我可不可以倒推：
如果red想赢面更大一点，他肯定时从后往前看的，他看到了对应对都自己有利的局面，如果当前没有办法看到让自己有利的局面，他就会往后看，直到看到对自己有利的情况便执行，如果看完了都还没看到对自己有利的，那则局比赛对于red来说必定时输的；

**4.那这里又要问了，red时怎么得出当前这一层对自己时有利的，我刚刚不是在第一点说了吗，因为当前时red下先手，则就是看第一种red会赢的情况；从最大到最小的层级看，如果当前层的数字数量时奇数时，red肯定时会赢的；否则就往后看，直到发现当前层时奇数；如果看完了还没有，那red必输

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