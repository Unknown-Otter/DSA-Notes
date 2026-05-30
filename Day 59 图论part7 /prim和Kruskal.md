# 最小生成树（MST）：Prim 与 Kruskal

> 卡码网 53. 寻宝：https://kamacoder.com/problempage.php?pid=1053
> LeetCode 1584. Min Cost to Connect All Points：https://leetcode.com/problems/min-cost-to-connect-all-points/

## 一、基础知识

**最小生成树（Minimum Spanning Tree）**：在一个带权无向图中，用 `n - 1` 条边把全部 `n` 个点连通，且总权重最小的那棵树。

两种主流构造算法，都基于 **Cut Property（切割性质）**——任何切割上的最小权重边一定属于某棵 MST，所以两者都是**贪心**：

| | Kruskal | Prim |
|---|---|---|
| 视角 | **边**（全局排序） | **点**（局部扩张） |
| 防止成环 | 并查集判断连通性 | 只从「树→树外」选边，天然不成环 |
| 核心结构 | Union-Find | `minDist` 数组 / 最小堆 |
| 适合 | **稀疏图**（边少，`O(E log E)`） | **稠密图**（边多，朴素 `O(V²)`） |

> **53.寻宝 vs LeetCode 1584 是同一道题。** 53题直接给边；1584 是**稠密图**：任意两点间都有一条边，边权 = 曼哈顿距离 `|xi-xj| + |yi-yj|`。点数 `n ≤ 1000` → 边数约 `n²/2 ≈ 50万`。**稠密图首选 Prim（朴素 O(V²)）**，无需显式建出所有边。

---

## 二、Prim 算法模板（朴素 O(V²)，稠密图首选）

### 1. 核心思想

维护一个 `minDist[v]` 数组，含义是 **「当前这棵树」到未加入点 v 的最短单条边**（注意：不是从起点累加的路径，这是和 Dijkstra 的关键区别）。每轮做两件事：

1. 从所有未加入点中，选 `minDist` 最小的点加入树
2. 用这个新点去**更新**它邻居的 `minDist`

### 2. Java 模板

```java
import java.util.*;

public class Main {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        int v = scanner.nextInt();
        int e = scanner.nextInt();

        // 邻接矩阵，不可达初始化为大值
        int[][] grid = new int[v + 1][v + 1];
        for (int i = 0; i <= v; i++) Arrays.fill(grid[i], 10001);
        for (int i = 0; i < e; i++) {
            int x = scanner.nextInt(), y = scanner.nextInt(), k = scanner.nextInt();
            grid[x][y] = k;
            grid[y][x] = k;
        }

        int[] minDist = new int[v + 1];   // 树到各点的最短边
        Arrays.fill(minDist, 10001);
        boolean[] isInTree = new boolean[v + 1];

        // 循环 v - 1 次，每次纳入一个点（也即确定一条边）
        for (int i = 1; i < v; i++) {
            int cur = -1, minVal = Integer.MAX_VALUE;
            // (1) 选离树最近的未访问点
            for (int j = 1; j <= v; j++) {
                if (!isInTree[j] && minDist[j] < minVal) {
                    minVal = minDist[j];
                    cur = j;
                }
            }
            isInTree[cur] = true;
            // (2) 用新点更新邻居的 minDist
            for (int j = 1; j <= v; j++) {
                if (!isInTree[j] && grid[cur][j] < minDist[j]) {
                    minDist[j] = grid[cur][j];
                }
            }
        }

        int result = 0;
        for (int i = 2; i <= v; i++) result += minDist[i];  // 从 2 开始：起点 1 无入边
        System.out.println(result);
        scanner.close();
    }
}
```

### 3. Python 模板

