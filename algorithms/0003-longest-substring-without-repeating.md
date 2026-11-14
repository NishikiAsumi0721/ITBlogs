# 0003. 无重复字符的最长子串

| 项 | 内容 |
|---|---|
| 难度 | 中等 |
| 标签 | 哈希表、字符串、滑动窗口 |
| 状态 | ✅ 已过 |
| 复习标记 | **会**（三次迭代到位） |
| 日期 | 2026-11-14 |
| 特别之处 | 第一版连编译都过不了（数组当集合用），最后只差一个 `if`→`while` |

---

## 题目

给定一个字符串 `s`，请你找出其中不含有重复字符的**最长子串**的长度。

```
输入: s = "abcabcbb"
输出: 3
解释: 最长无重复子串是 "abc"，长度 3

输入: s = "abba"
输出: 2      ← 这题的关键反例
```

---

## 最终解法（通过）

```java
class Solution {
    public int lengthOfLongestSubstring(String s) {
        Set<Character> window = new HashSet<>();
        int left = 0, res = 0;

        for (int right = 0; right < s.length(); right++) {
            char c = s.charAt(right);

            while (window.contains(c)) {        // ★ 缩到 c 被挤出窗口为止
                window.remove(s.charAt(left));
                left++;
            }

            window.add(c);
            res = Math.max(res, right - left + 1);   // ★ 每轮都更新，且 +1
        }
        return res;
    }
}
```

### 复杂度

| 项 | 值 |
|---|---|
| 时间 | **O(n)** —— 每个字符最多被加入一次、移出一次 |
| 空间 | **O(k)** —— `k` 是字符集大小，实际就是 O(1) |

---

## 关键洞察：`if` 必须是 `while`

**这是整题唯一的核心难点。**

```java
if (window.contains(c)) {      // ❌ 只缩一格
    window.remove(s.charAt(left));
    left++;
}

while (window.contains(c)) {   // ✅ 缩到不包含 c 为止
    window.remove(s.charAt(left));
    left++;
}
```

### 为什么 `if` 会错

`contains(c)` 为真只说明 **`c` 在窗口里某处**，**不代表 `c` 就在左边界**。

删掉 `s.charAt(left)`，可能根本没删到 `c`。

### 用 `"abba"` 证明

**`if` 版本**（错误）：

| 步 | right | 字符 | 窗口 Set | left | res | 实际窗口 |
|---|---|---|---|---|---|---|
| 1 | 0 | a | `{a}` | 0 | 1 | `"a"` |
| 2 | 1 | b | `{a,b}` | 0 | 2 | `"ab"` |
| 3 | 2 | **b** | 重复→删 `s[0]='a'`，left=1 → `{b}` | 1 | 2 | **`"bb"` 还有重复** |
| 4 | 3 | a | 没有 a→加入 → `{b,a}` | 1 | **3** ❌ | `"bba"` |

**第 3 步之后，Set 是 `{b}`，但实际窗口 `"bb"` 里有两个 `b`——Set 把重复掩盖了。**

第 4 步算出 `3`，正确答案是 `2`。

**`while` 版本**（正确）：

| 步 | right | 字符 | 内层 while | left | res |
|---|---|---|---|---|---|
| 3 | 2 | **b** | 删 `s[0]='a'`→`{b}`，还在！删 `s[1]='b'`→`{}`，停 | 2 | 1 |
| 4 | 3 | a | 没有 a，加入 `{a}` | 2 | **2** ✅ |

### 一句话记住

> **`if` 做的是「缩一次」，`while` 做的是「缩到条件不成立」。**
> **凡是「缩到……为止」，一律用 `while`。**

---

## 滑动窗口的固定节奏（背下来）

```java
int left = 0;
for (int right = 0; right < n; right++) {
    // 1. 右指针纳入新元素
    char c = s.charAt(right);

    // 2. 窗口不合法 → 收缩左指针，直到合法
    while (窗口不合法) {
        // 移出左边元素
        left++;
    }

    // 3. 此时窗口合法，更新答案
    res = Math.max(res, right - left + 1);
}
```

**`while` 的位置永远固定**：在「纳入新元素之后」、在「更新答案之前」。

### 不同题的「窗口不合法」定义

| 题 | 窗口不合法 | 内层动作 |
|---|---|---|
| **无重复子串** | 窗口内有重复字符 | `while (set.contains(c))` 删左边 |
| 最小覆盖子串 | 窗口**还没**覆盖目标 | 反向：`while (已覆盖)` 缩到刚好覆盖 |
| 长度最小的子数组 | 窗口和 ≥ target | `while (sum >= target)` 缩 |
| 字母异位词 | 窗口长度 > 目标长度 | 固定长度，每轮删一个 |

