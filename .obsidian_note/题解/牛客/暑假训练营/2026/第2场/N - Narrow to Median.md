## 相关链接：
题卷查看：[2026_暑期多校_2.pdf](file:///C:/Users/l'x/Downloads/2026_%E6%9A%91%E6%9C%9F%E5%A4%9A%E6%A0%A1_2.pdf)
题目链接：[N-Narrow to Median_2026牛客暑期多校训练营2](https://ac.nowcoder.com/acm/contest/133877/N)
## 题目大意：

![](../../../../../截图文件/Pasted%20image%2020260725141031.png)

## 本题大意：

$\textbf{从一个长度为 n 的数组中，任意选取 k 个元素，将这 k 个元素改为按照非递减排序的中位数}$
$\textbf{在恰好进行有一次操作后，求出数组所有元素和的最大可能值}$

## 思路：
### 一 . 本题看似任意取k个数，但是本题的逻辑早有暗示
对于一个非递减的k个元素
- 对于中位数左侧的元素肯定是“正”贡献（贡献 $>= 0$）
- 对于中位数右侧的元素肯定是“负”贡献（贡献 $<= 0$）

#### 思考，当我在一个排好序的数组中，左，右侧数据的取值都存在一个单调的趋势，为什么 ？？

$\textbf{1. 先看左侧：}$
$\textbf{因为找到最大贡献的时候，左侧的元素这么取才可以发挥他的最大价值？}$
$\textbf{→ 是不是我在取在数组中最小的几个数就可以了，也就是说对于左侧的元素选取是固定的}$

$\textbf{2. 再看右侧：}$
$\textbf{这里和左侧就有有点反这来了，但是同样思考，找到最大贡献的时候，右侧的元素这么取才可以抑制他的负价值？}$
$\textbf{解释：是不是我选一写离中位数越近越好的数字就行，拿在排好序中的数组中不就是一段连续区间吗}$

$\textbf{3. 知道这么选择左，右侧数，我现在思考如果找到最大的贡献情况：}$
- $\textbf{因为左侧的贡献在我中位数增加时，因选取固定，贡献价值时不断增加的}$
- $\textbf{但是右侧的数据则不一定了；}$

### 二 .  到这里我们就好想了，问题是不是就可以转化成了一个对右侧数据滑动窗口（定长型）维护最大情况，后面就是代码实现了，具体细节代码展示：

代码如下：
```c++
#include <bits/stdc++.h>  
using namespace std;  
#define int long long
#define endl '\n'
const int mod = 1e9 + 7;
const int MOD = 998244353;


void solve() {
    int n,k;
    cin >> n >> k;
    vector<int> a(n);
    int sum = 0;
    for(int i = 0;i < n;i++) {
        cin >> a[i];
        sum += a[i];
    }
    sort(a.begin(),a.end());
    
    int left_len = (k - 1) / 2;
    int right_len = k - left_len;
    
    int left_sum = 0,right_sum = 0;
    for(int i = 0;i < left_len;i++) {
        left_sum += a[i];
    }
    int mx = LLONG_MIN;
    for(int i = left_len;i < n;i++) {
        right_sum += a[i];
        if(i < k - 1) continue;
        
        int l = i - right_len + 1;
        //这里容易出现浮点数误差，可以这样特殊处理 
        int change_sum = (k & 1 ? a[l] * k : (k / 2) * (a[l] + a[l + 1]));   

        int add = change_sum - (right_sum + left_sum);
        mx = max(mx,add);
        
        //移窗
        right_sum -= a[l];
    }
    cout << mx + sum << endl;
}

signed main() {
	ios::sync_with_stdio(false);
	cin.tie(nullptr);
	int T = 1;
	cin >> T;
	while (T--) {
		solve();
	} 
    return 0;
}
```







