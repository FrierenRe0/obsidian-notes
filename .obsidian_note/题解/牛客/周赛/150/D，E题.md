[D-小红的正方形_牛客周赛 Round 150](https://ac.nowcoder.com/acm/contest/137073/D)
这里的题解在 一些经典问题集合 中

[E-小红的异或和_牛客周赛 Round 150](https://ac.nowcoder.com/acm/contest/137073/E)

### 这里我先放个前置知识：（也是我在打比赛过程中由于不了解这个点导致差点ac）

**1. 当a = 2的整数次幂时：
(a * b) ^ (a * c) ^ (a * d) …… ^ (a * x) = a * (b ^ c ^ d …… ^ x); 成立

**2. 对于从0到某个整数 x（x >= 0) 的异或和有如下规律：
$$ \begin{cases}\text{if(x \% 4 == 0) return x;}\\ \text{else if(x \% 4 == 1) return 1;}\\ \text{else if(x \% 4 == 2) return x + 1;}\\ \text{else return 0;} \end{cases} $$

### 题目大况：
  
对于给定的整数x = 2的k次幂(k >= 0)，小红认为一个整数是好数，当且仅当其是 x 的倍数。  
给定整数 l , r , x , 请你计算 `[l,r]`范围内所有好数的异或和是多少。

### 思路：
其实这里很容易思考了，这里我在这个范围所需要找到的值，在我不可以暴力的情况下，往往情况都和题目所给定的边界位置有关，这里给定了left and right，题目关于x 的 我是不是可以理解为题目之间差了一个double倍的关系，ok, 现在你有left and right 所得出的double倍关系，那我不就是在这两个double的范围内去找到一段连续的整数吗（left向上取整，right向下取整）；

再根据我的前置知识1, 再 a = 2的整数次幂时；
(a * b) ^ (a * c) ^ (a * d) …… ^ (a * x) = a * (b ^ c ^ d …… ^ x)
那我不就是计算这段连续的整数异或和就行了吗；
 (b ^ c ^ d …… ^ x) = pre_xor(0,x) ^ pre_xor(0,b - 1); 
根据题目所给的数据这里b一定时>= 1的，则b - 1一定是>= 0的，符合条件；
最后输出结果即可；


### 代码详解如下：
```c++
#include <bits/stdc++.h>
using namespace std;
#define int long long
const int mod = 1e9 + 7;
const int N = 3e5 + 7;
int f(int x) {
//     if(x < 0) return 0;
    if(x % 4 == 0) return x;
    else if(x % 4 == 1) return 1;
    else if(x % 4 == 2) return x + 1;
    else return 0;
}
void solve() { 
    int l,r,x;
    cin >> l >> r >> x;
    int left = (l + x - 1) / x;
    int right = r / x;
    
    int z = f(left - 1);
    int zz = f(right);
    cout << (x * (z ^ zz));
    return;
}
signed main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);
    cout.tie(0);
    int T = 1;
    //cin >> T;
    while (T--) {
        solve();
    }
    return 0;
}
```