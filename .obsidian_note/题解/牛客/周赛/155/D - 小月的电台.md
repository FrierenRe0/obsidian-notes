## 相关链接：
[D-小月的电台_牛客周赛 Round 155](https://ac.nowcoder.com/acm/contest/138240/D)

## 评价：
### 经典题
- **数论：数位状态压缩+容斥原理（解决：重复计数问题）**


## 题目大意：

![](../../../../截图文件/Pasted%20image%2020260803155157.png)


## 思路：
### 时间复杂度分析：

直接暴力枚举每对字符串并检查是否有共同的 `1`，时间复杂度为 $\mathcal{O}(n^2 \cdot m)$，在 $n$ 较大时会超时。由于 $m$ 通常较小（由代码中 `1 << m` 可推单，当 $m \le 20$ 时适用状态压缩），我们可以从**状态压缩与容斥原理**的角度切入。

### 1. 状态压缩 (Bitmask)

我们将每个长度为 $m$ 的 01 字符串看作一个二进制数（掩码 `mask`）。第 $j$ 位为 `1` 则将二进制的第 $j$ 位设为 1。
其中01 字符串转化成的掩码对应二进制对应的二进制中，当 `bit[i] == 1` 时，表示第`i`位频道和该电台互联；否则当 `bit[i] == 0` 时，表示第`i`位频道和该电台失联。

### 2. 子集贡献预处理

对于一个字符串，如果它的掩码为 `mask`，那么只要我们选择 `mask` 的任意一个**非空子集** `sub`，这个字符串都可以为该子集贡献一个计数。

代码中使用了经典的**高效子集枚举技巧**：

```c++
for (int i = mask; i > 0; i = (i - 1) & mask) {
    s_cnt[i]++;
}
```

通过这种方式，`s_cnt[sub]` 最终记录了：**有多少个字符串包含二进制子集 `sub` 所代表的所有 `1`**。

### 3. 组合计数与容斥原理

如果某个子集 `sub` 拥有 $C$ 个字符串，那么在这 $C$ 个字符串中任意选出两辆，它们都**至少**在 `sub` 所对应的那些位置上同时为 `1`。这一步可以产生 $\frac{C \times (C - 1)}{2}$ 对字符串。

但是，如果直接将所有子集的组合数加起来，会导致**重复计数**。例如：

- 字符串 A 和 B 在第 0 位和第 1 位都是 `1`。
    
- 那么在枚举子集 `01`（包含第 0 位）、`10`（包含第 1 位）以及 `11`（包含第 0, 1 位）时，这对 $(A, B)$ 都会被数进去。
    

为了解决这个“重复计数”问题，我们需要引入**容斥原理**：

- 当子集 `sub` 中 `1` 的个数（即特征数）为**奇数**时，我们**加上**它产生的对数。
    
- 当子集 `sub` 中 `1` 的个数为**偶数**时，我们**减去**它产生的对数。


## 代码细化讲解

### 第一步：状态压缩与子集计数

```c++
vector<int> s_cnt(1 << m, 0);

for (int i = 0; i < n; i++) {
    string s;
    cin >> s;
    int mask = 0;
    for (int j = 0; j < m; j++) {
        if (s[j] == '1') {
            mask |= (1 << j);
        }
    }
    // 高效枚举 mask 的所有非空子集
    for (int i = mask; i > 0; i = (i - 1) & mask) {
        s_cnt[i]++;
    }
}
```

_通过 `(i - 1) & mask` 可以在不遍历无关数字的情况下，只遍历 `mask` 的子集，效率极高。_

### 第二步：容斥原理计算答案


```c++
int ans = 0;
// 枚举全集的所有非空子集
for (int sub = 1; sub < (1 << m); sub++) {
    int C = s_cnt[sub];
    if (C < 2) continue; // 少于2个字符串包含该子集，无法构成对

    int pairs = C * (C - 1) / 2; // 计算组合数
    
    int bits = __builtin_popcount(sub); // 内建函数：计算 sub 二进制中 1 的个数
    
    // 容斥原理：奇加偶减
    if (bits & 1) {
        ans += pairs;
    } else {
        ans -= pairs;
    }
}
cout << ans << "\n";
```

## 复杂度分析

- **时间复杂度**：
    
    - 将字符串转化为 `mask` 需要 $\mathcal{O}(n \cdot m)$。
        
    - 遍历子集部分：对于每个掩码，其子集数量为 $2^{\text{popcount}(mask)}$。最坏情况下（所有字符串全为 `1`），复杂度为 $\mathcal{O}(n \cdot 2^m)$。如果从全局来看，这相当于遍历所有二进制状态的子集，总操作次数的理论上界由 $\sum_{i=0}^{m} \binom{m}{i} 2^i = 3^m$ 决定。但在此题的代码逻辑里，是对每个输入的 $n$ 单独枚举子集，故实际复杂度上限为 $\mathcal{O}(n \cdot 2^m)$。
        
    - 后半部分容斥循环固定为 $\mathcal{O}(2^m)$。
        
    - **总体时间复杂度**约为 $\mathcal{O}(n \cdot 2^m)$。当 $m$ 在 15 到 20 左右时，此算法非常高效。
    
- **空间复杂度**：$\mathcal{O}(2^m)$
    
    - 仅使用了一个大小为 $2^m$ 的一维数组 `s_cnt` 来存储各状态的频数。


## 代码如下：
```c++
#include <bits/stdc++.h>
using namespace std;
#define int long long

void solve() {
    int n, m;
    cin >> n >> m;

    vector<int> s_cnt(1 << m, 0);

    for (int i = 0; i < n; i++) {
        string s;
        cin >> s;
        int mask = 0;
        for (int j = 0; j < m; j++) {
            if (s[j] == '1') {
                mask |= (1 << j);
            }
        }

        for (int i = mask; i > 0; i = (i - 1) & mask) {
            s_cnt[i]++;
        }
    }

    int ans = 0;

    for (int sub = 1; sub < (1 << m); sub++) {
        int C = s_cnt[sub];
        if (C < 2) continue; 

        int pairs = C * (C - 1) / 2;
        
        int bits = __builtin_popcount(sub);
        
        if (bits & 1) {
            ans += pairs;
        } else {
            ans -= pairs;
        }
    }

    cout << ans << "\n";
}

signed main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);
    
    int T = 1;
    // cin >> T; 
    while (T--) {
        solve();
    }
    return 0;
}
```