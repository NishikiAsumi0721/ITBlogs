# 0438. 找到字符串中所有字母异位词

| 项 | 内容 |
|---|---|
| 难度 | 中等 |
| 标签 | 哈希表、字符串、滑动窗口 |
| 状态 | ✅ 已过 |
| 复习标记 | **会**（三次迭代，从 Set 到计数是关键升级） |
| 日期 | 2026-11-14 |
| 特别之处 | v1 语义错（Set 判异位词）+ 会崩；v2 复制粘贴忘改变量导致恒返回 true |

---

## 题目

给定两个字符串 `s` 和 `p`，找到 `s` 中所有 `p` 的**字母异位词**的子串，返回这些子串的起始索引。不考虑答案输出的顺序。

**字母异位词**：由相同字母重排列形成的字符串，且每个字符出现的**次数完全相同**。

```
输入: s = "cbaebabacd", p = "abc"
输出: [0,6]
解释:
起始索引 0 的子串 "cba" 是 "abc" 的异位词
起始索引 6 的子串 "bac" 是 "abc" 的异位词
```

---

## 最终解法（通过）

```java
class Solution {
    public List<Integer> findAnagrams(String s, String p) {
        List<Integer> res = new ArrayList<>();
        int n = s.length(), m = p.length();

        int[] need = new int[26];
        for (char c : p.toCharArray()) need[c - 'a']++;

        int[] window = new int[26];

        for (int right = 0; right < n; right++) {
            window[s.charAt(right) - 'a']++;              // 1. 右边进

            if (right >= m) {                             // 2. 窗口超长，左边出
                window[s.charAt(right - m) - 'a']--;
            }

            if (right >= m - 1 && isSame(window, need)) { // 3. 长度够了才比
                res.add(right - m + 1);                   // 起始下标
            }
        }
        return res;
    }

    private boolean isSame(int[] a, int[] b) {
        for (int i = 0; i < 26; i++) if (a[i] != b[i]) return false;
        return true;
    }
}
```

### 复杂度

| 项 | 值 |
|---|---|
| 时间 | **O(26n) = O(n)** —— 每个字符进出窗口各一次，每次比较 26 个桶 |
| 空间 | **O(1)** —— 两个固定 26 长度的数组 |

---

## 关键洞察 ①：`Set` 判不了异位词，必须用计数

**这是这题最大的认知分水岭，也是 v1 挂掉的根因。**

### 为什么 Set 是错的

字母异位词的定义是「**每个字符出现的次数完全相同**」。`Set` 只记录「有哪些字符」，**把次数丢掉了**。

**反例**：

| 字符串 | 字符 | Set | 计数 |
|---|---|---|---|
| `"aab"` | {a,b} | `{a,b}` | a:2, b:1 |
| `"abb"` | {a,b} | `{a,b}` | a:1, b:2 |

**Set 完全相同，但不是异位词。**

### 正确做法

```java
int[] cnt = new int[26];
for (char c : a.toCharArray()) cnt[c - 'a']++;
for (char c : b.toCharArray()) cnt[c - 'a']--;   // 一个加、一个减
for (int x : cnt) if (x != 0) return false;       // 全 0 才是异位词
return true;
```

### 触发换容器的关键词

**看到这几个词，一律用「计数」而不是「集合」**：

> **次数 / 异位 / 排列 / 频率 / 出现几次**

| 需求 | 容器 |
|---|---|
| 判断「有哪些字符」（去重、出现过没） | `Set` |
| **判断「每个字符出现几次」** | **`int[26]` 或 `Map<Character,Integer>`** |

---

## 关键洞察 ②：定长窗口 vs 可变窗口

同样是滑动窗口，这题和 [0003 无重复子串](0003-longest-substring-without-repeating.md) 是**两种不同的形态**。

| | 0003 无重复子串 | **0438 字母异位词** |
|---|---|---|
| 窗口长度 | **可变**（跟着重复字符伸缩） | **固定 = p.length()** |
| 左指针怎么动 | `while` 缩到窗口合法 | **每轮固定移一格** |
| 判断依据 | 窗口内有无重复 | **字符计数是否相等** |
| 窗口数据结构 | `Set` | **`int[26]` 计数** |
| 收缩方式 | 不确定几次 → **`while`** | **确定一次 → `if`** |

### 识别信号

> **题目出现「p 的异位词 / 排列 / 包含 p 的所有字符」+ 目标长度固定**
> → **定长滑动窗口 + 计数数组**

### 一个通用判断

| 题目求什么 | 窗口怎么动 |
|---|---|
| **最长**（无重复子串） | 不合法时 `while` 缩 |
| **最短**（最小覆盖子串） | 合法时 `while` 缩 |
| **固定长度**（本题） | 每轮固定进一个、出一个 |

---

## 关键洞察 ③：`substring` 的代价

v2 是能算出正确答案的版本，但写成了这样：

```java
for (index = 0; index < s.length() - p.length() + 1; index++) {
    String sub = s.substring(index, index + p.length());   // 每个窗口复制一次
    if (findEXWord(sub, p)) list.add(index);               // 再数一遍
}
```

**复杂度 O(n × m)**，`n`、`m` 都能到 3 万 → 最坏 **9 亿次字符操作**。

