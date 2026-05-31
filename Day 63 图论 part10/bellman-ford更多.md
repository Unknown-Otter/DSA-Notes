# Bellman-Ford 三大扩展

> 三个扩展共享同一个核心：控制"跑多少轮"以及"哪些边值得松弛"。
> 理解基础版之后，每个扩展只改一两处。

---

## 扩展一：队列优化（SPFA）

### 问题

基础版每轮对所有边松弛，但大量边的起点 `minDist[u]` 根本没有变化，松弛它们毫无意义，纯属浪费。

### 思路

只有上一轮 `minDist` 发生了变化的节点，才值得在这轮继续松弛它的出边。  
用一个队列维护"有变化的节点"，每次只处理队列里的节点，跳过所有无效松弛。

这个优化叫 **SPFA（Shortest Path Faster Algorithm）**，是 Bellman-Ford 的队列版本。

### 与基础版的区别

| | 基础版 | SPFA |
|---|---|---|
| 每轮松弛对象 | 所有边 | 只有距离发生变化的节点的出边 |
| 控制结构 | 固定跑 n-1 轮 | 队列空了就停 |
| 时间复杂度 | O(V·E) | 最优 O(E)，最差 O(V·E) |

### 模板题：LeetCode 743 Network Delay Time

SPFA 与基础版解的是同一道题，此处用 743 做对比。

### Java 实现

```java
import java.util.*;

class Solution {
    public int networkDelayTime(int[][] times, int n, int k) {
        final int INF = Integer.MAX_VALUE / 2;

        // 建邻接表
        List<int[]>[] graph = new List[n + 1];
        for (int i = 0; i <= n; i++) graph[i] = new ArrayList<>();
        for (int[] t : times) graph[t[0]].add(new int[]{t[1], t[2]});

        int[] minDist = new int[n + 1];
        Arrays.fill(minDist, INF);
        minDist[k] = 0;

        boolean[] inQueue = new boolean[n + 1]; // 记录节点是否已在队列中，避免重复入队
        Queue<Integer> queue = new LinkedList<>();
        queue.offer(k);
        inQueue[k] = true;

        while (!queue.isEmpty()) {
            int u = queue.poll();
            inQueue[u] = false;

            // 只松弛 u 的出边（而不是所有边）
            for (int[] edge : graph[u]) {
                int v = edge[0], w = edge[1];
                if (minDist[u] + w < minDist[v]) {
                    minDist[v] = minDist[u] + w;
                    if (!inQueue[v]) {       // 有更新且不在队列中，才入队
                        queue.offer(v);
                        inQueue[v] = true;
                    }
                }
            }
        }

        int ans = 0;
        for (int v = 1; v <= n; v++) {
            if (minDist[v] == INF) return -1;
            ans = Math.max(ans, minDist[v]);
        }
        return ans;
    }
}
```

### Python 实现

```python
from collections import deque

class Solution:
    def networkDelayTime(self, times, n, k):
        INF = float('inf')

        # 建邻接表
        graph = [[] for _ in range(n + 1)]
        for u, v, w in times:
            graph[u].append((v, w))

        min_dist = [INF] * (n + 1)
        min_dist[k] = 0

        in_queue = [False] * (n + 1)
        queue = deque([k])
        in_queue[k] = True

        while queue:
            u = queue.popleft()
            in_queue[u] = False

            for v, w in graph[u]:
                if min_dist[u] + w < min_dist[v]:
                    min_dist[v] = min_dist[u] + w
                    if not in_queue[v]:
                        queue.append(v)
                        in_queue[v] = True

        ans = max(min_dist[1:])
        return -1 if ans == INF else ans
```

### 注意

SPFA 在随机图上很快，但在特意构造的图（如网格图）上会退化到 O(V·E)，面试中不如直接用 Bellman-Ford 基础版或 Dijkstra 堆优化版更稳。

---

## 扩展二：负权回路检测

### 问题

如果图中存在负权环（绕一圈总权值 < 0），每多绕一圈距离就缩小，"最短路"趋向负无穷，没有意义。需要能检测并报告这种情况。

### 思路

基础版跑 n-1 轮的前提是"没有负权环"。如果多跑第 n 轮，正常图的 `minDist` 不会再变化；但如果存在负权环，信息仍在沿环传播，第 n 轮还会有节点被更新。

**检测方法**：跑完 n-1 轮后，再跑第 n 轮，如果任何节点的 `minDist` 仍能被更新，则存在负权环。

与基础版的唯一区别：多跑一轮，检查有没有更新发生。

### 模板题：LeetCode 207 课程表（判断是否有环）

207 是 BFS/DFS 拓扑排序的经典题，这里用它演示负权环检测思路。  
更直接的负权环检测模板题是 **LeetCode 1462 / 卡码网 94**，但 207 更常见。

以下用一个通用的负权环检测函数展示：

### Java 实现（通用负权环检测）

```java
import java.util.*;

public class NegativeCycleDetector {

    // 返回 true 表示存在负权环
    public boolean hasNegativeCycle(int n, int[][] edges) {
        final int INF = Integer.MAX_VALUE / 2;
        int[] minDist = new int[n + 1];
        // 初始化为 0（检测全图是否有负权环，不关心源点）
        // 若从特定源点出发，改为：minDist[src] = 0，其余 INF

        // 跑 n 轮（比基础版多一轮）
        for (int i = 1; i <= n; i++) {
            boolean updated = false;
            for (int[] edge : edges) {
                int u = edge[0], v = edge[1], w = edge[2];
                if (minDist[u] + w < minDist[v]) {
                    minDist[v] = minDist[u] + w;
                    updated = true;
                    // 第 n 轮（i == n）仍有更新 → 负权环
                    if (i == n) return true;
                }
            }
            if (!updated) break; // 提前收敛，可以停止
        }
        return false;
    }
}
```

