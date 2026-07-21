https://codeforces.com/gym/695065/problem/A
![配图](../../../截图文件/Pasted%20image%2020260531211250.png)

**这题的关键就在于对于两个结界不重合，一定满足： x2 + y2 - r2 > x1 + x1 + r1**

## 这里就有两种思考方式：

**1.** **区间最大不重叠调度问题** → 贪心

这是这种典型的 #区间问题
**思考**：
这里我们把结界理解，转化为一个区间，现在给定你n个区间，题目想要的情况为：怎么找到最大不重合的区间数量？
讲到这里基本就是板子了 ----> 贪心思想（对区间进行右端点排序）
问：
1.我按最早结束贪心选，会不会错过 对于可选点“选更少前面、换更多后面” 的更优解？/ 为什么是右端点排序？
因为这里我们要找到的情况是找出最大不重合的区间数量，对于每一个区间，我们应该是看从早到晚的“结束点”，这样对于后面的节点选择可以留下当前最大的空间取下个区间，这样每次都是最优子问题，符合贪心，不需要存在dp的困惑，这里可以用数学方法证明：

---
**数学证明**：
假设对于某个区间集合，存在一个绝对完美的**全局最优解 $O$**（这个解包含了最多的不重叠区间）。我们将这个最优解里的区间按照结束时间从早到晚排序：
$$O = \{i_1, i_2, \dots, i_m\}$$
同时，我们的**贪心解 $A$** 按照“结束时间最早”的策略，选出了一系列区间：
$$A = \{j_1, j_2, \dots, j_k\}$$
我们要证明的是：贪心解的数量和全局最优解的数量一样多，即 $k = m$。

### 第一步：贪心选择永远不比最优解差（局部替换）

对比最优解的第一个区间 $i_1$ 和贪心解的第一个区间 $j_1$：
- 根据贪心策略，$j_1$ 是所有可选区间中**结束时间最早**的。
- 因此，$j_1$ 的结束时间一定小于或等于 $i_1$ 的结束时间（即 $f(j_1) \le f(i_1)$）
如果我们将最优解 $O$ 中的 $i_1$ 暴力替换成 $j_1$，会发生什么？
因为 $j_1$ 比 $i_1$ 更早结束，所以 $j_1$ 绝对不会和最优解后面的区间（$i_2, i_3, \dots$）产生重叠。替换后的新集合 $O'$ 依然是一个**合法的、不重叠的、数量没变**的最优解。
### 第二步：数学归纳法（一步步逼近）


以此类推，假设前 $r$ 个区间我们都成功替换了，使得贪心解和某个最优解一致，且满足 $f(j_r) \le f(i_r)$
现在看第 $r+1$ 个区间：
- 在选完 $j_r$ 之后，贪心算法挑了 $j_{r+1}$（它是剩下所有兼容区间里结束最早的）。
- 而最优解挑了 $i_{r+1}$。
- 因为 $f(j_r) \le f(i_r)$，所以留给贪心算法的选择范围**大于或等于**留给最优解的选择范围。
- 因此，贪心挑出来的 $j_{r+1}$ 结束时间必然也小于或等于 $i_{r+1}$（即 $f(j_{r+1}) \le f(i_{r+1})$）。

我们再次把 $i_{r+1}$ 替换成 $j_{r+1}$，新解依然合法。

### 第三步：得出结论

通过这种“替换”，我们可以把全局最优解 $O$ 中的区间一个一个换成贪心解 $A$ 中的区间，期间**没有破坏任何合法性，也没有减少区间的数量**。

如果最优解 $O$ 容纳了 $m$ 个区间，贪心解 $A$ 也绝对有能力容纳 $m$ 个区间。由于 $O$ 已经是最大数量了，所以贪心解的数量 $k$ 必然等于 $m$。

### 再证明：为什么 $k$ 不能小于 $m$ ？

我们用刚才的“推导链”来看看，当贪心算法选到第 $k$ 个（也就是它的最后一个）区间时，发生了什么。
根据前面的归纳法，我们已经证明了每一步贪心选出的区间都比最优解结束得更早：
- $f(j_1) \le f(i_1)$
- $f(j_2) \le f(i_2)$
- ...
- $f(j_k) \le f(i_k)$

这说明：**当贪心算法选完第 $k$ 个区间 $j_k$ 时，它所消耗的时间，比最优解选完第 $k$ 个区间 $i_k$ 时所消耗的时间还要少（或一样）。**
#### 如果 $k < m$ 会发生什么？
如果 $k < m$，说明最优解里不仅有第 $k$ 个区间，后面**至少还有一个**第 $k+1$ 个区间（即 $i_{k+1}$）。
既然最优解能容纳 $i_{k+1}$，说明 $i_{k+1}$ 的开始时间一定在 $i_k$ 的结束时间之后：
$$\text{开始时间}(i_{k+1}) \ge f(i_k)$$
再结合我们的贪心结论 $f(j_k) \le f(i_k)$，我们利用传导性可以得出：
$$\text{开始时间}(i_{k+1}) \ge f(j_k)$$
> **这句话的意思是**：最优解里的第 $k+1$ 个区间（$i_{k+1}$），它的开始时间同样也在贪心第 $k$ 个区间的结束时间之后！
这意味着，在贪心算法选完 $j_k$ 之后，整个区间集合里**至少还剩下一个合法的、不冲突的区间 $i_{k+1}$ 可以被选择**。
### 致命冲突
- **根据贪心算法的定义**：只要还有不冲突的区间可以选，贪心算法就**一定会继续选下去**，绝不会在第 $k$ 个就停止。
- **但现实是**：贪心算法在第 $k$ 个区间就停止了（因为没有剩余兼容区间了）。
这产生了矛盾！
因此，“最优解里还能剩下第 $k+1$ 个区间（即 $k < m$）”的假设是绝对不可能成立的。