### 慢在哪

| 操作 | 代价 | 优化 |
|---|---|---|
| `substring` 创建新字符串 | **O(m) 拷贝** | **不截串**，直接用下标访问原串 |
| 每个窗口重新数一遍 | **O(m)** | **增量更新**：进一个 +1，出一个 -1 |

```java
window[s.charAt(right) - 'a']++;          // 只加 1 个
if (right >= m) window[s.charAt(right - m) - 'a']--;   // 只减 1 个
```

**永远只动两个字符，而不是重新数整串。** 这就是 O(n·m) → O(n) 的来源。

> **通用规律：滑动窗口里不要 `substring`。** 用下标 `s.charAt(i)` 访问，避免每次复制。

---

## 附带：`Iterator.next()` 只能调一次（Java 通用坑）

v1 里有这段代码，和本题算法无关，但**是一个非常典型的 Java 坑**：

```java
Iterator<Character> it = b1.iterator();
while (it.hasNext()) {
    if (a1.contains(it.next())) {      // ← 第 1 次 next()
        a1.remove(it.next());          // ← 第 2 次 next()！跳到下一个了
    }
}
```

### 两个后果

**① 删错了元素**：判断的是元素 A，删的是元素 B。

**② 抛 `NoSuchElementException`**：`next()` 每调一次指针前移一格。一轮消耗 2 个元素，**元素个数为奇数时，最后一轮第二次 `next()` 没有下一个**。

`s = "abc", p = "abc"` 这种最基础的用例直接崩。

### 正确写法

```java
while (it.hasNext()) {
    Character c = it.next();      // ← 只取一次，存成变量
    if (a1.contains(c)) {
        a1.remove(c);             // ← 用变量
    }
}
```

> **规则：`while (it.hasNext())` 的循环体里，`next()` 只能出现一次。**
> 需要多次使用就存变量。

---

## 易错点清单

| # | 坑 | 后果 |
|---|---|---|
| 1 | **用 `Set` 判异位词** | `"aab"` 和 `"abb"` 判为相同，全错 |
| 2 | **`Iterator` 里 `next()` 调两次** | 删错元素 + 抛 `NoSuchElementException` |
| 3 | **复制粘贴循环忘了改变量** | `for (c : a) cnt++; for (c : a) cnt--;` → 恒返回 true |
| 4 | **`substring` 上界写 `index + m - 1`** | 截出的串少一个字符（`substring` 右端是开区间） |
| 5 | **每个窗口 `substring`** | O(n·m)，大规模输入超时 |
| 6 | **`if (right >= m)` 写成 `> m`** | 窗口长度变成 m+1 |
| 7 | **结果下标写成 `right - m`** | 少了 `+1` |

### 第 3 条单独说：复制粘贴没改全

```java
for (char c : a.toCharArray()) cnt[c - 'a']++;
for (char c : a.toCharArray()) cnt[c - 'a']--;   // ❌ 应该是 b
```

**同一个字符串加了又减，`cnt` 恒为全 0 → `findEXWord` 对任何输入都返回 `true`。**

**症状**：编译通过、逻辑自洽、但输出"所有位置"。
**信号**：两个相似的循环头，变量名一样。

> **检查方法：写完两个相似的循环，先看循环头是不是复制来的。是就逐个对变量名。**

---

## 手推验证（复习时默写这遍）

`s = "cbaebabacd"`, `p = "abc"` → `need = {a:1, b:1, c:1}`，`m = 3`

| right | 字符 | window 操作 | window 内容 | 命中 |
|---|---|---|---|---|
| 0 | c | 加 c | c:1 | — |
| 1 | b | 加 b | b:1, c:1 | — |
| 2 | a | 加 a | a:1, b:1, c:1 | ✅ **索引 0** |
| 3 | e | 加 e，减 s[0]=c | a:1, b:1, e:1 | — |
| 4 | b | 加 b，减 s[1]=b | a:1, b:1, e:1 | — |
| 5 | a | 加 a，减 s[2]=a | a:1, b:1, e:1 | — |
| 6 | b | 加 b，减 s[3]=e | a:1, b:2 | — |
| 7 | a | 加 a，减 s[4]=b | a:2, b:1 | — |
| 8 | c | 加 c，减 s[5]=a | a:1, b:1, c:1 | ✅ **索引 6** |
| 9 | d | 加 d，减 s[6]=b | a:1, c:1, d:1 | — |

**结果 `[0, 6]` ✅**

---

## 可选优化：把 26 次比较降成 1 次

现在每个窗口都跑 `isSame` 做 26 次比较，总复杂度 O(26n)。**能过。**

如果面试官问"还能更快吗"，答案是**维护一个"匹配计数"**：

```
same = 有多少个字母的计数恰好相等（初始为 26）

每次改动一个字母 x：
  window[x] 变化前后，分别判断「改之前是否等于 need[x]」「改之后是否等于」
  → 用这两个判断更新 same

最后只需判断 same == 26
```

**从 O(26n) 降到严格 O(n)。** 常数 26 影响不大，但这体现的是「能不能消掉常数」的意识。

---

## 一句话总结

> **异位词 = 字符计数完全相同，不是集合相同。定长窗口每轮只动两个字符，不要 substring。**