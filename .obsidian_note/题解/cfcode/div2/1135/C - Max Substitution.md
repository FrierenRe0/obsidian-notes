## 相关链接 ：

- 题目链接 ：[Problem - C - Codeforces](https://codeforces.com/contest/2285/problem/C)


## 题目：

You are given an array $a$ of length $n$. In one operation, you may replace the **unique** largest element in $a$ with any strictly smaller positive integer (i.e., any integer $x$ such that $1 \le x \lt \max(a)$). You can only perform this operation if $a$ has a unique largest element.

Determine the number of distinct$^{\text{∗}}$ arrays that can be obtained by performing the operation any number of times (possibly zero). Since the answer may be large, output it modulo $10^9+7$.

$^{\text{∗}}$Two arrays $a$ and $b$ are considered distinct if they differ in at least one position, i.e., there exists an index $i$ such that $a_i \ne b_i$.

**Input**

Each test contains multiple test cases. The first line contains the number of test cases $t$ ($1 \le t \le 1000$). The description of the test cases follows.

The first line of each test case contains a single integer $n$ ($1 \le n \le 2 \cdot 10^5$) — the length of the array.

The second line of each test case contains $n$ integers $a_1, a_2, \ldots, a_n$ ($1 \le a_i \le 10^9$) — the elements of the array.

It is guaranteed that the sum of $n$ over all test cases does not exceed $2\cdot 10^5$.


## 思路 :

### 🧠 核心思路与推导

这是一个典型的由“操作过程”推导“最终组合状态”的题目。

由于我们要判定的是最终的**数组状态**，而数组中每个元素的位置是固定的，每次修改也都是独立地针对某个位置。为了方便观察“最大值”的降级过程，我们可以**先将数组从大到小降序排列**，即 $a_0 \ge a_1 \ge a_2 \dots \ge a_{n-1}$。_(注：排序仅用于推导每个数值可变的上界，操作实际对应在原位置，因此状态的组合数完全等价)_

我们按“被修改元素的深度”对所有能达到的最终数组进行互斥分类： 定义**深度 $k$**：意味着数组中排名前 $k+1$ 大的元素（即 $a_0, a_1, \dots, a_k$）**都被修改过了**，而从 $a_{k+1}$ 开始的元素依然保持原样。

1. **深度为 0（没有任何元素被修改）**： 这就是原数组本身，只有 **1** 种状态。
    
2. **深度为 1（仅 $a_0$ 被修改，其它不变）**： $a_0$ 是最初的唯一最大值，它能被改成 $[1, a_0 - 1]$ 中的任意正整数。 只要我们就此停下，就会产生 $(a_0 - 1)^1$ 种不同的数组。
    
3. **深度为 $k+1$（前 $k+1$ 个元素都被修改了）**： 要想修改 $a_k$，前提是 $a_k$ 必须暴露出成为“唯一的最大值”。 这就要求排在它前面的所有元素 $a_0, a_1, \dots, a_{k-1}$ 都已经被“削弱”成了**严格小于** $a_k$ 的值（即 $\le a_k - 1$）。 然后，我们再对 $a_k$ 动手，让 $a_k$ 也变成一个严格小于它自己的值（$\le a_k - 1$）。 此时，前 $k+1$ 个元素的值全部落在了区间 $[1, a_k - 1]$ 中！
    
    **神奇的结论**：只要保证每个数最终都在区间 $[1, a_k - 1]$ 内，**任意一种**组合 $(x_0, x_1, \dots, x_k)$ 都能被合法地操作出来！因为我们总可以优先选择把当前的唯一最大值降级为目标组合中最大的那个数，完美进行“唯一最大值”的接力。 因此，前 $k+1$ 个元素任意取 $[1, a_k - 1]$ 中的值，总方案数为 $(a_k - 1)^{k+1}$。 因为这些状态中 $a_{k+1}$ 是没动过的，所以它们与之前深度的状态**完全不重叠**。
    
4. **致命的“死锁”条件**： 如果在我们降序排列的数组中，出现了 $a_i == a_{i+1}$，会发生什么？ 当排在前面的元素都被削弱后，最终 $a_i$ 和 $a_{i+1}$ 会同时暴露出来成为最大值。因为它们相等，最大值**不再唯一**！操作会直接“卡死”，不仅这两个元素无法修改，它们后面的所有元素也都永远无法被修改了。 因此，一旦遇到 $a_i == a_{i+1}$，统计就必须直接终止（`break`）。


### ⏱️ 复杂度分析

- **时间复杂度**：
    
    - 排序数组耗时 $O(N \log N)$。
        
    - 循环遍历数组 $N$ 次，每次内部调用快速幂函数 `ksm` 耗时 $O(\log N)$。
        
    - 单组测试用例总时间复杂度为 $O(N \log N)$。在题目限定的 $\sum n \le 2 \cdot 10^5$ 范围内，性能极佳，运行时间绰绰有余。
        
- **空间复杂度**：
    
    - 主要消耗为存储数组的 `vector<int> a(n)`，空间复杂度为 $O(N)$。


## 代码展示：

```c++
#include <bits/stdc++.h>
using namespace std;
#define int long long
#define endl '\n'
#define ctz __builtin_ctzll         // 二进制表示中末尾连续0的个数
#define clz __builtin_clzll         // 二进驻表示中先导0的个数
#define count1 __builtin_popcountll // 二进制表示中1的个数
const int mod = 1e9 + 7;
const int MOD = 998244353;
const int N = 2e5 + 7;

int ksm(int di,int ci) {
    di %= mod;
    int res = 1;
    while(ci) {
        if(ci & 1) res = res * di % mod;
        ci /= 2;
        di = di * di % mod;
    }
    return res;
}

void solve() { 
    int n;
    cin >> n;
    vector<int> a(n);
    for(auto& x : a) cin >> x;
    sort(a.rbegin(),a.rend());

    int ans = 1;
    for(int i = 0;i < n;i++) {
        if(i < n - 1 && a[i + 1] == a[i]) break;
        ans = (ans + ksm(a[i] - 1,i + 1)) % mod;
    }
    cout << ans << endl;
    return;
}
signed main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);
    cout.tie(0);
    int T = 1;
    cin >> T;
    while (T--) {
        solve();
    }
    return 0;
}
```