---
**所以**：
因为结束越早，**剩下时间越多**，后面能塞的就**越多**！
 贪心的核心目的：
**给后面 “腾最大空间”，让总数最多！就是能选就选**
（但是这里区间带有权值，就不是贪心，而是dp）

```c++
#include <bits/stdc++.h>
using namespace std;
#define int long long
const int mod = 1e9 + 7;
const int N = 3e5 + 7;

int fast_io = []()
{
    ios::sync_with_stdio(false);
    cin.tie(nullptr);
    cout.tie(0);
    return 0;

}();

int init = []()
{
    return 0;
}();

bool cmp(const vector<int>& a,const vector<int>& b) {
    if(a[1] != b[1]) return a[1] < b[1];
    else return a[0] <= b[0];
}

void solve() {
    int n;
    cin >> n;
    vector<int> r(n);
    for(int i = 0;i < n;i++) cin >> r[i];
    vector<vector<int>> a(n,vector<int>(2));
    a[0][0] = a[0][1] = 0;
    for(int i = 1;i < n;i++) {
        int dx,dy;
        cin >> dx >> dy;
        a[i][0] = a[i - 1][0] + dx;
        a[i][1] = a[i - 1][1] + dy;
    }

    vector<vector<int>> s(n,vector<int>(2));
    for(int i = 0;i < n;i++) {
        int all = a[i][0] + a[i][1];
        s[i][0] = all - r[i];
        s[i][1] = all + r[i];
    }

    sort(s.begin(),s.end(),cmp);
    int ans = 1;
    int ed = s[0][1];
    for(int i = 1;i < n;i++) {
        if(s[i][0] > ed) {
            ans++;
            ed = s[i][1];
        }
    }
    cout << ans << endl;
    return;

}

signed main() {
    int T = 1;
    cin >> T;
    while (T--) {
        solve();
    }
    return 0;
}
```


**2.** 转化为类似于LIS（最大递增子区间）问题
**这里思路可能不太一样因为要满足x2 + y2 - r2 > x1 + x1 + r1**
这里我令：
`t = x + y - r;
`s = x + y + r;
看成： t > s;
这里就有点想到LIS了，只不过LIS是看一个信息点，这里是两个信息点；
回忆：LIS中的 `f[i]` 表示的是在以 i 结尾的最大递增子序列，找出对于前面子问题a[i] > a[j]的情况中的max,在加1；
这里就是在以 同样是以i结尾，找出对于前面子问题t[i] > s[j]的情况中的max,在加1；

```c++
#include <bits/stdc++.h>
using namespace std;
#define int long long
vector<int> tree;
void update(int i, int add)
{
    for (; i < tree.size(); i += i & -i)
    {
        tree[i] = max(tree[i], add);
    }
}
int pre_max(int i)
{
    int sum = 0;
    for (; i > 0; i &= (i - 1))
    {
        sum = max(sum, tree[i]);
    }
    return sum;
}

void solve()
{
    tree.clear();
    int n;
    cin >> n;
    vector<int> r(n + 1);
    for (int i = 1; i <= n; i++) cin >> r[i];
   
    vector<int> x(n + 1), y(n + 1);
    x[1] = 0, y[1] = 0;
    for (int i = 2; i <= n; i++)
    {
        int dx, dy;
        cin >> dx >> dy;
        x[i] = x[i - 1] + dx;
        y[i] = y[i - 1] + dy;
    }

    vector<int> s(n);
    vector<int> t(n);
    for (int i = 1; i <= n; i++)
    {
        t[i - 1] = x[i] + y[i] - r[i];
        s[i - 1] = x[i] + y[i] + r[i];
    }

    set<int> st(s.begin(), s.end());
    unordered_map<int, int> mp;
    int idx = 1;
    for (auto &x : st)
    {
        mp[x] = idx++;
    }
    idx--;
    tree.resize(idx + 1);
    int mx = 1;
    for (int i = 0; i < n; i++)
    {
        int h = *st.lower_bound(t[i]);
        int node = mp[h];
        int cur = pre_max(node - 1);
        update(mp[s[i]], cur + 1);
        mx = max(mx, cur + 1);
    }
    cout << mx << '\n';
}

signed main()
{
    ios::sync_with_stdio(false);
    cin.tie(0);
    int t;
    cin >> t;
    while (t--)
        solve();
    return 0;
}
```