```python
import sys

def main():
    data = sys.stdin.read().split()
    idx = 0
    v = int(data[idx]); idx += 1
    e = int(data[idx]); idx += 1

    # 邻接矩阵，不可达初始化为大值
    grid = [[10001] * (v + 1) for _ in range(v + 1)]
    for _ in range(e):
        x = int(data[idx]); y = int(data[idx + 1]); k = int(data[idx + 2])
        idx += 3
        grid[x][y] = k
        grid[y][x] = k

    min_dist = [10001] * (v + 1)   # 树到各点的最短边
    is_in_tree = [False] * (v + 1)

    # 循环 v - 1 次，每次纳入一个点（也即确定一条边）
    for _ in range(1, v):
        cur, min_val = -1, sys.maxsize
        # (1) 选离树最近的未访问点
        for j in range(1, v + 1):
            if not is_in_tree[j] and min_dist[j] < min_val:
                min_val = min_dist[j]
                cur = j
        is_in_tree[cur] = True
        # (2) 用新点更新邻居的 min_dist
        for j in range(1, v + 1):
            if not is_in_tree[j] and grid[cur][j] < min_dist[j]:
                min_dist[j] = grid[cur][j]

    result = sum(min_dist[2:v + 1])   # 从下标 2 开始：起点 1 无入边
    print(result)

if __name__ == "__main__":
    main()
```

### 4. 三个易错点

1. **`minDist[i]` 在「点被纳入树时」就已经锁定为它进树用的那条边**，所以最后直接累加 `minDist[2..v]` 就是总权重，无需额外记录。
2. 主循环跑 **`v - 1` 次**（建 `n-1` 条边）；求和**从下标 2 开始**，因为第一个点（起点 1）是白嫖进树的、没有入边。
3. 起点不需要显式设置——所有 `minDist` 初始都是大值，第一轮会随便选一个点当起点，结果不受影响。

---

## 三、Kruskal 算法模板（O(E log E)，稀疏图首选）

### 1. 核心思想

**所有边按权重升序排序**，依次尝试加入；用**并查集**判断这条边的两端是否已连通——若已连通则加它会成环，**跳过**；否则加入并合并。

### 2. Java 模板

```java
import java.util.*;

class Edge {
    int l, r, val;
    Edge(int l, int r, int val) { this.l = l; this.r = r; this.val = val; }
}

public class Main {
    private static int n = 10001;
    private static int[] father = new int[n];

    public static void init() {
        for (int i = 0; i < n; i++) father[i] = i;
    }

    // 路径压缩
    public static int find(int u) {
        if (u == father[u]) return u;
        return father[u] = find(father[u]);
    }

    public static void join(int u, int v) {
        u = find(u);
        v = find(v);
        if (u == v) return;
        father[v] = u;
    }

    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        int v = scanner.nextInt();
        int e = scanner.nextInt();
        List<Edge> edges = new ArrayList<>();
        for (int i = 0; i < e; i++) {
            int v1 = scanner.nextInt(), v2 = scanner.nextInt(), val = scanner.nextInt();
            edges.add(new Edge(v1, v2, val));
        }

        // (1) 按边权升序排序
        edges.sort(Comparator.comparingInt(edge -> edge.val));
        init();

        int result_val = 0;
        // (2) 贪心选边，并查集去环
        for (Edge edge : edges) {
            int x = find(edge.l);
            int y = find(edge.r);
            if (x != y) {            // 两端未连通 → 加入
                result_val += edge.val;
                join(x, y);
            }                        // 已连通 → 自动跳过（会成环）
        }
        System.out.println(result_val);
        scanner.close();
    }
}
```

### 3. Python 模板

```python
import sys

n = 10001
father = list(range(n))

# 路径压缩
def find(u):
    if u != father[u]:
        father[u] = find(father[u])
    return father[u]

def join(u, v):
    u, v = find(u), find(v)
    if u == v:
        return
    father[v] = u

def main():
    data = sys.stdin.read().split()
    idx = 0
    v = int(data[idx]); idx += 1
    e = int(data[idx]); idx += 1

    edges = []   # (边权, 左端, 右端)
    for _ in range(e):
        v1 = int(data[idx]); v2 = int(data[idx + 1]); val = int(data[idx + 2])
        idx += 3
        edges.append((val, v1, v2))

    # (1) 按边权升序排序
    edges.sort()

    result_val = 0
    # (2) 贪心选边，并查集去环
    for val, l, r in edges:
        if find(l) != find(r):   # 两端未连通 → 加入
            result_val += val
            join(l, r)
        # 已连通 → 自动跳过（会成环）

    print(result_val)

if __name__ == "__main__":
    main()
```

