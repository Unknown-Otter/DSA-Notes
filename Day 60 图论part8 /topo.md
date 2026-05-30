# 210. Course Schedule II — 拓扑排序（Kahn's Algorithm / BFS）

> https://leetcode.com/problems/course-schedule-ii/

## 题意

给定 `numCourses` 门课和先修关系 `prerequisites[i] = [a, b]`（上课 `a` 之前必须先上课 `b`），返回一种合法的学习顺序。若存在循环依赖，返回空数组。

---

## 核心思路：拓扑排序 = BFS + 入度

**本质是 BFS**，以"入度为 0"替代普通 BFS 里的"是否访问过"作为入队条件。

**两步循环：**

1. 找所有入度为 0 的节点加入队列（无前置依赖，可最先处理）
2. 取出队头节点，加入结果；将其所有邻居入度 `-1`，若某邻居入度变为 0，则入队

循环至队列为空。

**判断有环：** 结果集长度 < 节点总数 → 有环 → 返回 `[]`

---

## 关键细节

- `prerequisites[i] = [a, b]` 表示 `b → a`，**边方向是"先修 → 待修"**，入度加在 `a` 上
- "删边"不需要真正修改图，只需对邻居做 `inDegree -= 1`
- 时间复杂度 `O(V + E)`，空间 `O(V + E)`

---

## 复杂度

| 复杂度 | 值 |
|--------|------|
| 时间   | `O(V + E)` |
| 空间   | `O(V + E)` |

---

## Java 代码

```java
class Solution {
    public int[] findOrder(int numCourses, int[][] prerequisites) {
        // 构建邻接表和入度数组
        int[] inDegree = new int[numCourses];
        List<List<Integer>> adj = new ArrayList<>();
        for (int i = 0; i < numCourses; i++) {
            adj.add(new ArrayList<>());
        }

        for (int[] pre : prerequisites) {
            int course = pre[0];  // 待修课程
            int prereq = pre[1];  // 先修课程
            // prereq -> course：先上 prereq，才能上 course
            adj.get(prereq).add(course);
            inDegree[course]++;
        }

        // 把所有入度为 0 的课程加入队列
        Queue<Integer> queue = new LinkedList<>();
        for (int i = 0; i < numCourses; i++) {
            if (inDegree[i] == 0) queue.offer(i);
        }

        // BFS
        int[] result = new int[numCourses];
        int idx = 0;
        while (!queue.isEmpty()) {
            int cur = queue.poll();
            result[idx++] = cur;
            // 移除 cur，对其所有邻居入度减 1
            for (int next : adj.get(cur)) {
                if (--inDegree[next] == 0) {
                    queue.offer(next);
                }
            }
        }

        // 结果集长度不等于总课程数 → 有环
        return idx == numCourses ? result : new int[0];
    }
}
```

---

## Python 代码

```python
from collections import deque, defaultdict

class Solution:
    def findOrder(self, numCourses: int, prerequisites: list[list[int]]) -> list[int]:
        in_degree = [0] * numCourses
        adj = defaultdict(list)

        for course, prereq in prerequisites:
            # prereq -> course
            adj[prereq].append(course)
            in_degree[course] += 1

        # 入度为 0 的节点入队
        queue = deque(i for i in range(numCourses) if in_degree[i] == 0)
        result = []

        while queue:
            cur = queue.popleft()
            result.append(cur)
            for next_course in adj[cur]:
                in_degree[next_course] -= 1
                if in_degree[next_course] == 0:
                    queue.append(next_course)

        # 有环则结果集不完整
        return result if len(result) == numCourses else []
```

---

## 总结

1. **建图**：遍历 `prerequisites`，构建邻接表 + 统计每个节点入度
2. **初始化队列**：所有入度为 0 的节点入队
3. **BFS**：取出节点 → 加入结果 → 邻居入度 `-1` → 入度为 0 则入队
4. **判断有环**：`result.size() == numCourses` 则无环，否则返回空

**关联题目：** LC 207 课程表 I（只判断能否完成，不需要输出顺序，逻辑相同）
