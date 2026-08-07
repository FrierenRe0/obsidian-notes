## 相关链接：
- 题卷：[第3场题卷](../题卷/第3场.pdf)
- 题目链接：[G-Matrix Marking_2026牛客暑期多校训练营3](https://ac.nowcoder.com/acm/contest/133878/G)

## 评价：
hash统计 + 二维差分

## 题目大意：
![](../../../../../截图文件/Pasted%20image%2020260804174758.png)


## 思路：
### 1. 按数值分组与行聚类

题目要求相同数值的点作为矩形角点，因此首先将网格中数值相等的单元格按值分组。

对于每一种数值：

1. 收集该数值出现的所有坐标，按行号 $r$ 升序排序。
    
2. 同一行的坐标聚类，仅保留该行出现该数值的**最小列号** $\min(c)$ 和**最大列号** $\max(c)$。
    
3. 压缩为按行号升序排列的行信息数组，记第 $i$ 行的行号为 $p[i]$，最小列号为 $mn[i]$，最大列号为 $mx[i]$。


### 2. 前缀/后缀极值与区间覆盖

对于某一行 $p[i]$，如果能与下方某行 $p[j]$（$j > i$）的同值点构成矩形，只需寻找下方出现的**最大列号**是否严格大于当前出现的**最小列号**。

定义：

- **前缀最小列号** $mnl[i] = \min_{k \le i} (mn[k])$：表示前 $i$ 行同值点出现的最小列号；
    
- **后缀最大列号** $mxr[i] = \max_{k \ge i} (mx[k])$：表示第 $i$ 行及以后同值点出现的最大列号。


对于行序列 $p[0 \dots sz2-1]$，遍历每一行 $i$：

1. **当前行 $p[i]$ 的贡献**：
    
    - 上方（前 $i-1$ 行）出现过的最小列号为 $l_1 = mnl[i-1]$，下方（第 $i$ 行及以后）出现过的最大列号为 $r_2 = mxr[i]$。若 $l_1 < r_2$，则在第 $p[i]$ 行，列区间 $[l_1, r_2]$ 被标记。
        
    - 下方（第 $i+1$ 行及以后）出现过的最大列号为 $r_1 = mxr[i+1]$，当前（第 $i$ 行及以前）出现过的最小列号为 $l_2 = mnl[i]$。若 $l_2 < r_1$，则在第 $p[i]$ 行，列区间 $[l_2, r_1]$ 被标记。
        
2. **跨行中间区域（$p[i] < r < p[i+1]$）的贡献**：
    
    - 若第 $i$ 行与第 $i+1$ 行之间存在空行（即 $p[i] + 1 < p[i+1]$），且上方的最小列号 $mnl[i]$ 严格小于下方的最大列号 $mxr[i+1]$；
        
    - 则行区间 $[p[i]+1, p[i+1]-1]$ 内的所有行，在列区间 $[mnl[i], mxr[i+1]]$ 上都会被跨越该区域的矩形覆盖。
    

### 3. 二维差分与还原

利用**二维差分**对所有需要标记的矩形区域进行 `+1` 标记：

- 矩形 $[x_1, x_2] \times [y_1, y_2]$ 在差分数组上的更新为：
    
    - `diff[x1][y1]++`
        
    - `diff[x1][y2 + 1]--`
        
    - `diff[x2 + 1][y1]--`
        
    - `diff[x2 + 1][y2 + 1]++`
    

最后使用**二维前缀和**统计每个位置的被标记次数：若 `diff[i][j] > 0` 输出 `'1'`，否则输出 `'0'`

## 复杂度分析

- **时间复杂度**：$\mathcal{O}(n \cdot m \log(n \cdot m))$
    
    - 遍历网格构建哈希表耗时 $\mathcal{O}(n \cdot m)$；
        
    - 同值点坐标排序总耗时为 $\mathcal{O}(n \cdot m \log(n \cdot m))$；
        
    - 遍历处理与二维前缀和恢复均在 $\mathcal{O}(n \cdot m)$ 内完成。
    
- **空间复杂度**：$\mathcal{O}(n \cdot m)$
    
    - 哈希表存储坐标向量以及 $(n+2) \times (m+2)$ 的二维差分数组占用空间与网格大小同阶。



## 代码如下：

```c++
#include <bits/stdc++.h>
using namespace std;
#define int long long
const int mod = 1e9 + 7;
const int MOD = 998244353;
const int N = 2e5 + 7;

void solve() {
    int n, m;
    cin >> n >> m;
    unordered_map<int, vector<pair<int, int>>> mp;

    for (int i = 0; i < n; i++)
    {
        for (int j = 0; j < m; j++)
        {
            int x;
            cin >> x;
            mp[x].emplace_back(i, j);
        }
    }

    vector<vector<int>> diff(n + 2, vector<int>(m + 2, 0));
    for (auto &[_, a] : mp)
    {
        sort(a.begin(), a.end());
        int sz = a.size();
        vector<int> p, mn, mx;

        for (int i = 0; i < sz; i++)
        {
            int x = a[i].first + 1;
            int y = a[i].second + 1;
            if (p.empty() || p.back() != x)
            {
                p.push_back(x);
                mn.push_back(y);
                mx.push_back(y);
            }
            else
            {
                mn.back() = min(mn.back(), y);
                mx.back() = max(mx.back(), y);
            }
        }

        int sz2 = p.size();
        if (sz2 < 2) continue;
		
		//左右端点的列标min（前缀和），max（后缀和）维护
        vector<int> mnl(sz2), mxr(sz2);
        //min（前缀和）
        mnl[0] = mn[0];
        for (int i = 1; i < sz2; i++)
        {
            mnl[i] = min(mnl[i - 1], mn[i]);
        }
        //max（后缀和）
        mxr[sz2 - 1] = mx[sz2 - 1];
        for (int i = sz2 - 2; i >= 0; i--)
        {
            mxr[i] = max(mxr[i + 1], mx[i]);
        }
		
		//差分标记
        for (int i = 0; i < sz2; i++)
        {
            int l1 = (i > 0) ? mnl[i - 1] : m + 1;
            int l2 = mnl[i];
            int r1 = (i < sz2 - 1) ? mxr[i + 1] : 0;
            int r2 = mxr[i];
            
			//计算对于当前行的贡献
            if (l1 < r2)
            {
                int x = p[i];
                diff[x][l1]++;
                diff[x][r2 + 1]--;
                diff[x + 1][l1]--;
                diff[x + 1][r2 + 1]++;
            }
            if (l2 < r1)
            {
                int x = p[i];
                diff[x][l2]++;
                diff[x][r1 + 1]--;
                diff[x + 1][l2]--;
                diff[x + 1][r1 + 1]++;
            }
            
			//计算对于中空行的贡献
            if (i < sz2 - 1 && p[i] + 1 < p[i + 1])
            {
                if (mnl[i] < mxr[i + 1])
                {
                    int x1 = p[i] + 1;
                    int x2 = p[i + 1];
                    int yL = mnl[i];
                    int yR = mxr[i + 1];
                    diff[x1][yL]++;
                    diff[x1][yR + 1]--;
                    diff[x2][yL]--;
                    diff[x2][yR + 1]++;
                }
            }
        }
    }
	//还原 -> 二维前缀和
    for (int i = 1; i <= n; i++)
    {
        for (int j = 1; j <= m; j++)
        {
            diff[i][j] += diff[i - 1][j] + diff[i][j - 1] - diff[i - 1][j - 1];
        }
    }
	
	//遍历输出
    for (int i = 1; i <= n; i++)
    {
        string res;
        res.reserve(m);
        for (int j = 1; j <= m; j++)
        {
            res += (diff[i][j] > 0 ? '1' : '0');
        }
        cout << res << '\n';
    }
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