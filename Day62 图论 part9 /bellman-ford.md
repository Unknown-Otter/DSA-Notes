# Bellman-Ford 基础版

> 模板题：LeetCode 743 Network Delay Time
> https://leetcode.com/problems/network-delay-time/
> （同样适用卡码网 47，但 743 更经典）

---

## 应用场景

| 场景 | 用 Dijkstra | 用 Bellman-Ford |
|---|---|---|
| 所有边权 >= 0 | 首选，更快 | 也能用，但慢 |
| 存在负权边 | 结果错误，不能用 | 必须用 |
| 需要检测负权环 | 无法检测 | 天然支持 |
| 限制最多走 k 步 | 不自然支持 | 控制轮数即可 |

一句话选择标准：**边权全非负 → Dijkstra；有负权边 / 限步数 / 要检测负权环 → Bellman-Ford。**

---

## 核心思想

Dijkstra 用贪心（每次锁定最近节点），Bellman-Ford 放弃贪心，改用暴力：

> 一个有 n 个节点的图，任意最短路（不绕圈）最多经过 n-1 条边。
> 所以只要把所有边"松弛" n-1 轮，minDist 就一定收敛到正确答案。

**松弛**：对边 (u → v, 权重 w)，如果 `minDist[u] + w < minDist[v]`，就更新 `minDist[v]`。

为什么 n-1 轮够？每跑一轮，最短路信息就沿着边多传播一跳。n 个节点最长的非环路径有 n-1 条边，所以 n-1 轮后信息一定传遍所有节点。

---

## 算法步骤

```
1. 初始化 minDist 数组，源点 = 0，其余 = ∞
2. 重复 n-1 轮：
       对每一条边 (u, v, w)：
           if minDist[u] + w < minDist[v]:
               minDist[v] = minDist[u] + w
3. 返回 minDist[终点]
```

与 Dijkstra 三部曲的根本区别：没有"选最近节点"和"锁定"这两步，就是对所有边无脑暴力跑 n-1 轮。

---

## 模板题：LeetCode 743 Network Delay Time

**题意**：n 个节点的有向带权图，从节点 k 发出信号，求所有节点都收到信号的最短时间。即：求源点 k 到所有节点最短距离的最大值。

**为什么用 Bellman-Ford 也能过**：边权全为正，Dijkstra 和 Bellman-Ford 都适用。743 是练习两种算法的好模板。

---

## Java 实现

```java
import java.util.*;

class Solution {
    public int networkDelayTime(int[][] times, int n, int k) {
        final int INF = Integer.MAX_VALUE / 2; // 除2防止加法溢出
        int[] minDist = new int[n + 1];        // 节点编号1~n，开n+1大小
        Arrays.fill(minDist, INF);
        minDist[k] = 0;                         // 源点到自己距离为0

        // 松弛 n-1 轮
        for (int i = 1; i <= n - 1; i++) {
            // 每轮遍历所有边
            for (int[] edge : times) {
                int u = edge[0], v = edge[1], w = edge[2];
                // 松弛：如果经过u能让v更近，就更新
                if (minDist[u] != INF && minDist[u] + w < minDist[v]) {
                    minDist[v] = minDist[u] + w;
                }
            }
        }

        // 取所有节点最短距离的最大值
        int ans = 0;
        for (int v = 1; v <= n; v++) {
            if (minDist[v] == INF) return -1; // 有节点不可达
            ans = Math.max(ans, minDist[v]);
        }
        return ans;
    }
}
```

---

## Python 实现

```python
class Solution:
    def networkDelayTime(self, times: list[list[int]], n: int, k: int) -> int:
        INF = float('inf')
        min_dist = [INF] * (n + 1)  # 节点编号1~n
        min_dist[k] = 0

        # 松弛 n-1 轮
        for _ in range(n - 1):
            for u, v, w in times:  # 遍历所有边
                if min_dist[u] != INF and min_dist[u] + w < min_dist[v]:
                    min_dist[v] = min_dist[u] + w

        # 取最大值
        ans = max(min_dist[1:])
        return -1 if ans == INF else ans
```

---

## 代码关键点

**为什么 Java 用 `INF = Integer.MAX_VALUE / 2`？**
`minDist[u] + w` 如果 `minDist[u]` 是 `Integer.MAX_VALUE`，加法会整数溢出变成负数，导致错误更新。除以 2 留出空间，Python 用 `float('inf')` 不存在这个问题。

**与 Dijkstra 代码对比**：
- Dijkstra：两层 for 循环，内层找最小节点 + 更新邻居
- Bellman-Ford：两层 for 循环，外层控制轮数 + 内层遍历所有边

结构相似，但逻辑完全不同——Bellman-Ford 没有"选节点"这一步。

---

## 复杂度

| | 值 |
|---|---|
| 时间复杂度 | O(V·E)，V 为节点数，E 为边数 |
| 空间复杂度 | O(V)，只需要 minDist 数组 |

比 Dijkstra 朴素版 O(n²) 慢，但能处理负权边，这是它存在的价值。

---

## 常见错误

1. 轮数写成 `n` 而不是 `n-1`，会多做无效计算（第 n 轮若有更新说明有负权环）
2. Java 忘记用 `INF/2`，直接用 `Integer.MAX_VALUE` 导致溢出
3. 节点编号从 1 开始时数组开 `n` 大小（应开 `n+1`）