**判断依据：题目要「最长」还是「最短」**

- 求**最长** → 窗口**不合法**时缩（本题）
- 求**最短** → 窗口**合法**时缩（最小覆盖子串）

---

## 两种写法：Set vs Map

### 版本 A：Set（好理解，推荐先背）

```java
while (window.contains(c)) {
    window.remove(s.charAt(left));
    left++;                          // 一格一格移
}
```

**特点**：逻辑直白，每步都能说出含义。缺点是重复时可能要移很多格。

### 版本 B：Map（能直接跳）

```java
Map<Character, Integer> lastIndex = new HashMap<>();
int left = 0, res = 0;

for (int right = 0; right < s.length(); right++) {
    char c = s.charAt(right);

    // 上次出现过，且位置在当前窗口内 → 直接跳到它后面
    if (lastIndex.containsKey(c) && lastIndex.get(c) >= left) {
        left = lastIndex.get(c) + 1;      // ★ 一步到位
    }

    lastIndex.put(c, right);
    res = Math.max(res, right - left + 1);
}
```

**特点**：`left` 直接跳，不用一格一格移。但 `>= left` 这个判断很容易漏。

> ⚠️ **`lastIndex.get(c) >= left` 不能省。** 如果上次出现的位置**已经在 left 左边**（已经被窗口抛掉了），就不该动 `left`。省掉会导致 `left` 往回退。

**面试建议**：先写 Set 版（稳），如果面试官问优化，再说 Map 版的思路。

---

## 易错点清单

| # | 坑 | 后果 |
|---|---|---|
| 1 | **`if` 代替 `while`** | `"abba"` 返回 3（应为 2） |
| 2 | **长度忘了 `+1`** | `right - left` 是下标差，长度要 `+1` |
| 3 | **只在遇到重复时更新 `res`** | 全程无重复的字符串返回 0 |
| 4 | **`left = right`**（而不是跳到重复字符后一位） | 窗口变成空，前面攒的全丢 |
| 5 | **用 `char[]` 当集合** | 数组没有 `add`/`contains`，编译不过；而且定长 |
| 6 | **`s.length() - 2` 之类的缩边界** | 静默少处理几个字符，比崩溃更难发现 |
| 7 | **Map 版漏 `lastIndex.get(c) >= left`** | `left` 可能往回退 |
| 8 | 空串加 guard | **不需要**，`for` 循环天然返回 0 |

---

## 手推验证（复习时默写这遍）

`s = "abba"`，答案 **2**

| right | 字符 | 内层 while | window | left | 长度 | res |
|---|---|---|---|---|---|---|
| 0 | a | — | `{a}` | 0 | 1 | 1 |
| 1 | b | — | `{a,b}` | 0 | 2 | 2 |
| 2 | **b** | 删 a → `{b}`，仍有 b → 删 b → `{}` | `{b}` | 2 | 1 | 2 |
| 3 | a | — | `{b,a}` | 2 | 2 | **2** |

**再推 `"abcabcbb"`**（答案 3），重点看 `right=3` 时 `left` 从 0 跳到 1 之后，窗口为什么刚好是 `"bca"`。

---

## 三次迭代记录

### v1：数组当集合用，编译不过

```java
char[] c = new char[100];
c.add(s.charAt(fast));        // ❌ 数组没 add
c.contains(...)               // ❌ 数组没 contains
s.tochar(fast)                // ❌ 拼错，是 charAt
while (fast < s.length() - 2) // ❌ 少 2 格
```

**六个问题**：容器选错、API 用错、边界缩 2 格、`fast++` 位置错、`slow = fast` 错、`res` 只在重复时更新。

### v2：改用 Set，但 `if` 只缩一格

```java
if (window.contains(c1)) {    // ❌ 只缩一次
    window.remove(s.charAt(left));
    left++;
}
```

`"abba"` 返回 3。

### v3：`if` → `while` ✅ 通过

**一次迭代解决 6 个问题，只剩 1 个。**

---

## 进度记录

**这一版有一件事值得单独提**：你把第一版里那句注释掉的

```java
//if(s.length()==0)return 0;
```

**主动删掉了。**

这是四道题以来第一次**主动去掉多余的防御**——见 [代码习惯：习惯性加防御](00-代码习惯-习惯性加防御.md)。
之前移动零、盛最多水里都是习惯性加上，这次自己判断出"不需要"。

**习惯是可以改的，这一版就是证据。**

---

## 一句话总结

> **`if` 缩一次，`while` 缩到条件不成立。滑动窗口要的是后者——缩到重复字符被挤出窗口为止。**