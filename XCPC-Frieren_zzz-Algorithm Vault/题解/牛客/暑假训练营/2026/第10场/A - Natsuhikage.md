## 相关链接：
- 题卷查看：[第10场题卷](../题卷/第10场.pdf)
- 题目链接： [A-Natsuhikage_2026牛客暑期多校训练营10](https://ac.nowcoder.com/acm/contest/133885/A)

## 评价：
几何数学题

## 题目大意：


![](../../../../../截图文件/Pasted%20image%2020260820174242.png)

如图所示，一根长度固定为 $R$ 的硬杆一端固定在原点 $O$，可在角度区间 $[\theta_A, \theta_B]$ 内旋转。

太阳光以平行光束（方向为矢量 $\vec{S} = (s_x, s_y)$）照射到地面（$X$ 轴）。硬杆 $OP$ 会在地面上投射出影子 $OQ$。要求找到当硬杆在允许旋转范围内时，影子长度 $\vert{}OQ\vert{}$ 的**最小值**和**最大值**。


## 思路：

杆的倾角为 $\theta$（即 $P = (R \cos\theta, R \sin\theta)$），太阳光的方向向量为 $(s_x, s_y)$。

根据几何关系，过点 $P$ 沿光线方向作直线交 $X$ 轴于影子端点 $Q(x_Q, 0)$

影子端点的横坐标（即影长）：

$$f(\theta) = \left\vert{} R \cos\theta - R \sin\theta \cdot \frac{s_x}{s_y} \right\vert{}$$
利用三角函数辅助角公式化简 $f(\theta)$ 内的表达式：

$$g(\theta) = R \left( \cos\theta - \frac{s_x}{s_y} \sin\theta \right)$$

对于影子的极值点：

- **影长极小值点（影长 $f(\theta) = 0$）**：
    
    当杆的方向与光线方向平行时，杆没有影子，即：
    $$\cos\theta - \frac{s_x}{s_y} \sin\theta = 0 \implies \tan\theta = \frac{s_y}{s_x} \implies \theta_0 = \text{atan2}(s_y, s_x)$$

- **影长极大值点**：
    
    求 $g'(\theta) = 0$：
    $$-R \sin\theta - R \frac{s_x}{s_y} \cos\theta = 0 \implies \tan\theta = -\frac{s_x}{s_y} \implies \theta_1 = \text{atan2}(-s_x, s_y)$$
### 存在点枚举

由于影长函数 $f(\theta)$ 在区间内是连续的，它的最大值和最小值只可能在以下**候选点**中取得：

1. 旋转边界的**两个端点** $\theta_A, \theta_B$。
    
2. **零点（影子最小）**：$\theta = \text{atan2}(s_y, s_x)$（需正规化到区间角度，并判断是否落在 $[\theta_A, \theta_B]$ 内）。
    
3. **极值点（影子最大）**：$\theta = \text{atan2}(-s_x, s_y)$（需正规化到区间角度，并判断是否落在 $[\theta_A, \theta_B]$ 内）。


**注意角度跨越问题**：

使用 `atan2(y, x)` 算出的角度在 $[-\pi, \pi]$，在判断极值点是否在区间 $[\theta_A, \theta_B]$ 内部时，需要将角度统一调整（例如加上或减去 $\pi / 2\pi$），尽量统一到 $0 / 2\pi$，保证逆时针旋转区间的连续性。

## 代码如下：
```c++
#include <bits/stdc++.h>
using namespace std;

void solve() { 
    double sx, sy, ax, ay, bx, by;
    cin >> sx >> sy >> ax >> ay >> bx >> by;

    double r = sqrt(ax * ax + ay * ay);
    double l = atan2(ay, ax);
    double r_a = atan2(by, bx);
    
    //这里求：影子的长度
    auto f = [&](double rad) {
        return abs(r * cos(rad) - (sx / sy) * r * sin(rad));
    };
    
    //端点
    vector<double> p = {l, r_a};
    
    //这里看看两个极值点 
    //len影子 : 0
    double t_mi = atan2(sy, sx);
    if (t_mi > l && t_mi < r_a) p.push_back(t_mi);
    //len影子 : max
    double t_mx = atan2(-sx, sy);
    while(t_mx < 0) t_mx += M_PI;
    if (t_mx > l && t_mx < r_a) p.push_back(t_mx);

    double mn = 1e18, mx = -1e18;
    for (double rad : p) {
        double len = f(rad);
        mn = min(mn, len);
        mx = max(mx, len);
    }

    cout << setprecision(9) << mn << " " << mx << endl;
}

signed main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);
    int T;
    cin >> T;
    while (T--) solve();
    return 0;
}
```