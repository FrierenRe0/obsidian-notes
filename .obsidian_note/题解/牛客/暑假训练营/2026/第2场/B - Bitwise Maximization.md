这是一道非常经典的线性基（Linear Basis）在异或性质上的变形应用题。你的核心代码逻辑和位运算转化方向完全正确，且切中了问题的最关键突破口——**区分“独立位”与“绑定位”**。

## 题目大意

给定一个包含 $n$ 个非负整数的序列，要求将所有数字划分到两个多重集（Multiset）$S_1$ 和 $S_2$ 中。

令 $X_1$ 为 $S_1$ 中所有元素的异或和，$X_2$ 为 $S_2$ 中所有元素的异或和（空集的异或和为 0）。

求 $X_1 + X_2$ 的最大可能值。

## 核心思路与数理推导

这道题如果直接去枚举集合划分，复杂度是 $O(2^n)$，显然不可行。我们需要从按位贡献的角度去简化贡献公式。

### 1. 全局异或和的常数性质

首先，设整个序列所有元素的总异或和为 $S_{all}$。无论我们怎么划分：

$$X_1 \oplus X_2 = S_{all}$$

这意味着，对于任意一个二进制位 $i$：

- 如果 $S_{all}$ 的第 $i$ 位是 1：说明 $X_1$ 和 $X_2$ 的第 $i$ 位一定互异（一个是 1，一个是 0）。那么无论怎么划分，这一位对最终加法答案 $X_1 + X_2$ 的贡献都是固定的 $2^i$。
    
- 如果 $S_{all}$ 的第 $i$ 位是 0：说明 $X_1$ 和 $X_2$ 的第 $i$ 位一定相同（要么都是 1，要么都是 0）。如果我们能让它们都是 1，那么这一位对答案的贡献就是 $2 \times 2^i$；如果都是 0，贡献就是 0。
    

### 2. 目标转化

基于上面的分析，我们把最终的收益 $X_1 + X_2$ 拆开：

- 对于 $S_{all}$ 为 1 的位：贡献固定为 $S_{all}$。
    
- 对于 $S_{all}$ 为 0 的位：我们要尽可能让 $X_1$（或 $X_2$）在这些位上为 1。如果 $X_1$ 在这些位上的最大异或和为 $M_x$，那么两者的共同贡献就是 $2 \times M_x$。
    

因此，最终的答案公式可以转化为：

$$\text{Max Score} = S_{all} + 2 \times M_x$$

### 3. 如何最大化 $M_x$

现在已知 $M_x = x_1 \& y_1$。

我们直接处理 $M_x$ 不好弄，可以将 $M_x$ 进行转化：

$$M_x = x_1 \& y_1 = x_1 \& (\sim s)$$

这里我来解释一下，为什么存在 $x_1 \& y_1 = x_1 \& (\sim s)$：

对于 $x_1$ 和 $y_1$ 在二进制位运算中无非就是两种情况：

- 当 $\text{bit}[x_1] = \text{bit}[y_1]$ 时，这里此时的 $\text{bit}[x_1] \& \text{bit}[y_1] = \text{bit}[x_1]$，这里对应的 $\text{bit}[S_{all}] = 0$，反之，$\sim \text{bit}[S_{all}] = 1$，这可以推算出 $\text{bit}[x_1] \& (\sim \text{bit}[S_{all}]) = \text{bit}[x_1]$，合理满足上述说法。
    
- 当 $\text{bit}[x_1] \neq \text{bit}[y_1]$ 时，这里此时的 $\text{bit}[x_1] \& \text{bit}[y_1] = 0$，这里对应的 $\text{bit}[S_{all}] = 1$，反之，$\sim \text{bit}[S_{all}] = 0$，这可以推算出 $\text{bit}[x_1] \& (\sim \text{bit}[S_{all}]) = 0$，合理满足上述说法。
    

继续对 $M_x$ 进行转化：

$$M_x = (x_1 \& (\sim s)) \oplus (x_2 \& (\sim s)) \oplus (x_3 \& (\sim s)) \oplus \dots \oplus (x_k \& (\sim s))$$

这里我们把 $(x_i \& (\sim s))$ 看成一个整体 $t$，求解其最大异或和可以用到线性基（对应的时间复杂度为 $O(n \times m)$，这里的 $m$ 对应为 $t$ 的二进制长度，相当于把这里的时间复杂度从 $O(n^2)$ 降到了 $O(n)$）；

之后就是创建线性基 + 求最大异或和的板子了，之后就用代码展示。

### 代码如下：

```c++
#include <bits/stdc++.h>
using namespace std;
#define int long long
const int mod = 1e9 + 7;
const int N = 3e5 + 7;
int bit[30];
void init() {
    for(int i = 0;i < 30;i++) bit[i] = 0;
}
bool insert(int x) {
    for(int i = 29;i >= 0;i--) {
        int cur = (x >> i) & 1;
        if(cur) {
            if(bit[i] == 0) {
                bit[i] = x;
                return true;
            }    
            x ^= bit[i];
        }
    }
    return false;
}
void solve() {
    int n;
    cin >> n;
    int xr_sum = 0;
    vector<int> a(n);
    for(int i = 0;i < n;i++) {
        cin >> a[i];
        xr_sum ^= a[i];
    }
    for(int i = 0;i < n;i++) {
        insert(a[i] & (~xr_sum));
    }
    int mx = 0;
    for(int i = 29;i >= 0;i--) {
        mx = max(mx,mx ^ bit[i]);
    }
    cout << xr_sum + 2 * mx << endl;
    init();
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