# LC 46 & 47 — Permutations I & II

> LC 46: https://leetcode.com/problems/permutations/
> LC 47: https://leetcode.com/problems/permutations-ii/

---

## LC 46 — Permutations（无重复元素全排列）

### 题意

给定一个**不含重复数字**的数组 `nums`，返回所有可能的全排列。

### 核心思路

全排列和组合/子集不同：**每一层都要重新扫整个数组**，靠 `used[]` 标记"当前路径已经选了哪些位置"，而不是靠 `startIndex` 往后切。

### 递归树（`nums = [1, 2, 3]`）

```
                        []
           /             |             \
         [1]            [2]            [3]
        /   \          /   \          /   \
      [1,2] [1,3]  [2,1] [2,3]  [3,1] [3,2]
       |     |      |     |      |     |
    [1,2,3][1,3,2][2,1,3][2,3,1][3,1,2][3,2,1]
```

每一层 for 从 `i=0` 扫到末尾，`used[i]=true` 的跳过。

### 正确代码

```java
class Solution {

    List<List<Integer>> result = new ArrayList<>();
    LinkedList<Integer> path = new LinkedList<>();
    boolean[] used;

    public List<List<Integer>> permute(int[] nums) {
        if (nums.length == 0) return result;
        used = new boolean[nums.length];
        helper(nums);
        return result;
    }

    private void helper(int[] nums) {
        if (path.size() == nums.length) {
            result.add(new ArrayList<>(path));
            return;
        }

        for (int i = 0; i < nums.length; i++) {
            if (used[i]) continue;          // 同一路径不重复选同一位置

            used[i] = true;
            path.add(nums[i]);
            helper(nums);                   // 不传 startIndex，每次从 0 开始
            path.removeLast();
            used[i] = false;
        }
    }
}
```

### 关键点

| 要点 | 说明 |
|---|---|
| 不传 `startIndex` | 全排列每层都要看全数组 |
| `used[i]` | 防止同一路径重复选同一位置 |
| 不需要排序 | 元素本身无重复，不需要去重 |

### 复杂度

- **时间**：O(n × n!)，共 n! 个排列，每个长度为 n
- **空间**：O(n)，递归深度 + path + used 数组

---

---

## LC 47 — Permutations II（含重复元素全排列）

### 题意

给定一个**可能含重复数字**的数组 `nums`，返回所有**不重复**的全排列。

### 与 LC 46 的关系

LC 47 = LC 46 的框架 + **两处新增**：
1. `Arrays.sort(nums)` —— 排序，让相同元素相邻
2. 去重剪枝条件 —— 同一层跳过值相同的重复分支

---

### 为什么需要额外去重？

以 `[1a, 1b, 2]`（两个1用a/b区分位置）为例：

```
不加去重时会产生：

路径1：选1a → 选1b → 选2   结果 [1,1,2]
路径2：选1b → 选1a → 选2   结果 [1,1,2]  ← 重复！
```

`used[i]` 只能防止同一路径选同一位置，**无法阻止**两条不同路径产生相同结果。

---

### 去重条件详解

```java
if (i > 0 && nums[i] == nums[i-1] && !used[i-1]) continue;
```

逐项拆解：

| 子条件 | 作用 |
|---|---|
| `i > 0` | 防止 `nums[i-1]` 越界 |
| `nums[i] == nums[i-1]` | 当前元素和前一个值相同 |
| `!used[i-1]` | 前一个相同元素已被撤回（说明是同一层） |

**`!used[i-1]` 的本质：**

```
used[i-1] = true  → nums[i-1] 在当前路径里（上层选的）→ 不同分支，允许选
used[i-1] = false → nums[i-1] 已撤回（同层用过又回溯了）→ 重复分支，跳过
```

---

### 递归树对比（`nums = [1, 1, 2]`，排序后）

