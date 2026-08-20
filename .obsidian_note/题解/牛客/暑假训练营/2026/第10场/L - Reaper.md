## 相关链接：
- 题卷查看：[第10场题卷](../题卷/第10场.pdf)
- 题目链接： [L-Reaper_2026牛客暑期多校训练营10](https://ac.nowcoder.com/acm/contest/133885/L)

## 评价：
思维题

## 题目大意：

![](../../../../../截图文件/Pasted%20image%2020260820175654.png)

## 解题思路

因为时间趋近于无限长，最佳策略一定是一个**周期性循环**。

在连续两次激活“幽灵姿态”之间，假设死神一共手动装填了 $x$ 次（$x \ge 0$）：

1. **开火的总子弹数（总开火时间）**：
    
    每一次满弹（包含初始满弹以及 $x$ 次手动装填）都能开火 $m$ 发，因此在本次幽灵姿态 CD 恢复之前，一共发射了：
    $$\text{Shots} = (x + 1) \times m \text{ 发}$$
    
    对应耗费的开火时间为 $(x + 1) \times m$ 秒。
    
2. **这期间消耗的总实际时间**：
    
    - 开火总时间：$(x + 1) \times m$ 秒。
        
    - 手动装填总时间：$x \times r$ 秒。
        
    - 动作总耗时为：$(x + 1) \times m + x \times r$ 秒。
        
    - **CD 约束**：由于两次激活幽灵姿态之间必须间隔至少 $c$ 秒冷却时间，如果上述动作总耗时不足 $c$ 秒，死神需要**发呆/等待**直到 $c$ 秒 CD 结束。
        
        因此，一个循环周期的总时长为：
        $$\text{Time} = \max\Big( (x + 1) \cdot m + x \cdot r,\; c \Big)$$
        
3. **平均射率函数**：
    
    在 $x$ 次手动装填策略下的平均射率为：
    $$g(x) = \frac{(x + 1) \cdot m}{\max\Big( (x + 1) \cdot m + x \cdot r,\; c \Big)}$$

### 函数极大值分析：

- **情况 1**：如果不使用幽灵姿态，纯靠常规开火与手动装填，极限射率为：
    
    $$\text{rate}_{\text{normal}} = \frac{m}{m + r}$$
    
- **情况 2**：如果使用幽灵姿态（循环周期由 $c$ 约束）：
    
    - 当 $(x + 1)m + xr \le c$ 时，分子随着 $x$ 增加而单调递增，分母恒为 $c$。因此 $x$ 越大越好，直到达到临界点。
        
    - 当 $(x + 1)m + xr \ge c$ 时，$g(x) = \frac{(x+1)m}{(x+1)m + xr}$。随着 $x \to \infty$，该值收敛于 $\frac{m}{m + r}$。

**因此，由于 $x$ 的最佳范围很小，我们可以直接利用三分法，或者直接在极小范围内枚举 $x$（$\le 100$） 找到最大值。**


## 代码如下：

```C++
#include <bits/stdc++.h>
using namespace std;
#define int long long

void solve() {
    double m, r, c;
    cin >> m >> r >> c;
    //normal
    double ans = m / (m + r);
    
    //暴力搭配
    for (int x = 0; x <= 100; ++x) {
        double s = (x + 1) * m;
        double time = max((x + 1) * m + x * r, c);
        ans = max(ans, s / time);
    }

    cout << setprecision(12) << ans << endl;
}

signed main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);
    cout.tie(nullptr);
    int T = 1;
    cin >> T;
    while (T--) {
        solve();
    }
    return 0;
}
```