### Python 实现（通用负权环检测）

```python
def has_negative_cycle(n, edges):
    """
    n: 节点数
    edges: [[u, v, w], ...] 有向边列表
    返回 True 表示存在负权环
    """
    min_dist = [0] * (n + 1)  # 全0初始化，检测全图负权环

    for i in range(1, n + 1):      # 跑 n 轮
        updated = False
        for u, v, w in edges:
            if min_dist[u] + w < min_dist[v]:
                min_dist[v] = min_dist[u] + w
                updated = True
                if i == n:         # 第 n 轮仍有更新 → 负权环
                    return True
        if not updated:
            break                  # 提前收敛

    return False
```

### 结合 SPFA 检测负权环

SPFA 也能检测负权环：若某个节点入队次数超过 n 次，说明它在环中被反复更新，即存在负权环。

```java
// 在 SPFA 基础上加入队次数统计
int[] count = new int[n + 1];  // 记录每个节点入队次数

while (!queue.isEmpty()) {
    int u = queue.poll();
    inQueue[u] = false;

    for (int[] edge : graph[u]) {
        int v = edge[0], w = edge[1];
        if (minDist[u] + w < minDist[v]) {
            minDist[v] = minDist[u] + w;
            if (!inQueue[v]) {
                queue.offer(v);
                inQueue[v] = true;
                count[v]++;
                if (count[v] >= n) return true; // 存在负权环
            }
        }
    }
}
```

---

## 扩展三：单源有限最短路

### 问题

不是求"任意步数内的最短路"，而是求"最多经过 k 条边（或 k 个中转站）的最短路"。Dijkstra 无法自然表达这个限制，Bellman-Ford 只需控制轮数。

### 思路

基础版跑 n-1 轮 = 允许走 n-1 条边 = 不限步数。  
限制最多走 k 条边 = 只跑 k 轮。

**一个关键细节**：每轮松弛必须用上一轮的 `minDist` 快照，而不是实时更新的值。  
原因：如果用实时值，同一轮内的更新会"串"到下一条边，等于多走了一步，破坏了"最多 k 步"的约束。

### 模板题：LeetCode 787 Cheapest Flights Within K Stops

**题意**：n 个城市，m 条有向航线，每条有费用。从 src 出发，最多经过 k 个中转站（即最多走 k+1 条边），求到 dst 的最小费用。

### Java 实现

```java
import java.util.*;

class Solution {
    public int findCheapestPrice(int n, int[][] flights, int src, int dst, int k) {
        final int INF = Integer.MAX_VALUE / 2;
        int[] minDist = new int[n];
        Arrays.fill(minDist, INF);
        minDist[src] = 0;

        // 最多 k 个中转站 = 最多 k+1 条边 = 松弛 k+1 轮
        for (int i = 0; i <= k; i++) {
            // 关键：用上一轮的快照，防止同轮内"串改"
            int[] temp = Arrays.copyOf(minDist, n);

            for (int[] flight : flights) {
                int u = flight[0], v = flight[1], w = flight[2];
                // 用 minDist（上一轮值）更新 temp（本轮值）
                if (minDist[u] != INF && minDist[u] + w < temp[v]) {
                    temp[v] = minDist[u] + w;
                }
            }
            minDist = temp; // 本轮结束，更新为快照结果
        }

        return minDist[dst] >= INF ? -1 : minDist[dst];
    }
}
```

### Python 实现

```python
class Solution:
    def findCheapestPrice(self, n: int, flights: list[list[int]],
                          src: int, dst: int, k: int) -> int:
        INF = float('inf')
        min_dist = [INF] * n
        min_dist[src] = 0

        for _ in range(k + 1):          # 松弛 k+1 轮
            temp = min_dist[:]           # 快照：用上一轮的值做判断
            for u, v, w in flights:
                if min_dist[u] != INF and min_dist[u] + w < temp[v]:
                    temp[v] = min_dist[u] + w
            min_dist = temp

        return -1 if min_dist[dst] == INF else min_dist[dst]
```

### 为什么必须用快照

```
假设边：A→B 权1，B→C 权1，src=A，k=1（最多1个中转，即最多2条边）

错误（不用快照）：
  第1轮：A→B，minDist[B]=1（正确）
         B→C，minDist[C]=2（错误！B刚在本轮更新，相当于走了2条边，违反k=1的限制）

正确（用快照）：
  第1轮开始时快照：A=0, B=INF, C=INF
  处理A→B：temp[B]=1
  处理B→C：minDist[B]还是INF（快照值），不更新
  第1轮结束：minDist = [0, 1, INF]
  → C 无法在1轮内到达，结果正确
```

---

## 三个扩展对比总结

| | SPFA | 负权环检测 | 单源有限最短路 |
|---|---|---|---|
| 改动点 | 用队列替代固定轮数 | 多跑第 n 轮，检查是否还有更新 | 限制轮数为 k+1，加快照 |
| 核心代码改动 | 把两层 for 改成 while + queue | 把 `n-1` 改成 `n`，加判断 | 把 `n-1` 改成 `k+1`，加 `temp = copy` |
| 模板题 | LC 743 | 卡码网 94 / 通用检测函数 | LC 787 |
| 时间复杂度 | O(E) ~ O(V·E) | O(V·E) | O(k·E) |
| 能处理负权边 | 能 | 能（这正是检测的目的） | 能 |
