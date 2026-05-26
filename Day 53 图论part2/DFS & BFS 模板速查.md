# DFS & BFS 模板速查（Java）

> 图论核心搜索算法模板汇总。按场景分类，写题时直接抄。
>
> **核心心法**：
> - **DFS** = 栈（递归调用栈） + 一条路走到黑
> - **BFS** = 队列（FIFO） + 一圈一圈扩散
> - **求最短路用 BFS**，**求所有路径用 DFS**

---

## 一、DFS 模板

### 模板 1：邻接表上的 DFS（找所有路径）

**适用场景**：图以邻接表给出，找从起点到终点的所有路径。  
**代表题**：[LeetCode 797. All Paths From Source to Target](https://leetcode.com/problems/all-paths-from-source-to-target/)

```java
class Solution {
    List<List<Integer>> result = new ArrayList<>();
    List<Integer> path = new ArrayList<>();

    public List<List<Integer>> allPathsSourceTarget(int[][] graph) {
        path.add(0);                              // 起点先放进 path
        dfs(graph, 0, graph.length - 1);
        return result;
    }

    private void dfs(int[][] graph, int x, int end) {
        // 1. 终止条件
        if (x == end) {
            result.add(new ArrayList<>(path));    // ★ 必须拷贝
            return;
        }
        // 2. 遍历当前节点的所有邻居
        for (int next : graph[x]) {
            path.add(next);                       // 做选择
            dfs(graph, next, end);                // 递归
            path.remove(path.size() - 1);         // 回溯：撤销选择
        }
    }
}
```

**关键点**：
- `result.add(new ArrayList<>(path))` **必须拷贝**，否则后续修改 `path` 会污染已收集的结果
- 邻接表场景**不需要方向数组**，邻居直接由 `graph[x]` 给出
- DAG 不需要 `visited`，有环图需要

### 模板 2：网格上的 DFS（标记连通块 / 染色）

**适用场景**：在二维网格上找连通块、计数、染色。  
**代表题**：[LeetCode 200. Number of Islands](https://leetcode.com/problems/number-of-islands/)

```java
class Solution {
    int[][] dirs = {{-1,0},{1,0},{0,-1},{0,1}};   // 4 个方向
    
    public int numIslands(char[][] grid) {
        int rows = grid.length, cols = grid[0].length;
        int count = 0;
        for (int r = 0; r < rows; r++) {
            for (int c = 0; c < cols; c++) {
                if (grid[r][c] == '1') {
                    dfs(grid, r, c);              // 把整片岛染成 '0'
                    count++;
                }
            }
        }
        return count;
    }
    
    private void dfs(char[][] grid, int r, int c) {
        // 1. 越界 / 已访问 / 不是陆地 → 直接返回
        if (r < 0 || r >= grid.length || c < 0 || c >= grid[0].length) return;
        if (grid[r][c] != '1') return;
        
        // 2. 标记当前格子（"染色一次永久标记"，不需要回溯）
        grid[r][c] = '0';
        
        // 3. 向 4 个方向递归
        for (int[] d : dirs) {
            dfs(grid, r + d[0], c + d[1]);
        }
    }
}
```

**与"找路径 DFS"的关键区别**：
- 这里**不需要回溯**（不需要 `grid[r][c] = '1'`），因为标记是永久的
- "找路径"要还原状态（用完归还），"染色 / 连通块"标记完就不需要还原

### 模板 3：图上的 DFS（有环图，需要 visited）

**适用场景**：图可能有环，找路径 / 判连通。

```java
class Solution {
    List<List<Integer>> graph;
    boolean[] visited;
    List<List<Integer>> result = new ArrayList<>();
    List<Integer> path = new ArrayList<>();
    
    public void solve(int n, List<List<Integer>> graph, int start, int end) {
        this.graph = graph;
        this.visited = new boolean[n];
        
        path.add(start);
        visited[start] = true;
        dfs(start, end);
    }
    
    private void dfs(int x, int end) {
        if (x == end) {
            result.add(new ArrayList<>(path));
            return;
        }
        for (int next : graph.get(x)) {
            if (visited[next]) continue;
            
            visited[next] = true;                 // 做选择
            path.add(next);
            dfs(next, end);                       // 递归
            path.remove(path.size() - 1);         // 回溯
            visited[next] = false;                // ★ 回溯：取消标记
        }
    }
}
```

**关键点**：
- **找路径**时，visited 也要回溯（用完归还）
- **染色 / 连通块**时，visited 不需要回溯（标记一次永久）

### DFS 三部曲（通用心法）

```java
void dfs(参数) {
    if (终止条件) {
        收集结果;
        return;
    }
    for (选择 : 当前节点能去的所有邻居) {
        做选择;             // path.add(next) / visited[i] = true
        dfs(新状态);        // 递归
        撤销选择;           // path.remove(last) / visited[i] = false（看场景）
    }
}
```

---

## 二、BFS 模板

### 模板 1：基础 BFS（遍历 / 染色 / 连通块）

**适用场景**：只需访问所有可达节点，不关心步数。  
**代表题**：[LeetCode 200. Number of Islands](https://leetcode.com/problems/number-of-islands/)（BFS 版）

```java
int[][] dirs = {{-1,0},{1,0},{0,-1},{0,1}};

void bfs(int[][] grid, boolean[][] visited, int startR, int startC) {
    int rows = grid.length, cols = grid[0].length;
    Queue<int[]> queue = new ArrayDeque<>();
    
    queue.offer(new int[]{startR, startC});
    visited[startR][startC] = true;              // ★ 入队立即标记
    
    while (!queue.isEmpty()) {
        int[] cur = queue.poll();
        int r = cur[0], c = cur[1];
        
        for (int[] d : dirs) {
            int nr = r + d[0];
            int nc = c + d[1];
            
            if (nr < 0 || nr >= rows || nc < 0 || nc >= cols) continue;
            if (visited[nr][nc]) continue;
            // if (grid[nr][nc] == '0') continue;  // 题目特定的"墙"条件
            
            queue.offer(new int[]{nr, nc});
            visited[nr][nc] = true;
        }
    }
}
```

### 模板 2：按层 BFS（求最短步数）★

**适用场景**：求最短路径、最少步数、最少层数。  
**代表题**：[LeetCode 1091. Shortest Path in Binary Matrix](https://leetcode.com/problems/shortest-path-in-binary-matrix/)

```java
public int shortestPath(int[][] grid) {
    int n = grid.length;
    if (grid[0][0] == 1 || grid[n-1][n-1] == 1) return -1;
    
    int[][] dirs = {
        {-1,-1},{-1, 0},{-1, 1},
        { 0,-1},        { 0, 1},
        { 1,-1},{ 1, 0},{ 1, 1}
    };
    
    Queue<int[]> queue = new ArrayDeque<>();
    boolean[][] visited = new boolean[n][n];
    queue.offer(new int[]{0, 0});
    visited[0][0] = true;
    
    int steps = 1;                                // 起点本身算 1 步
    
    while (!queue.isEmpty()) {
        int size = queue.size();                  // ★ 固定当前圈大小
        
        for (int i = 0; i < size; i++) {          // 只处理这一圈
            int[] cur = queue.poll();
            int r = cur[0], c = cur[1];
            
            if (r == n-1 && c == n-1) return steps;
            
            for (int[] d : dirs) {
                int nr = r + d[0];
                int nc = c + d[1];
                if (nr < 0 || nr >= n || nc < 0 || nc >= n) continue;
                if (visited[nr][nc]) continue;
                if (grid[nr][nc] == 1) continue;
                
                queue.offer(new int[]{nr, nc});
                visited[nr][nc] = true;
            }
        }
        
        steps++;                                  // ★ 一圈处理完，步数 +1
    }
    
    return -1;
}
```

**与基础 BFS 的关键区别**：
- 多了 `int size = queue.size()` + 内层 for 循环
- 每处理完一圈，`steps++`
- 这就是和二叉树**层序遍历**完全一样的模式

### 模板 3：多源 BFS（多个起点同时扩散）

**适用场景**：从多个源点同时出发，找每个点到最近源点的距离。  
**代表题**：[LeetCode 542. 01 Matrix](https://leetcode.com/problems/01-matrix/) / [LeetCode 994. Rotting Oranges](https://leetcode.com/problems/rotting-oranges/)

```java
public int[][] updateMatrix(int[][] mat) {
    int rows = mat.length, cols = mat[0].length;
    int[][] dist = new int[rows][cols];
    Queue<int[]> queue = new ArrayDeque<>();
    
    // ★ 把所有源点（值为 0 的格子）一次性入队
    for (int r = 0; r < rows; r++) {
        for (int c = 0; c < cols; c++) {
            if (mat[r][c] == 0) {
                queue.offer(new int[]{r, c});
            } else {
                dist[r][c] = -1;                  // -1 表示未访问
            }
        }
    }
    
    int[][] dirs = {{-1,0},{1,0},{0,-1},{0,1}};
    while (!queue.isEmpty()) {
        int[] cur = queue.poll();
        int r = cur[0], c = cur[1];
        for (int[] d : dirs) {
            int nr = r + d[0];
            int nc = c + d[1];
            if (nr < 0 || nr >= rows || nc < 0 || nc >= cols) continue;
            if (dist[nr][nc] != -1) continue;     // 已访问
            
            dist[nr][nc] = dist[r][c] + 1;        // 距离 = 来源距离 + 1
            queue.offer(new int[]{nr, nc});
        }
    }
    
    return dist;
}
```

**关键点**：
- **所有源点同时入队**，BFS 自动处理"距离最近的源点"
- 用 `dist` 数组同时充当 visited 和距离记录
- 不用按层处理，因为距离信息直接存在 `dist[][]` 里

### BFS 三要素（通用心法）

```
1. 队列（FIFO）—— 保证按距离分层扩散
2. 方向数组 —— 4 / 8 方向的扩展规则
3. visited —— 入队时立即标记（防止重复入队）
```

---

## 三、Java 实现细节

### Queue 用什么？

```java
// ✓ 推荐
Queue<int[]> queue = new ArrayDeque<>();

// ✗ 也能用但更慢
Queue<int[]> queue = new LinkedList<>();
```

`ArrayDeque` 比 `LinkedList` 更快（没有节点对象开销），除非需要存 `null`，永远选 `ArrayDeque`。

### Queue 主要方法

```java
queue.offer(x);          // 入队（队尾）
queue.poll();            // 出队（队首），空时返回 null
queue.peek();            // 查看队首但不出队
queue.size();
queue.isEmpty();
```

### 坐标的存储

```java
// ✓ 主流：int[] 存 {r, c}
queue.offer(new int[]{r, c});
int[] cur = queue.poll();
int r = cur[0], c = cur[1];

// ✓ 备选：编码成单个 int（坐标范围有限时）
queue.offer(r * cols + c);
int code = queue.poll();
int r = code / cols, c = code % cols;
```

### 方向数组

```java
// 4 方向
int[][] dirs = {{-1,0},{1,0},{0,-1},{0,1}};

// 8 方向
int[][] dirs = {
    {-1,-1},{-1, 0},{-1, 1},
    { 0,-1},        { 0, 1},
    { 1,-1},{ 1, 0},{ 1, 1}
};

// 国际象棋"马"
int[][] dirs = {{-2,-1},{-2,1},{-1,-2},{-1,2},{1,-2},{1,2},{2,-1},{2,1}};
```

**记忆口诀**：`{dr, dc}` —— 行增量在前，列增量在后。**向上是 `r - 1`，不是 `r + 1`**。

---

## 四、选择策略

### DFS 还是 BFS？

| 需求 | 选择 |
|---|---|
| 最短路径 / 最少步数 | **BFS** |
| 找所有路径 | **DFS** |
| 找一条任意路径 | 都可以 |
| 判断连通 / 找连通块 | 都可以（DFS 写法更短） |
| 遍历整棵树 / 图 | 都可以 |
| 拓扑排序 | BFS（Kahn 算法）/ DFS 都可 |
| 检测环 | DFS（三色标记）更直观 |

### 用 visited 吗？

| 场景 | visited 需求 |
|---|---|
| 树（无环） | 不需要 |
| DAG（无环有向图） | 不需要 |
| 普通图（可能有环） | **需要** |
| 网格遍历 | **需要**（除非修改 grid 当 visited） |

### visited 要不要回溯？

| 场景 | 是否回溯 visited |
|---|---|
| 找所有路径（用完归还） | **要**（标记 → 递归 → 取消标记） |
| 染色 / 连通块（永久标记） | **不要** |
| BFS | **不要**（BFS 永远不撤销） |

---

## 五、易错点速查

### DFS 常见坑

1. **忘记拷贝**：`result.add(new ArrayList<>(path))` 而不是 `result.add(path)`
2. **忘记回溯**：找路径场景，递归后必须 `path.remove(last)` 和 `visited[i] = false`
3. **染色场景画蛇添足**：连通块场景**不要**回溯 visited

### BFS 常见坑

1. **visited 标记时机**：必须**入队时**标记，不是出队时
2. **size 时机**：按层 BFS 必须先 `int size = queue.size()`，不能写 `for (i=0; i<queue.size(); i++)`
3. **steps 初值**：路径长度 = 格子数时从 1 开始，= 边数时从 0 开始
4. **终点判断位置**：建议在出队时判断，`steps` 自然就是距离

### 通用坑

1. **方向数组里 `dr` 和 `dc` 弄反**：`{dr, dc}`，第一个是行
2. **越界检查写错**：`nr >= rows` 不是 `nr > rows`
3. **混用 `x, y` 和 `r, c`**：网格题永远用 `r, c`

---

## 六、终极心智模型

**DFS 和 BFS 的唯一本质区别**：用什么容器存"待处理节点"。

```
BFS：用 Queue（FIFO）→ 先入先出 → 按层扩散
DFS：用 Stack（LIFO）→ 后入先出 → 一条路走到黑
     （递归本质上用的是调用栈，也是 LIFO）
```

把 BFS 模板里的 `Queue.poll()` 换成 `Deque.pop()`（栈），不改任何其他逻辑，BFS 就变成 DFS 了。这就是为什么所有教材都把它们放一起讲——**它们是同一个算法的两个变种**。

---

## 七、推荐刷题路径

| 序号 | 题目 | 类型 | 推荐解法 |
|---|---|---|---|
| 1 | [797. All Paths From Source to Target](https://leetcode.com/problems/all-paths-from-source-to-target/) | 邻接表 DFS | DFS |
| 2 | [200. Number of Islands](https://leetcode.com/problems/number-of-islands/) | 网格连通块 | DFS / BFS |
| 3 | [695. Max Area of Island](https://leetcode.com/problems/max-area-of-island/) | 网格连通块 | DFS / BFS |
| 4 | [1091. Shortest Path in Binary Matrix](https://leetcode.com/problems/shortest-path-in-binary-matrix/) | 网格最短路 | BFS（按层） |
| 5 | [994. Rotting Oranges](https://leetcode.com/problems/rotting-oranges/) | 多源 BFS | BFS |
| 6 | [542. 01 Matrix](https://leetcode.com/problems/01-matrix/) | 多源 BFS | BFS |
| 7 | [127. Word Ladder](https://leetcode.com/problems/word-ladder/) | 抽象图最短路 | BFS（按层） |
| 8 | [130. Surrounded Regions](https://leetcode.com/problems/surrounded-regions/) | 边界 DFS | DFS / BFS |
| 9 | [417. Pacific Atlantic Water Flow](https://leetcode.com/problems/pacific-atlantic-water-flow/) | 反向 BFS | DFS / BFS |
| 10 | [207. Course Schedule](https://leetcode.com/problems/course-schedule/) | 拓扑排序 | BFS（Kahn）/ DFS |