### 4. 三个易错点

1. **并查集的 `find` 一定要写路径压缩**（`father[u] = find(father[u])`），否则在大数据量下会退化超时。
2. 排序是性能瓶颈，整体复杂度由 `O(E log E)` 主导。
3. `father` 数组大小要 ≥ 最大点编号 + 1，本题点从 1 编号，开 `10001` 足够。
4. Python 把边存成 `(边权, 左, 右)` 元组直接 `sort()`，按首位边权排序，比自定义 `Edge` 类更简洁。

---

## 四、迁移到 LeetCode 1584（稠密图 → Prim 朴素版）

1584 给的是点坐标，需要**自己把图建出来**：任意两点 `i, j` 之间有一条边，权重 `= |xi-xj| + |yi-yj|`。点数 `n ≤ 1000`，是典型稠密图，**Prim 朴素 O(n²) 最合适**，且不必显式存所有边——`minDist` 更新时现算距离即可。

```java
class Solution {
    public int minCostConnectPoints(int[][] points) {
        int n = points.length;
        int[] minDist = new int[n];
        boolean[] inTree = new boolean[n];
        Arrays.fill(minDist, Integer.MAX_VALUE);
        minDist[0] = 0;          // 从 0 号点起步
        int result = 0;

        for (int i = 0; i < n; i++) {
            // (1) 选离树最近的未访问点
            int cur = -1;
            for (int j = 0; j < n; j++) {
                if (!inTree[j] && (cur == -1 || minDist[j] < minDist[cur])) cur = j;
            }
            inTree[cur] = true;
            result += minDist[cur];   // 这条边正式入树

            // (2) 用新点现算距离，更新邻居 minDist
            for (int j = 0; j < n; j++) {
                if (!inTree[j]) {
                    int d = Math.abs(points[cur][0] - points[j][0])
                          + Math.abs(points[cur][1] - points[j][1]);
                    if (d < minDist[j]) minDist[j] = d;
                }
            }
        }
        return result;
    }
}
```

```python
class Solution:
    def minCostConnectPoints(self, points: List[List[int]]) -> int:
        n = len(points)
        min_dist = [float('inf')] * n
        in_tree = [False] * n
        min_dist[0] = 0          # 从 0 号点起步
        result = 0

        for _ in range(n):
            # (1) 选离树最近的未访问点
            cur = -1
            for j in range(n):
                if not in_tree[j] and (cur == -1 or min_dist[j] < min_dist[cur]):
                    cur = j
            in_tree[cur] = True
            result += min_dist[cur]   # 这条边正式入树

            # (2) 用新点现算距离，更新邻居 min_dist
            for j in range(n):
                if not in_tree[j]:
                    d = abs(points[cur][0] - points[j][0]) + abs(points[cur][1] - points[j][1])
                    if d < min_dist[j]:
                        min_dist[j] = d

        return result
```

> **与卡码网模板的两处差别**：
> 1. 这里循环 `n` 次并在每轮 `result += minDist[cur]` 累加（0 号点 `minDist[0]=0` 不影响），写法更直接，不必最后再单独求和。
> 2. 边权不预存，在第 (2) 步**用坐标现算**曼哈顿距离，省掉 `O(n²)` 的邻接矩阵空间。

---

## 五、Prim 堆优化版（O(E log E)，稀疏图也适用）

### 1. 与朴素版的区别

朴素 Prim 每轮用 `O(V)` 遍历找「离树最近的点」，总共 `O(V²)`。堆优化的思路是：**把所有「树→树外」的候选边丢进最小堆**，要找最近点时直接 `O(log E)` 弹堆顶，省掉那层线性扫描。

- 朴素版按 **点** 维护 `minDist` 数组；堆优化版按 **边** 往堆里塞。
- 堆里可能有**过期边**（指向已入树的点），弹出时判一下 `if (visited[to]) continue;` 跳过即可，无需主动删除。
- 稀疏图 `E ≈ V` 时优势明显；**稠密图 `E ≈ V²` 反而 `O(V² log V)` 更慢**，那时用朴素版。

