# 0021. 合并两个有序链表

| 项 | 内容 |
|---|---|
| 难度 | 简单 |
| 标签 | 链表、递归 |
| 状态 | ✅ 已过（第 2 版） |
| 复习标记 | **会**（但要改成 O(1) 拼接版才算完全掌握） |
| 日期 | 2026-11-15 |
| 特别之处 | 第 1 版丢了整个结果链表；第 2 版对了但多花 O(n) 空间 |

> 🔗 链表套路第 2 题。**虚拟头节点（dummy）** 模板从这里建立。

---

## 题目

将两个升序链表合并为一个新的**升序**链表并返回。新链表是通过拼接给定的两个链表的所有节点组成的。

```
输入：l1 = [1,2,4], l2 = [1,3,4]
输出：[1,1,2,3,4,4]
```

---

## 我的第一版（错：结果链表整个丢了）

```java
public ListNode mergeTwoLists(ListNode list1, ListNode list2) {
    ListNode res = new ListNode();
    if (list1 == null) return list2;
    if (list2 == null) return list1;

    while (list1.next != null && list2.next != null) {   // ❌ 少判一格
        if (list1.val > list2.val) {
            res = new ListNode(list2.val);
            res = res.next;              // ❌ res 立刻变 null
            list2 = list2.next;
        } else {
            res = new ListNode(list1.val);
            res = res.next;              // ❌ 同上
            list1 = list1.next;
        }
    }
    if (list1.next != null) res = list1;   // ❌ 覆盖了前面拼好的
    if (list2.next != null) res = list2;
    return res;
}
```

### 为什么返回 `[2,4]` 而不是 `[1,2,3,4]`

以 `list1 = [1,3]`、`list2 = [2,4]` 为例：

| 步骤 | 发生了什么 |
|---|---|
| `res = dummy` | res 指向占位节点 |
| 进 while | `1.next=3` ✓、`2.next=4` ✓ |
| `1 <= 2` 分支 | — |
| `res = new ListNode(1)` | res 指向**新节点**，但**前一个节点的 next 没指向它** |
| `res = res.next` | 新节点的 next 是 null → **res = null** |
| `list1 = [3]` | — |
| while 判断 `list1.next` | `3.next = null` → 退出 |
| `if (list2.next != null)` | `2.next = 4` ✓ → **`res = list2 = [2,4]`** |
| return | **`[2,4]`** ❌ |

### 三个 bug

| # | bug | 后果 |
|---|---|---|
| 1 | `res` 既当头又当尾 | 新建的节点没被接上，`res` 变 null |
| 2 | `while (list.next != null)` | **最后一个节点永远不参与比较** |
| 3 | `res = list1` 覆盖 | 前面拼好的全丢 |

**根因：只用了 1 个指针，但它需要同时干"记住头"和"往后拼"两件事。**

---

## 我的第二版（对，但多花空间）

```java
public ListNode mergeTwoLists(ListNode list1, ListNode list2) {
    ListNode res = new ListNode(0);
    ListNode tail = res;

    if (list1 == null) return list2;      // ⚠️ 多余，见下文
    if (list2 == null) return list1;      // ⚠️ 多余

    while (list1 != null && list2 != null) {
        if (list1.val > list2.val) {
            tail.next = new ListNode(list2.val);   // ⚠️ 新建节点
            list2 = list2.next;
        } else {
            tail.next = new ListNode(list1.val);   // ⚠️ 新建节点
            list1 = list1.next;
        }
        tail = tail.next;
    }
    if (list1 != null) tail.next = list1;
    if (list2 != null) tail.next = list2;
    return res.next;
}
```

**这版逻辑是对的**，`dummy + tail` 也用对了，`while` 条件改对了，`return res.next` 也对了。

**三处可以更好：**

| # | 问题 | 说明 |
|---|---|---|
| 1 | 开头两个 guard | **多余**，尾部挂载逻辑已经覆盖空链表 |
| 2 | `new ListNode(val)` | 多花 O(m+n) 空间，且只复制 val，丢其他字段 |
| 3 | 混合状态 | 前段新建节点、后段用原节点，思维模型不统一 |

### 为什么 guard 是多余的

以 `list1 = null`、`list2 = [1,2]` 为例，**删掉 guard**：

| 步骤 | 结果 |
|---|---|
| while `(null != null && ...)` | false，不进循环 |
| `if (list1 != null)` | false，跳过 |
| `if (list2 != null)` | true → `tail.next = list2` |
| return `res.next` | **`[1,2]`** ✅ |

**一样的。**

---

## 标答：直接拼接，O(1) 空间

```java
class Solution {
    public ListNode mergeTwoLists(ListNode list1, ListNode list2) {
        ListNode dummy = new ListNode(0);
        ListNode tail = dummy;

        while (list1 != null && list2 != null) {
            if (list1.val <= list2.val) {
                tail.next = list1;        // ★ 直接接原节点
                list1 = list1.next;
            } else {
                tail.next = list2;
                list2 = list2.next;
            }
            tail = tail.next;
        }

        tail.next = (list1 != null) ? list1 : list2;   // 剩余整段挂上
        return dummy.next;
    }
}
```

### 复杂度

| 项 | 值 |
|---|---|
| 时间 | **O(m + n)** |
| 空间 | **O(1)** —— 只用了两个指针 |

### 两个版本对比

