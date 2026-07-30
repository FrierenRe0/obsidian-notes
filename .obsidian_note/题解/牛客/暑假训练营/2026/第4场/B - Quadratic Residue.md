## 相关链接：
题卷查看：[chinese.pdf](https://uploadfiles.nowcoder.com/files/20260729/999991351_1785297491535/chinese.pdf)
题目链接：[B-Quadratic Residue_2026牛客暑期多校训练营4](https://ac.nowcoder.com/acm/contest/133879/B)

## 评价：数学题


## 题目大意：

![](Pasted%20image%2020260730182518.png)

## 核心数学推导与证明

为了构造出一组通解，我们可以尝试简化条件。

### 1. 约束简化假设

从你的代码注释中可以看到一个非常精妙的启发式简化：**令 $x_1 = x_2 = x$**。

如果 $x_1 = x_2 = x$，那么原条件可改写为：

- $1 \le x < \min(p, q)$
    
- $x^2 \equiv p \pmod q$
    
- $x^2 \equiv q \pmod p$


### 2. 将同余式转化为等式

由于 $x < q$，在正整数范围内，由 $x^2 \equiv p \pmod q$ 我们可以引入一个最简单的情况，即直接让恒等式成立。令：

$$x^2 - p = q$$

也就是说，我们主动让 $q$ 成为 $x^2 - p$ 的值。此时，由于 $q = x^2 - p$，显然有：

$$x^2 = q + p \implies x^2 \equiv p \pmod q$$

第一个同余式自动满足。

现在我们将 $q = x^2 - p$ 代入第二个同余式 $x^2 \equiv q \pmod p$ 中：

$$x^2 \equiv (x^2 - p) \pmod p$$

由于 $x^2 - p \equiv x^2 \pmod p$，这个同余式在**任何**满足 $q = x^2 - p$ 的情况下都是**恒成立**的！

因此，只要我们选定一个 $x$，令 $q = x^2 - p$，那么在数学上，这两个同余方程就已经完全满足了。

### 3. 求解边界范围（关键不等式）

我们还需要满足范围约束。在 $x_1 = x_2 = x$ 且 $q = x^2 - p$ 的假设下，约束条件为：

$$1 \le x < p \quad \text{且} \quad 1 \le x < q$$

因为 $q = x^2 - p$，我们将 $q$ 代入第二个不等式 $x < q$ 中：

$$x < x^2 - p \implies x^2 - x - p > 0$$

这是一个关于 $x$ 的一元二次不等式。我们先求解对应的方程 $x^2 - x - p = 0$。根据求根公式，其正根为：

$$x_0 = \frac{1 + \sqrt{1 + 4p}}{2}$$

为了使 $x^2 - x - p > 0$ 成立，正整数 $x$ 必须严格大于这个正根：

$$x \ge \left\lfloor \frac{1 + \sqrt{1 + 4p}}{2} \right\rfloor + 1$$

这里的x满足不等式$x < q$第一个条件；
但是，我们还需要满足第一个不等式 $x < p$：($2 <= p <= 10^9$)

$$\left\lfloor \frac{1 + \sqrt{1 + 4p}}{2} \right\rfloor + 1 < p$$

可得，对于所有 $p \ge 4$，都有 $\sqrt{p} + 1 < p$ 恒成立。

### 4. 特殊情况处理（边界小数据）

由于上述不等式链的放缩在 $p$ 极小时可能因为 $x_1=x_2=x$ 的强限制而无解，代码中对 $p=2$ 和 $p=3$ 进行了单独的手工特判赋值（这也符合样例或暴力搜索出的可行解）：

- 当 $p = 2$ 时，输出 `12 1 71`。验证：
    
    - $12^2 = 144 \equiv 2 \pmod{71}$（满足）
	    
    - $1^2 = 1 \equiv 71 \equiv 1 \pmod 2$（满足）
    
- 当 $p = 3$ 时，输出 `4 2 13`。验证：
    
    - $4^2 = 16 \equiv 3 \pmod{13}$（满足）
	    
    - $2^2 = 4 \equiv 13 \equiv 4 \pmod 3$（满足）
    
- 对于 $p \ge 4$，公式法解出的 $x_1 = x_2 = x$ 与 $q = x^2 - p$ 保证 100% 正确且不越界。

## 代码如下：

```c++
#include <bits/stdc++.h>  
using namespace std;  
#define int long long
#define endl '\n'
const int mod = 1e9 + 7;
const int MOD = 998244353;

void ap() {
	int p; cin >> p;
    
    if (p == 2) {
        cout << "12 1 71" << endl;
        return;
    }
    
    if (p == 3) {
        cout << "4 2 13" << endl;
        return;
    }
    
    //1 <= x1 = x2 = x < min(p, q);
    //x ^ 2  = p + q;
    //x < q, x ^ 2 > x + p;
    
    int x1 = (1 + (int)sqrt(1 + 4 * p)) / 2 + 1;
    int q = x1 * x1 - p;
    
    cout << x1 << ' ' << x1   << ' ' << q << endl;
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