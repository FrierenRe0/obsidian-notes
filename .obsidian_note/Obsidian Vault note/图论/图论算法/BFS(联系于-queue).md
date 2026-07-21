层序模板)（二维网格/迷宫为例，直接适配最短路径问题）
```c++
通用BFS层序模板（二维网格/迷宫为例，直接适配最短路径问题）

const int DIRS[4][2] = {{-1,0},{1,0},{0,-1},{0,1}}; // 上下左右四方向

int bfs(vector<vector<char>>& maze, vector<int>& entrance) {
    int m = maze.size();
    int n = maze[0].size();
    int sx = entrance[0], sy = entrance[1]; // 起点坐标
    vector<vector<bool>> visited(m, vector<bool>(n, false));
    queue<pair<int, int>> q; // 队列存储当前层所有节点
    // 初始化：起点入队 + 标记已访问
    q.push({sx, sy});
    visited[sx][sy] = true;
    int step = 0; // 记录步数/层数，即最短路径长度
    while (!q.empty()) {
        int len = q.size(); // 关键：固定当前层节点数量（层序核心）
        // 遍历当前层所有节点（逐层扩散）
        for (int i = 0; i < len; i++) {
            auto [x, y] = q.front();
            q.pop(); // 出队 = 处理当前节点	
            // 业务逻辑：判断是否到达目标（如迷宫出口、终点）
            if ((x == 0 || x == m-1 || y == 0 || y == n-1) && (x != sx || y != sy)) {
                return step; // 找到最短路径，直接返回步数
            }
            // 遍历相邻节点，加入下一层
            for (auto& dir : DIRS) {
                int nx = x + dir[0];
                int ny = y + dir[1];
                // 未越界 + 未访问 + 合法节点（如迷宫的通路）
                if (nx >=0 && nx <m && ny >=0 && ny <n && !visited[nx][ny] && maze[nx][ny] == '.') {
                    visited[nx][ny] = true;
                    q.push({nx, ny}); // 入队 = 加入下一层，待后续遍历
                }
            }
        }
        step++; // 遍历完当前层，步数+1（进入下一层）
    }
    return -1; // 队列空仍未找到目标，说明无可行路径
}
```