| | 我的第 2 版 | 标答（拼接） |
|---|---|---|
| 时间 | O(m+n) | O(m+n) |
| **空间** | **O(m+n)** | **O(1)** |
| 数据完整性 | ⚠️ 只复制 val | ✅ 原节点原样 |
| 代码行数 | 多 4 行 | 少 |

> **面试如果问「空间能不能优化」，就是在考这个。**

---

## 关键洞察：为什么必须是两个指针

> **`dummy` 负责「记住头在哪」，`tail` 负责「往后拼」。**
> **一个从头到尾不动，一个每拼一个节点前进一次。**

| 指针 | 作用 | 变化 |
|---|---|---|
| `dummy` | 占住头的位置 | **始终不变** |
| `tail` | 指向结果链表最后一个节点 | 每拼一个前进一次 |

**只用一个指针，它就得同时当"头"和"尾巴"，两头都不是。**

### 为什么需要 dummy

**因为新链表的头不确定**：可能是 `list1[0]`，也可能是 `list2[0]`，取决于谁小。

没有 dummy，你就得写「第一个节点特殊处理」的一堆判断。有了 dummy，**所有节点一视同仁**，最后 `return dummy.next` 就行。

> **虚拟头节点的全部价值：消除头部的特殊判断。**

---

## 什么时候用 dummy，什么时候不用

| 题 | 要不要 dummy | 为什么 |
|---|---|---|
| **21 合并链表** | ✅ **要** | 新头不确定 |
| 19 删除倒数第 N 个 | ✅ 要 | 可能要删头节点 |
| 2 两数相加 | ✅ 要 | 新头不确定 |
| **206 反转链表** | ❌ **不用** | `pre` 最后自然就是新头 |
| 141 环形链表 | ❌ 不用 | 不产生新链表 |

**判据：会不会产生一个"头不确定"的新链表。**

---

## 易错点清单

| # | 坑 | 后果 |
|---|---|---|
| 1 | **只用一个指针** | 头丢了或被覆盖 |
| 2 | `while (list.next != null)` | 最后一个节点不参与合并 |
| 3 | **忘了 `tail = tail.next`** | 所有节点都挂在 dummy 后面，链表只剩一个 |
| 4 | `return res` 而不是 `res.next` | 多返回一个值为 0 的 dummy |
| 5 | 尾部写成 `tail.next = list1; tail.next = list2` | 第二个覆盖第一个（虽然本题不会两个都非空） |
| 6 | 用 `new ListNode(val)` | 多花空间，丢其他字段 |
| 7 | 开头加空链表 guard | 多余，尾部挂载已覆盖 |

### 第 3 条最隐蔽

```java
tail.next = list1;
list1 = list1.next;
// tail = tail.next;   ← 第一次写这题忘了
```

**忘了这行，`tail` 永远指向 dummy**，第二次拼接就覆盖了第一次的结果，最后链表只剩最后一个节点。

---

## 手推验证

`list1 = [1,3]`、`list2 = [2,4]`

| 轮 | 比较 | 拼接 | tail 后 | list1 | list2 |
|---|---|---|---|---|---|
| 初始 | — | — | dummy | [1,3] | [2,4] |
| 1 | 1 ≤ 2 | tail→1 | 1 | [3] | [2,4] |
| 2 | 3 > 2 | tail→2 | 2 | [3] | [4] |
| 3 | 3 ≤ 4 | tail→3 | 3 | [] | [4] |
| 退出 | list1 为 null | — | — | — | — |
| 收尾 | — | `tail.next = list2` | — | — | — |

**结果 `dummy → 1 → 2 → 3 → 4`，返回 `dummy.next = [1,2,3,4]`** ✅

---

## 递归版（面试常追问）

```java
class Solution {
    public ListNode mergeTwoLists(ListNode l1, ListNode l2) {
        if (l1 == null) return l2;
        if (l1.val <= l2.val) {
            l1.next = mergeTwoLists(l1.next, l2);
            return l1;
        } else {
            l2.next = mergeTwoLists(l1, l2.next);
            return l2;
        }
    }
}
```

**时间 O(m+n)，空间 O(m+n)**（递归栈）。

> 递归版的「空判断」**是必要的**（递归出口），和迭代版开头那两个 guard 不一样。

---

## 一句话总结

> **合并链表 = dummy 记头 + tail 拼尾。两个指针分工明确，缺一个就丢链表。**

---

## 和反转链表对比（两个模板的差别）

| | 206 反转 | 21 合并 |
|---|---|---|
| 指针数 | **三个**（pre / cur / ne） | **两个**（dummy / tail） |
| dummy | ❌ 不用 | ✅ 要 |
| 核心动作 | 改 `next` 方向 | 接 `next` 指针 |
| 循环条件 | `cur != null` | `l1 != null && l2 != null` |
| 收尾 | 无 | **剩余整段挂载** |

**共同点：都要"存好再改"，都不能只用一个指针。**

---

## 链表进度

| 模板 | 题 | 状态 |
|---|---|---|
| 三指针反转 | 206 | ✅ |
| **虚拟头节点** | **21** | **✅** |
| 快慢指针判环 | 141 | ⬜ |
| 快慢指针找中点 | 234 | ⬜ |
| 双指针交叉走 | 160 | ⬜ |
| 快慢指针隔 N | 19 | ⬜ |
| 虚拟头 + 进位 | 2 | ⬜ |
| 环入口数学 | 142 | ⬜ |

**下一题建议 141 环形链表**——快慢指针入门，10 分钟。