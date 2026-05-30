# Dijkstra 最短路算法学习笔记（LeetCode 743 / 卡码网 47）

> LeetCode 743: https://leetcode.com/problems/network-delay-time/
> 卡码网 47: https://kamacoder.com/problempage.php?pid=1047

---

## 问题理解

给定一张带权有向图（节点 = 城市，边权 = 路上花的时间），从起点出发，求到达终点的最短时间。

**卡码网 47**：固定从节点 1 出发，求到节点 N 的最短距离。
**LeetCode 743**：从节点 k 出发，求所有节点都收到信号的最短时间（即所有节点最短距离的最大值）。

两题核心算法完全一致，只差最后一行返回值。

---

## 为什么不能枚举所有路径

路径数量随节点增多会指数级爆炸，根本算不完。  
需要一个更聪明的策略——**每次只确认"当前最近的一个节点"，利用贪心保证正确性。**

---

## 核心思想：贪心 + 备忘录

**关键洞察：** 所有边权 ≥ 0，所以一旦某个节点是"当前未确认节点里距离源点最近的"，它的距离就永远不可能再变小了。绕任何弯路只会更长。

因此可以安全地把它"锁定"，然后用它去更新邻居。

**minDist 数组**：记录每个节点到源点的当前最短距离（估计值），随着锁定不断更新，最终全部变为确认值。

---

## 三部曲（两个版本的逻辑骨架完全一样）

```
① 选：在还没锁定的节点里，找到距源点最近的那个
② 锁：把它标记为 visited，距离永久确认，不再修改
③ 更新：以这个节点为中转，看能否给邻居找到更短的路
         条件：minDist[cur] + edge.weight < minDist[neighbor]
         如果满足，就更新 minDist[neighbor]
重复，直到所有节点都被锁定。
```

---

## 朴素版 Dijkstra — O(n²)

### 数据结构

- **邻接矩阵** `grid[i][j] = w`：存所有节点对之间的边权，无边则为 ∞
- **minDist 数组**：`minDist[i]` = 源点到节点 i 的当前最短距离
- **visited 数组**：标记节点是否已锁定

### 瓶颈

第①步"找最近节点"需要遍历整个 minDist 数组，每轮 O(n)，共 n 轮，总体 O(n²)。

### Java 实现（LeetCode 743）

```java
import java.util.*;

class Solution {
    public int networkDelayTime(int[][] times, int n, int k) {
        // 建邻接矩阵，初始化为"无穷大"
        int[][] grid = new int[n + 1][n + 1];
        for (int[] row : grid) Arrays.fill(row, Integer.MAX_VALUE);
        for (int[] t : times) grid[t[0]][t[1]] = t[2];

        // minDist[i] = 源点到节点 i 的当前最短距离
        int[] minDist = new int[n + 1];
        Arrays.fill(minDist, Integer.MAX_VALUE);
        boolean[] visited = new boolean[n + 1];
        minDist[k] = 0;

        for (int i = 1; i <= n; i++) {
            // ① 选：找未锁定节点中距源点最近的
            int cur = -1;
            for (int v = 1; v <= n; v++) {
                if (!visited[v] && (cur == -1 || minDist[v] < minDist[cur])) {
                    cur = v;
                }
            }
            if (minDist[cur] == Integer.MAX_VALUE) break; // 剩余节点全不可达

            // ② 锁：标记已确认
            visited[cur] = true;

            // ③ 更新：松弛所有从 cur 出发的边
            for (int v = 1; v <= n; v++) {
                if (!visited[v] && grid[cur][v] != Integer.MAX_VALUE) {
                    minDist[v] = Math.min(minDist[v], minDist[cur] + grid[cur][v]);
                }
            }
        }

        // 取所有节点最短距离的最大值（最慢收到信号的那个）
        int ans = 0;
        for (int v = 1; v <= n; v++) {
            if (minDist[v] == Integer.MAX_VALUE) return -1;
            ans = Math.max(ans, minDist[v]);
        }
        return ans;
    }
}
```

### Python 实现（LeetCode 743）

```python
class Solution:
    def networkDelayTime(self, times: list[list[int]], n: int, k: int) -> int:
        INF = float('inf')

        # 建邻接矩阵
        grid = [[INF] * (n + 1) for _ in range(n + 1)]
        for u, v, w in times:
            grid[u][v] = w

        # minDist[i] = 源点到节点 i 的当前最短距离
        min_dist = [INF] * (n + 1)
        visited = [False] * (n + 1)
        min_dist[k] = 0

        for _ in range(n):
            # ① 选：找未锁定节点中距源点最近的
            cur = -1
            for v in range(1, n + 1):
                if not visited[v] and (cur == -1 or min_dist[v] < min_dist[cur]):
                    cur = v
            if min_dist[cur] == INF:
                break  # 剩余节点全不可达

            # ② 锁：标记已确认
            visited[cur] = True

            # ③ 更新：松弛所有从 cur 出发的边
            for v in range(1, n + 1):
                if not visited[v] and grid[cur][v] != INF:
                    min_dist[v] = min(min_dist[v], min_dist[cur] + grid[cur][v])

        # 取最大值（最慢收到信号的那个）
        ans = max(min_dist[1:])
        return -1 if ans == INF else ans
```

