
类似题型：[1510. 石子游戏 IV - 力扣（LeetCode）](https://leetcode.cn/problems/stone-game-iv/description/)
## 相关链接
- 题卷：[第3场题卷](../题卷/第3场.pdf)
- 题目链接：[L-Uphill Duel_2026牛客暑期多校训练营3](https://ac.nowcoder.com/acm/contest/133878/L)

## 评价：
一道经典的博弈题
## 题目大意：

![](../../../../../截图文件/Pasted%20image%2020260730154436.png)

## 思路：
### 1. 目的：我们先look一下本题的形式估计时间复杂度，如果对于一个范围在$1 <= q <= 10^5$的查询次数，对于每次的查询时间复杂度一定是一个$o(n)$形式下的查询，则对于每个网格节点状态一定需要先做一次预处理来完成对应时间复杂度的check 

### 2. 基于该目的，我们如何进行预处理 ？？看看他们在博弈中的情况 --> 看看有没有什么特点和规律

1. 每个节点的可以走的路径方向取决于该点四周比当前节点更大的路径方向 - > 找到了状态转移关系（逻辑上等价于状态树中父子节点关系）；
	
2. 现在知道了当前节点取决于该店四周更高节点的状态，也就是说前序节点状态由后序节点状态确定，对应一个逆向过程，这里就是我们 逆向思想 -> （这里再逻辑上等价于再状态树中从下到上的过程 / 或者说是一个递推的过程）的标志了；
	
3. 则现在我们可以先处理后续节点状态，前序节点的节点的状态就自然由后续状态推得
	
4. 如何处理高位状态（后续状态）？（关键规律）
	
	- 先做一下定义：先手为”F“，后手为”S“;
	
	- 如果"F”玩家胜利是一个什么情况？
		会发现：对于“F”作为先手
		
		- 如果在四周所有更高节点存在走偶数路径的，当前节点一定是F胜S负;
		
		- 反则，没有偶数路径全是奇数路径，一定是S胜F负；
	
	
5. 这样我们就把所有的节点做了预处理，只要当前节点的状态定义为奇数，那就是F胜；否则为奇数，就是S胜。


## 代码如下：
```c++
#include <bits/stdc++.h>  
using namespace std;  
#define int long long
#define endl '\n'
const int mod = 1e9 + 7;
const int MOD = 998244353;

int dx[4] = {0, 0, 1, -1}, dy[4] = {1, -1, 0, 0};

void ap() {
	int n, m; cin >> n >> m;
    vector<vector<int>> a(n, vector<int>(m));
    vector<array<int, 2>> b(n * m);
    int k = 0;
    for (int i = 0; i < n; i++) {
        for (int j = 0; j < m; j++) {
            cin >> a[i][j];
            
            b[k][0] = a[i][j], b[k][1] = i * m + j;
            k++;
        }
    }
    
    sort(b.rbegin(), b.rend());
    vector<vector<int>> c(n, vector<int> (m,1));
    for (int i = 0; i < b.size(); i++) {
        int x = b[i][1] / m, y = b[i][1] % m;
        bool vis = false;
        for (int k = 0; k < 4; k++) {
            int nx = x + dx[k], ny = y + dy[k];
            if (nx < 0 || nx >= n || ny < 0 || ny >= m) continue;
            if (a[nx][ny] < a[x][y]) continue;
            
            if(c[nx][ny] % 2 == 0) {
                vis = true;
                break;
            }
        }
        if(!vis) c[x][y] = 0;
    }
    
//  check 
//  for(int i = 0;i < n;i++) {
//     for(int j = 0;j < m;j++) {
//         cout << c[i][j] << " ";
//     }
//     cout << endl;
//  }

    int q; cin >> q;
    while (q--) {
        int x, y; cin >> x >> y;
        x--, y--;
        if(c[x][y] & 1) cout << "First";
        else cout << "Second";
        cout << endl;
    }
	return ;
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
