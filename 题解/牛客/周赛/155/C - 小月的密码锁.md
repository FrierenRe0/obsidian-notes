## 相关链接：
[C-小月的密码锁_牛客周赛 Round 155](https://ac.nowcoder.com/acm/contest/138240/C)

## 题目大意：

![](../../../../截图文件/Pasted%20image%2020260803154333.png)


## 思路：
### 1. 状态转移步数计算

对于第 $i$ 个位置，从初始字符 $s[i]$ 到目标字符 $t[i]$ 独立需要的轮转步数是通过 `get_need` 函数计算的：

```
int get_need(char s, chac++
vector<vector<int>> pre(n + 1, vector<int>(5, 0));
for(int i = 0; i < n; i++) {
    pre[i + 1] = pre[i]; // 继承上一个状态
    int need = get_need(s[i], t[i]);
    pre[i + 1][need]++;  // 当前步数计数 +1
}
```

### 第二步：预处理后缀频率

`back[i][x]` 表示在后缀 $s[i \dots n-1]$ 中，所需步数为 $x$ 的位置有多少个。

```
vector<vector<int>> back(n + 1, vector<int>(5, 0));
for(int i = n - 1; i >= 0; i--) {
    back[i] = back[i + 1]; // 继承后面的状态
    int need = get_need(s[i], t[i]);
    back[i][need]++;       // 当前步数计数 +1
}
```

### 第三步：枚举分割点求最大匹配

```
int max_same = 0;
for(int c = 0; c <= n; c++) {
    int max_p = 0, max_q = 0;
    for(int x = 0; x < 5; x++) {
        max_p = max(max_p, pre[c][x]);  // 找出前缀 [0...c-1] 中最多的步数数量
        max_q = max(max_q, back[c][x]); // 找出后缀 [c...n-1] 中最多的步数数量
    }
    max_same = max(max_same, max_p + max_q); // 更新全局最大匹配数
}
cout << n - max_same;
```


## 复杂度分析

- **时间复杂度**：$\mathcal{O}(n)$
    
    - 预处理前缀和后缀数组需要遍历字符串各一次，复杂度为 $\mathcal{O}(5 \cdot n) = \mathcal{O}(n)$。
        
    - 枚举分割点 $c$ 循环 $n+1$ 次，内层循环固定为 5 次，复杂度同样为 $\mathcal{O}(5 \cdot n) = \mathcal{O}(n)$。
        
    - 因此，整体时间复杂度为线性，能够轻松通过 $n \le 10^5$ 级别的数据。
    
- **空间复杂度**：$\mathcal{O}(n)$
    
    - 开辟了 `pre` 和 `back` 两个大小为 $(n+1) \times 5$ 的二维矢量，空间开销极小且稳定。




## 代码如下：
```c++
#include <bits/stdc++.h>
using namespace std;
#define int long long

int get_need(char s, char t) {
    return (t - s + 5) % 5;
}

void solve() { 
    int n;
    cin >> n;
    string s, t;
    cin >> s >> t;
    
    vector<vector<int>> pre(n + 1, vector<int>(5, 0));
    for(int i = 0; i < n; i++) {
        pre[i + 1] = pre[i];
        int need = get_need(s[i], t[i]);
        pre[i + 1][need]++;
    }
    
    vector<vector<int>> back(n + 1, vector<int>(5, 0));
    for(int i = n - 1; i >= 0; i--) {
        back[i] = back[i + 1];
        int need = get_need(s[i], t[i]);
        back[i][need]++;
    }
    
    int max_same = 0;
    
    for(int c = 0; c <= n; c++) {
        int max_p = 0, max_q = 0;
        for(int x = 0; x < 5; x++) {
            max_p = max(max_p, pre[c][x]);
            max_q = max(max_q, back[c][x]);
        }
        max_same = max(max_same, max_p + max_q);
    }
    
    cout << n - max_same;
}

signed main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);
    int T = 1;
    while (T--) {
        solve();
    }
    return 0;
}
```