### 2. Java 模板

```java
import java.util.*;

public class Main {
    // 边：to = 目标点, val = 边权
    static class Edge {
        int to, val;
        Edge(int to, int val) { this.to = to; this.val = val; }
    }

    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        int v = scanner.nextInt();
        int e = scanner.nextInt();

        // 邻接表
        List<List<Edge>> graph = new ArrayList<>();
        for (int i = 0; i <= v; i++) graph.add(new ArrayList<>());
        for (int i = 0; i < e; i++) {
            int x = scanner.nextInt(), y = scanner.nextInt(), k = scanner.nextInt();
            graph.get(x).add(new Edge(y, k));
            graph.get(y).add(new Edge(x, k));   // 无向图，双向加
        }

        boolean[] inTree = new boolean[v + 1];
        // 小根堆，按边权排序
        PriorityQueue<Edge> pq = new PriorityQueue<>((a, b) -> a.val - b.val);

        int result = 0, count = 0;
        pq.offer(new Edge(1, 0));   // 从 1 号点起步，起始边权 0

        while (!pq.isEmpty() && count < v) {
            Edge cur = pq.poll();
            if (inTree[cur.to]) continue;   // 过期边：目标点已入树，跳过
            inTree[cur.to] = true;
            result += cur.val;
            count++;
            // 把新点的所有出边丢进堆
            for (Edge next : graph.get(cur.to)) {
                if (!inTree[next.to]) pq.offer(next);
            }
        }
        System.out.println(result);
        scanner.close();
    }
}
```

### 3. Python 模板

```python
import sys
import heapq

def main():
    data = sys.stdin.read().split()
    idx = 0
    v = int(data[idx]); idx += 1
    e = int(data[idx]); idx += 1

    # 邻接表
    graph = [[] for _ in range(v + 1)]
    for _ in range(e):
        x = int(data[idx]); y = int(data[idx + 1]); k = int(data[idx + 2])
        idx += 3
        graph[x].append((k, y))   # (边权, 目标点)
        graph[y].append((k, x))   # 无向图，双向加

    in_tree = [False] * (v + 1)
    pq = [(0, 1)]   # (边权, 点)，从 1 号点起步
    result = 0
    count = 0

    while pq and count < v:
        val, cur = heapq.heappop(pq)
        if in_tree[cur]:           # 过期边，跳过
            continue
        in_tree[cur] = True
        result += val
        count += 1
        for w, nxt in graph[cur]:  # 新点的出边入堆
            if not in_tree[nxt]:
                heapq.heappush(pq, (w, nxt))

    print(result)

if __name__ == "__main__":
    main()
```

### 4. 三个易错点

1. **必须判 `if (inTree[to]) continue;`**——堆里会有大量指向「已入树点」的过期边，不跳过会重复计算。这是堆优化版最容易写错的地方。
2. Python 的 `heapq` 是小根堆，元组 `(边权, 点)` 会**按第一个元素排序**，所以边权要放元组首位。
3. 起步时往堆里塞一条 `(0, 起点)` 的虚拟边，`count` 计到 `v` 时说明 `n` 个点全部入树（含起点那一轮）。

---

## 六、一句话总结

1. **MST = 用 `n-1` 条边连通所有点、总权重最小**，两个算法都靠贪心 + 切割性质成立。
2. **Kruskal 按边贪**：排序 + 并查集去环，**稀疏图**（`O(E log E)`）。
3. **Prim 按点贪**：`minDist` 数组从一点滚雪球扩张，**稠密图**（朴素 `O(V²)`）。
4. **Prim 堆优化**：候选边入最小堆，弹堆顶选最近点，**稀疏图**（`O(E log E)`），但需判过期边 `if inTree[to] continue`。
5. **卡码网 53 与 LeetCode 1584 是同一道 MST 题**，1584 是稠密图、需自建曼哈顿距离边，**首选 Prim 朴素版**。