---

## 堆优化版 Dijkstra — O(E log E)

### 优化动机

朴素版的 O(n²) 瓶颈在于第①步每轮都要扫一遍数组。  
当图是**稀疏图**（节点很多但边很少）时，大量时间浪费在"扫没有边的节点"上。

**两个改动，解决两个问题：**

| 问题 | 朴素版 | 堆优化版 |
|---|---|---|
| 找最近节点 | 遍历 minDist，O(n) | 小顶堆堆顶直接取，O(log E) |
| 存储图 | 邻接矩阵，O(n²) 空间 | 邻接表，O(n+E) 空间，只存有边的部分 |

**三部曲逻辑完全不变**，只是数据结构换了。

### 新细节：堆里可能有"过时的旧版本"

每次发现更短路就往堆里推一个新版本，同一节点可能被推入多次。  
弹出时用 `if visited[node]: continue` 跳过已经锁定的旧版本。

### Java 实现（LeetCode 743）

```java
import java.util.*;

class Solution {
    public int networkDelayTime(int[][] times, int n, int k) {
        // 邻接表：grid[i] 存 {邻居, 权值}
        List<int[]>[] grid = new List[n + 1];
        for (int i = 0; i <= n; i++) grid[i] = new ArrayList<>();
        for (int[] t : times) grid[t[0]].add(new int[]{t[1], t[2]});

        int[] minDist = new int[n + 1];
        Arrays.fill(minDist, Integer.MAX_VALUE);
        boolean[] visited = new boolean[n + 1];
        minDist[k] = 0;

        // 小顶堆，按距离排序：{距离, 节点}
        PriorityQueue<int[]> pq = new PriorityQueue<>((a, b) -> a[0] - b[0]);
        pq.offer(new int[]{0, k});

        while (!pq.isEmpty()) {
            int[] cur = pq.poll();
            int dist = cur[0], node = cur[1];

            if (visited[node]) continue;   // ← 跳过堆里的旧版本
            visited[node] = true;          // ② 锁定

            for (int[] edge : grid[node]) { // ③ 只遍历真实邻居
                int next = edge[0], w = edge[1];
                if (!visited[next] && dist + w < minDist[next]) {
                    minDist[next] = dist + w;
                    pq.offer(new int[]{minDist[next], next}); // 推入堆
                }
            }
        }

        int ans = 0;
        for (int v = 1; v <= n; v++) {
            if (minDist[v] == Integer.MAX_VALUE) return -1;
            ans = Math.max(ans, minDist[v]);
        }
        return ans;
    }
}
```

### Python 实现（LeetCode 743）

```python
import heapq

class Solution:
    def networkDelayTime(self, times, n, k):
        grid = [[] for _ in range(n + 1)]
        for u, v, w in times:
            grid[u].append((v, w))

        min_dist = [float('inf')] * (n + 1)
        visited = [False] * (n + 1)
        min_dist[k] = 0

        pq = [(0, k)]  # (距离, 节点)，heapq 默认小顶堆

        while pq:
            dist, node = heapq.heappop(pq)

            if visited[node]: continue   # 跳过旧版本
            visited[node] = True

            for nxt, w in grid[node]:
                if not visited[nxt] and dist + w < min_dist[nxt]:
                    min_dist[nxt] = dist + w
                    heapq.heappush(pq, (min_dist[nxt], nxt))

        ans = max(min_dist[1:])
        return -1 if ans == float('inf') else ans
```

---

## 两题的唯一区别：最后一行

```java
// 卡码网 47：到固定终点
return minDist[n] == Integer.MAX_VALUE ? -1 : minDist[n];

// LeetCode 743：所有节点都收到 = 等最慢的那个
int ans = 0;
for (int v = 1; v <= n; v++) {
    if (minDist[v] == Integer.MAX_VALUE) return -1;
    ans = Math.max(ans, minDist[v]);
}
return ans;
```

---

## 常见错误

1. **数组越界**：节点编号从 1 开始，数组要开 `n + 1` 大小
2. **忘记处理不连通**：某节点 minDist 仍为 ∞ 时需返回 -1
3. **堆优化忘写 `if visited: continue`**：同一节点被重复处理，结果出错
4. **松弛条件写错**：是 `minDist[cur] + w < minDist[v]`，不是 `<=`

---

## 复杂度总结

| 版本 | 时间 | 空间 | 适合场景 |
|---|---|---|---|
| 朴素版 | O(n²) | O(n²) | 稠密图，节点少 |
| 堆优化版 | O(E log E) | O(n + E) | 稀疏图，边少节点多 |

---

## 算法局限性

Dijkstra **不能处理负权边**。  
原因：贪心假设"绕路只会更远"在负权边下不成立。已经锁定的节点可能通过负权边被更新，但锁定后不再处理，导致结果出错。  
有负权边的场景需改用 Bellman-Ford 算法。
