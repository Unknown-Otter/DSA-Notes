# 78-Subsets & 90-Subsets-II

> [LeetCode 78. Subsets](https://leetcode.com/problems/subsets/)
> [LeetCode 90. Subsets II](https://leetcode.com/problems/subsets-ii/)

## 一、问题概述

**78. 子集**：给定一个**互不相同**的整数数组 `nums`，返回该数组所有可能的子集（幂集）。

**90. 子集 II**：给定一个**可能包含重复元素**的整数数组 `nums`，返回该数组所有可能的子集（幂集），解集**不能包含重复的子集**。

```
示例 78:  nums = [1,2,3]
输出:    [[], [1], [2], [3], [1,2], [1,3], [2,3], [1,2,3]]   共 2³ = 8 个

示例 90:  nums = [1,2,2]
输出:    [[], [1], [2], [1,2], [2,2], [1,2,2]]               共 6 个（去重后）
```

## 二、回溯三大模板对比（基础认知）

理解子集问题前，先把回溯**三大模板**放一起对比，看清子集在体系中的位置。

### 树形差异（以 `nums = [1,2,3]` 为例）

```
─────────────────────────────────────────────────────────────
子集（每个节点都收，2³=8）
─────────────────────────────────────────────────────────────
                   [] ★
        ┌──────────┼──────────┐
       [1]★      [2]★       [3]★
      ┌──┴──┐     │
   [1,2]★[1,3]★[2,3]★
      │
  [1,2,3]★


─────────────────────────────────────────────────────────────
组合 k=2（满足 size==k 才收，C(3,2)=3）
─────────────────────────────────────────────────────────────
                   []
        ┌──────────┼──────────┐
        [1]        [2]        [3]
      ┌──┴──┐      │
   [1,2]★[1,3]★ [2,3]★


─────────────────────────────────────────────────────────────
排列（满足 size==n 才收，3!=6）
─────────────────────────────────────────────────────────────
                       []
       ┌───────────────┼───────────────┐
      [1]             [2]             [3]
     ┌─┴─┐           ┌─┴─┐           ┌─┴─┐
   [1,2][1,3]      [2,1][2,3]      [3,1][3,2]
     │    │          │    │          │    │
[1,2,3]★[1,3,2]★[2,1,3]★[2,3,1]★[3,1,2]★[3,2,1]★
```

### 关键维度对比表

| 维度 | 子集 | 组合 | 排列 |
|---|---|---|---|
| **去重机制** | `start` | `start` | `used[]` |
| **for 起点** | `start`（递增） | `start`（递增） | 永远从 0 开始 |
| **顺序敏感** | 否（`{1,2}={2,1}`） | 否 | **是**（`[1,2]≠[2,1]`） |
| **收集时机** | 每个节点 | `size == k` | `size == n` |
| **是否 return** | 否（靠 for 自然终止） | 是 | 是 |
| **是否需要 used[]** | 不需要 | 不需要 | **需要** |
| **空集是答案吗** | 是（`[]`） | 否（除非 k=0） | 否（除非 n=0） |
| **结果数量（n=3）** | 8 | 3（k=2） | 6 |
| **公式** | 2ⁿ | C(n,k) | n! |

### 三大模板代码骨架

```java
// ──────── 子集 ────────
void backtrack(int start, List<Integer> path) {
    res.add(new ArrayList<>(path));         // 进门就收
    for (int i = start; i < n; i++) {
        path.add(nums[i]);
        backtrack(i + 1, path);
        path.remove(path.size() - 1);
    }
}

// ──────── 组合 ────────
void backtrack(int start, List<Integer> path) {
    if (path.size() == k) {                 // 到位才收
        res.add(new ArrayList<>(path));
        return;
    }
    for (int i = start; i < n; i++) {
        path.add(nums[i]);
        backtrack(i + 1, path);
        path.remove(path.size() - 1);
    }
}

// ──────── 排列 ────────
void backtrack(boolean[] used, List<Integer> path) {
    if (path.size() == n) {                 // 选满才收
        res.add(new ArrayList<>(path));
        return;
    }
    for (int i = 0; i < n; i++) {
        if (used[i]) continue;
        used[i] = true;
        path.add(nums[i]);
        backtrack(used, path);
        path.remove(path.size() - 1);
        used[i] = false;
    }
}
```

### 统一记忆法

> **三件事决定题目类型**：
> 1. **顺序重要吗** → 不重要用 `start`，重要用 `used[]`
> 2. **什么时候收** → 全程收（子集）/ 到位收（组合、排列）
> 3. **空集算吗** → 子集算（在函数入口收，自然包含 `[]`），其他不算

> **决策树**：
> ```
> 答案需要 [1,2] 和 [2,1] 都出现吗？
>   ├─ 是 → 排列模板（used[]，从 0 扫）
>   └─ 否 → 子集/组合系列（start 控制递增）
>           ├─ 收集所有路径 → 子集模板（入口收）
>           └─ 限定 size 或满足 sum → 组合模板（条件收）
> ```

## 三、78. 子集 —— 标准模板

### 思路

子集问题的本质是：**回溯树上每一个节点的 `path` 都是一个合法子集**，所以无条件收集每个节点。

- 用 `start` 控制下次递归的起始位置，保证元素**按下标递增**选取，避免 `{1,2}` 和 `{2,1}` 重复出现。
- **没有显式的终止 `return`**，靠 `for` 循环边界（`i < nums.length`）自然终止。

### 树形图（`nums = [1,2,3]`）

```
                          [] ★                              ← start=0
            ┌──────────────┼──────────────┐
           i=0            i=1            i=2
            │              │              │
          [1] ★          [2] ★          [3] ★               ← start=1,2,3
        ┌───┴───┐          │
       i=1     i=2        i=2
        │       │          │
      [1,2]★ [1,3]★     [2,3] ★                             ← start=2,3,3
        │
       i=2
        │
     [1,2,3] ★                                              ← start=3
```

### Java 代码

```java
class Solution {
    List<List<Integer>> result = new ArrayList<>();// 存放符合条件结果的集合
    LinkedList<Integer> path = new LinkedList<>();// 用来存放符合条件结果

    public List<List<Integer>> subsets(int[] nums) {
        subsetsHelper(nums, 0);
        return result;
    }

    private void subsetsHelper(int[] nums, int startIndex){
        result.add(new ArrayList<>(path));// 「遍历这个树的时候，把所有节点都记录下来，就是要求的子集集合」
        if (startIndex >= nums.length){ // 终止条件可不加
            return;
        }
        for (int i = startIndex; i < nums.length; i++){
            path.add(nums[i]);
            subsetsHelper(nums, i + 1);
            path.removeLast();
        }
    }
}
```

### 关键点

1. **收集位置**：`result.add(new ArrayList<>(path))` 必须放在**函数入口**，否则会丢掉空集 `[]`。
2. **`new ArrayList<>(path)`**：必须**深拷贝**，否则后续修改 `path` 会影响已加入 `result` 的引用。
3. **`startIndex` 的作用**：保证不回头选已经考虑过的元素，避免重复子集。
4. **终止条件可省略**：`for` 循环在 `startIndex >= nums.length` 时不会执行，自然返回。

## 四、90. 子集 II —— 同层去重

### 核心难点：什么是"同层去重"

`nums = [1,2,2]` 排序后，回溯树会出现两条路径：
- 选 `nums[1]=2`，跳过 `nums[2]=2` → `[1,2]`
- 跳过 `nums[1]=2`，选 `nums[2]=2` → `[1,2]`

这两条路径产生**相同的子集** `[1,2]`，需要去重。

但**同一条路径上**连续选两个 `2`（即 `[2,2]`）是合法的，不能去掉。

> **结论**：要去掉的是「**同一树层**」上重复值产生的分支，保留「**同一树枝**」上的重复值。

### 树形图（`nums = [1,2,2]`，排序后）

```
                          [] ★
            ┌──────────────┼──────────────┐
           i=0            i=1            i=2 ✗ 同层去重
            │              │            （nums[2]==nums[1]，跳过）
          [1] ★          [2] ★
        ┌───┴───┐          │
       i=1     i=2 ✗      i=2
        │     同层去重      │
      [1,2]★              [2,2] ★
        │
       i=2
        │
     [1,2,2] ★

         结果：[], [1], [1,2], [1,2,2], [2], [2,2]   共 6 个
```

**注意区别**：
- `[1] → [1,2]` 后再选 `2` 得到 `[1,2,2]`：这是**同一树枝**上的重复，**保留**。
- 根节点处 `i=1` 已选过 `2`，`i=2` 再选 `2`：这是**同一树层**上的重复，**去掉**。

### 思路

1. **必须先排序**：让相同的值相邻，才能用 `nums[i] == nums[i-1]` 判断重复。
2. **同层去重的条件**：`i > startIndex && nums[i] == nums[i-1]`
   - `i > startIndex`：保证不是该层第一个元素（第一个不算重复）
   - `nums[i] == nums[i-1]`：当前值与同层前一个值相同
3. 子集模板**没有显式终止**，所以这里也不需要 `return`。

### Java 代码（推荐：不使用 used 数组）

```java
class Solution {
    List<List<Integer>> result = new ArrayList<>();
    LinkedList<Integer> path = new LinkedList<>();

    public List<List<Integer>> subsetsWithDup(int[] nums) {
        Arrays.sort(nums); // 必须先排序，让重复值相邻
        subsetsWithDupHelper(nums, 0);
        return result;
    }

    private void subsetsWithDupHelper(int[] nums, int startIndex) {
        result.add(new ArrayList<>(path)); // 子集模板：每个节点都收

        for (int i = startIndex; i < nums.length; i++) {
            // 同层去重：i > startIndex 表示不是该层第一个
            if (i > startIndex && nums[i] == nums[i - 1]) {
                continue;
            }
            path.add(nums[i]);
            subsetsWithDupHelper(nums, i + 1);
            path.removeLast();
        }
    }
}
```

### Java 代码（备选：使用 used 数组）

`used[]` 写法更通用，**排列去重也用同一套**（LC 47），值得掌握。

```java
class Solution {
    List<List<Integer>> result = new ArrayList<>();
    LinkedList<Integer> path = new LinkedList<>();
    boolean[] used;

    public List<List<Integer>> subsetsWithDup(int[] nums) {
        Arrays.sort(nums);
        used = new boolean[nums.length];
        subsetsWithDupHelper(nums, 0);
        return result;
    }

    private void subsetsWithDupHelper(int[] nums, int startIndex) {
        result.add(new ArrayList<>(path));

        for (int i = startIndex; i < nums.length; i++) {
            // used[i-1] == false 说明同层 nums[i-1] 已用过（撤销了），需跳过
            // used[i-1] == true  说明同枝 nums[i-1] 还在 path 里，可以继续
            if (i > 0 && nums[i] == nums[i - 1] && !used[i - 1]) {
                continue;
            }
            path.add(nums[i]);
            used[i] = true;
            subsetsWithDupHelper(nums, i + 1);
            used[i] = false;
            path.removeLast();
        }
    }
}
```

### 两种写法对比

| 写法 | 去重判断 | 是否需要 used | 是否能扩展到排列 |
|---|---|---|---|
| `i > startIndex` | 简洁，逻辑直观 | 否 | **否**（排列没有 startIndex） |
| `!used[i-1]` | 通用，含义抽象 | 是 | **是**（同样适用于 LC 47） |

**理解 `!used[i-1]` 为什么能区分树层 vs 树枝**：

```
排序后：nums = [1, 2(a), 2(b)]
                      ↑i-1     ↑i

情况 A：树枝去重
  当前路径选了 2(a)，正在递归内部考虑 2(b)
  此时 used[i-1] = true（2(a) 在 path 里）
  → 不跳过，因为 [2(a), 2(b)] 是合法的 [2,2]

情况 B：树层去重
  根节点 for 循环，i=1 时选了 2(a) 又撤销
  现在 i=2，要看 2(b)
  此时 used[i-1] = false（2(a) 已撤销）
  → 跳过，因为这会和刚才 i=1 的分支生成重复结果
```

## 五、复杂度分析

| 题目 | 时间 | 空间 |
|---|---|---|
| 78. Subsets | O(n × 2ⁿ) | O(n)（递归栈 + path） |
| 90. Subsets II | O(n × 2ⁿ) | O(n) |

- 时间：最多 2ⁿ 个子集，每个子集复制到 result 需要 O(n)。
- 空间：不计输出空间，递归深度最多 n。

## 六、总结

1. **78 是子集模板的最纯粹形态**：进门就收，没有显式终止，靠 `start` 避免重复，靠 `for` 自然结束。
2. **90 = 78 + 同层去重**：在标准子集模板上加一行 `if (i > startIndex && nums[i] == nums[i-1]) continue;`，前提是**先排序**。
3. **去重的本质**：把"会产生相同结果的分支"在同一层内去掉，但保留树枝上的重复（因为一条树枝就是一个唯一组合）。
4. **`i > startIndex` vs `!used[i-1]`**：前者更简洁，后者更通用。子集/组合用前者足够，排列必须用后者。
5. **三大模板的统一框架**：
   - 顺序不敏感 + 全收 → 子集
   - 顺序不敏感 + 限定收 → 组合
   - 顺序敏感 + 选满收 → 排列
