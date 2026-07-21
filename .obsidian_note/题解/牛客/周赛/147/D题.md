[D-小红的子序列_牛客周赛 Round 147](https://ac.nowcoder.com/acm/contest/136224/D)
![[Pasted image 20260608153823.png]] 

**我觉得这题的入手点很新鲜，这里值得分享一下：
思路形成：
这里我刚刚看到题目的时候，觉得有点类似于LIS的类型，但是这里去找符合条件的子序列并不像LIS那样轻松，这里像要找到以i下标结尾的子序列需要两个满足两个条件：
- bi​ 是 bi−1​ 的倍数
- bi / bi−1是一个质数
**这就不是什么数据结构优化可以解决的问题了，这里就没有那某好找了
### 问题来了，那我怎么去找到这个满足条件的子序列，我又不能暴力一个一个往前面找符合条件的子序列？（关键）

**思考：
这里我们就需要思考对应的满足条件了；
看倍数关系：
我想要bi和bi - 1满足该条件，你可以想到什么：

**是不是bi - 1一定是bi的因子；（关键）
-> 其实这种思想已经很常见了，你发现我想要找到的子序列，一定是在bi对应因子中的数，这样我们就把时间复杂度0（n）降到常数级；
(质数条件就不说了，用个埃式筛筛一下就行了)

**然后再对应去找到满足这两个条件的最长子序列；

### 对应代码如下：
```c++
#include <bits/stdc++.h>
using namespace std;
#define int long long
const int mod = 1e9 + 7;
const int N = 1e6 + 1;
vector<int> isprime(N);
vector<int> prime;

int fast_io = []()
{
    ios::sync_with_stdio(false);
    cin.tie(nullptr);
    cout.tie(0);
    return 0;
}();

int init = []()
{
    isprime[0] = isprime[1] = 1;
    for(int i = 2;i < N;i++) {
        if(!isprime[i]) {
            prime.push_back(i);
        }
        for(int j = 0;j < prime.size() && i * prime[j] < N;j++) {
            isprime[prime[j] * i] = 1;
            if(i % prime[j] == 0) break;
        }
    }
    return 0;
}();

vector<int> yizi(int x) {
    vector<int> b;
    for(int i = 1;i * i <= x;i++) {
        if(x % i == 0) {
            b.push_back(i);
            if(x / i != i) b.push_back(x / i);
        }
    }
    return b;
}
void solve() { 
    int n;
    cin >> n;
    vector<int> a(n);
    for(int i = 0;i < n;i++) cin >> a[i];
    vector<int> pre_idx(n,-1);
    map<int,int> mp;
    map<int,int> idx;
    int mx = 0;
    int jilu = 0;
    for(int i = 0;i < n;i++) {
        int x = a[i];
        vector<int> pos = yizi(x);
        int cur_mx = 0;
        for(auto& y : pos) {
            if(mp.count(y) && isprime[x / y] == 0) {
                if(mp[y] > cur_mx) {
                    pre_idx[i] = idx[y];
                    cur_mx = mp[y];
                }
            }
        }
        if(cur_mx + 1 > mx) {
            mx = cur_mx + 1;
            jilu = i;
        }
        if(mp[x] < cur_mx + 1) {
            idx[x] = i;
            mp[x] = cur_mx + 1;
        }
    }
    vector<int> ans;
    ans.push_back(a[jilu]);
    while(pre_idx[jilu] != -1) { 
        ans.push_back(a[pre_idx[jilu]]);
        jilu = pre_idx[jilu];
    }
    reverse(ans.begin(),ans.end());
    cout << ans.size() << endl;
    for(auto& x : ans) cout << x << " ";
    return;
}
signed main() {
    int T = 1;
    //cin >> T;
    while (T--) {
        solve();
    }
    return 0;
}
```
