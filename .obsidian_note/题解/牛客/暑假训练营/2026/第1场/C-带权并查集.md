[C-Fish Eating_2026牛客暑期多校训练营1](https://ac.nowcoder.com/acm/contest/133876/C)
![[Pasted image 20260717194126.png]]

```c++
#include <bits/stdc++.h>
using namespace std;
#define int long long
const int mod = 1e9 + 7;
const int N = 3e5 + 7;
const int mxn = 5e5 + 5;
int d[4][2] = {{0, 1}, {0, -1}, {1, 0}, {-1, 0}};
int n, m, q;

int fa[mxn],sz[mxn], req[mxn], a[mxn];
//int fa[mxn]; // 并查集父节点 
//int req[mxn]; // 点权：从该点出发吃完整个连通块需要的最小初始体型 
//int sz[mxn]; // 集合总鱼数，仅根节点有效 
//int a[mxn]; // 格子鱼体型，只有集合根的r代表连通块最大鱼体型

int find(int x)
{
    if (fa[x] == x)
        return x;
    int t = find(fa[x]);
    req[x] = max(req[x], req[fa[x]]);//更新数据在该点上吃完整个连通块需要的最小初始体型
    //因为这里我们只要看比他大的节点所需的初始最小体格
    fa[x] = t;
    return t;
}
void merge(int x, int y)
{
    x = find(x);
    y = find(y);
    if (x == y) return;
    // y --》 x    因为这里x >= y;
    fa[y] = x;
    sz[x] += sz[y];
    req[y] = max(0LL, a[x] - sz[y] + 1);
    return;
}

int getid(int x, int y)
{
    return x * m + y;
}
void solve()
{
    cin >> n >> m >> q;
    int N = n * m;
    for (int i = 0; i < N; i++)
    {
        fa[i] = i;
        sz[i] = 1;
        req[i] = 0;
        a[i] = 0;
    }
    int before = 0;
    while (q--)
    {
        int tp, x, y;
        cin >> tp >> x >> y;
        x = x ^ before;
        y = y ^ before;
        x--;
        y--;
        if (tp == 1)
        {
            int v;
            cin >> v;
            
            a[getid(x, y)] = v;
            for (auto &add : d)
            {
                int dx = x + add[0];
                int dy = y + add[1];
                if (dx >= 0 && dy >= 0 && dx < n && dy < m)
                {
                    if(a[getid(dx,dy)]) merge(getid(x, y), getid(dx, dy));
                }
            }
            before = sz[getid(x, y)] - 1;
        }
        else
        {
            find(getid(x, y));//find->更新数据+路径压缩
            before = max(0LL, req[getid(x, y)] - a[getid(x, y)]);
        }
        cout << before << endl;
    }
    return;
}
signed main()
{
    ios::sync_with_stdio(false);
    cin.tie(nullptr);
    cout.tie(0);
    int T = 1;
    // cin >> T;
    while (T--)
    {
        solve();
    }
    return 0;
}
```