```
不去重（LC46框架直接用）：
                []
        /        |        \
      [1a]      [1b]      [2]        ← 1a 和 1b 值相同，产生重复
      ...        ...      ...

去重后（LC47）：
                []
        /        |        \
      [1]       跳过      [2]        ← i=1时 nums[1]==nums[0] && !used[0] → 跳过
      ...                 ...

最终结果（无重复）：
[1,1,2]  [1,2,1]  [2,1,1]
```

---

### 两个 `continue` 的分工

| 条件 | 过滤对象 | 解决问题 |
|---|---|---|
| `if (used[i]) continue` | 同一路径中的重复位置 | 防止 [1,1,...] 选同一个位置两次 |
| `if (i>0 && nums[i]==nums[i-1] && !used[i-1]) continue` | 同一层的重复值分支 | 防止不同路径产生相同排列 |

**二者缺一不可，目的完全不同。**

---

### 错误代码分析

```java
// ❌ 原始错误代码
public void helper(int[] nums, int startIndex) {
    for (int i = startIndex; i < nums.length; i++) {
        if (nums[i] == nums[i-1] && !used[i-1]) {  // 问题①②③
            continue;
        }
        used[i] = true;
        path.add(nums[i]);
        helper(nums, i+1);   // 问题④
        path.removeLast();
        used[i] = false;
    }
}
```

| # | 问题 | 原因 | 修复 |
|---|---|---|---|
| ① | `nums[i-1]` 越界 | `i=0` 时访问 `nums[-1]` | 加 `i > 0` 判断 |
| ② | 去重条件不完整 | 缺少 `i > 0` 保护 | `i > 0 && nums[i]==nums[i-1] && !used[i-1]` |
| ③ | 缺少 `used[i]` 跳过 | 同一路径会重复选同一位置 | 加 `if (used[i]) continue` |
| ④ | 传了 `startIndex` | 全排列不该限制起始位置 | 改为 `helper(nums)`，for 从 `0` 开始 |
| ⑤ | `used` 未初始化 | `used[]` 声明但未 `new` | `used = new boolean[nums.length]` |
| ⑥ | 未排序 | 去重前提是相同元素相邻 | `Arrays.sort(nums)` |

---

### 正确代码

```java
class Solution {

    List<List<Integer>> result = new ArrayList<>();
    LinkedList<Integer> path = new LinkedList<>();
    boolean[] used;

    public List<List<Integer>> permuteUnique(int[] nums) {
        used = new boolean[nums.length];    // ✅ 初始化
        Arrays.sort(nums);                  // ✅ 排序（去重前提）
        helper(nums);
        return result;
    }

    private void helper(int[] nums) {
        if (path.size() == nums.length) {
            result.add(new ArrayList<>(path));
            return;
        }

        for (int i = 0; i < nums.length; i++) {
            if (used[i]) continue;          // ✅ 跳过同路径已用位置

            // ✅ 同层去重：前一个相同元素已撤回，说明同层已走过，跳过
            if (i > 0 && nums[i] == nums[i - 1] && !used[i - 1]) continue;

            used[i] = true;
            path.add(nums[i]);
            helper(nums);                   // ✅ 不传 startIndex
            path.removeLast();
            used[i] = false;
        }
    }
}
```

---

## LC 46 vs LC 47 对比总结

| | LC 46 Permutations | LC 47 Permutations II |
|---|---|---|
| 输入 | 无重复元素 | 可能含重复元素 |
| 需要排序 | ❌ | ✅ |
| `used[i]` 跳过 | ✅ | ✅ |
| 同层去重条件 | ❌ | ✅ |
| `startIndex` | ❌（全排列不用）| ❌ |
| 时间复杂度 | O(n × n!) | O(n × n!)（最坏） |

---

## 模板记忆口诀

```
全排列框架：
  - used[] 标记当前路径
  - for 从 0 开始，used[i] 跳过
  - 不传 startIndex

有重复元素再加：
  - 先排序
  - i > 0 && nums[i]==nums[i-1] && !used[i-1] → 同层去重
```
