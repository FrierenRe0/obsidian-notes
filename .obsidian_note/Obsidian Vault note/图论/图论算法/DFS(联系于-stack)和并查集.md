DFS非递归模板（栈模拟，二维网格为例）
```c++

void dfsStack(vector<vector<int>>& grid, int startX, int startY) {
    m = grid.size();
    n = grid[0].size();
    vector<vector<bool>> visited(m, vector<bool>(n, false));
    stack<pair<int, int>> st; // 栈存储待探索节点坐标
    // 初始化：起点入栈 + 标记已访问
    st.push({startX, startY});
    visited[startX][startY] = true;
    while (!st.empty()) {
        auto [x, y] = st.top(); // 取栈顶节点（当前探索节点）
        st.pop(); // 弹出（后续处理相邻节点，模拟递归返回）
        // 业务逻辑：处理当前节点
        // ...
        // 遍历相邻节点，未访问则入栈（注意：反向入栈保证遍历顺序与递归一致）
        for (auto& dir : DIRS) {
            int nx = x + dir[0];
            int ny = y + dir[1];
            if (nx >=0 && nx <m && ny >=0 && ny <n && !visited[nx][ny] && grid[nx][ny]==1) {
                visited[nx][ny] = true;
                st.push({nx, ny}); // 入栈 = 递归调用，继续深度探索
            }
		}
    }
}
```

**DFS递归模板（以二维网格/迷宫为例，可适配树、图等结构）<推荐>**
```c++
const int DIRS[4][2] = {{-1,0},{1,0},{0,-1},{0,1}}; // 上下左右四方向
int m, n; // 全局/类内变量，存储网格行、列数
vector<vector<bool>> visited; // 访问标记，避免重复遍历/死循环
void dfs(vector<vector<int>>& grid, int x, int y) {
    // 1. 终止条件：越界 + 已访问 + 非法节点（如迷宫的墙）
    if (x < 0 || x >= m || y < 0 || y >= n || visited[x][y] || grid[x][y] == 0) {
        return;
    }
    // 2. 标记当前节点：已访问（核心，避免重复遍历）
    visited[x][y] = true;
    // 3. 业务逻辑：根据题目需求编写（如统计数量、记录路径、判断目标）
    // ...（例：cnt++ 统计岛屿面积/数量）
    // 4. 遍历所有相邻节点，递归探索（深度优先，一条路走到底）
    for (auto& dir : DIRS) {
        int nx = x + dir[0];
        int ny = y + dir[1];
        dfs(grid, nx, ny); // 递归向下探索，无返回值则遍历所有可达节点
    }
    // 5.回溯恢复（可选）：若需遍历所有路径/组合，需撤销标记
    // visited[x][y] = false; // 例：排列组合、所有路径问题需开启
}
```
