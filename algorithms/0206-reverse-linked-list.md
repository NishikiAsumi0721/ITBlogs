# 0206. 反转链表

| 项 | 内容 |
|---|---|
| 难度 | 简单 |
| 标签 | 链表、递归 |
| 状态 | ✅ 已过（改后版本） |
| 复习标记 | **会** |
| 日期 | 2026-11-15 |
| 特别之处 | 自己想的版本会成环，改后是标准三指针 |

> 🔗 链表套路第一题。三个模板（反转 / 快慢指针 / 虚拟头节点）从这里开始建立。

---

## 题目

给你单链表的头节点 `head`，请你反转链表，并返回反转后的链表。

```
输入：head = [1,2,3,4,5]
输出：[5,4,3,2,1]
```

---

## 我的第一版（有环，不通过）

```java
public ListNode reverseList(ListNode head) {
    ListNode l = new ListNode();
    ListNode r = new ListNode();
    if (head.next.next == null) {      // ❌ head 或 head.next 为 null 时 NPE
        head.next.next = head;
        head.next = null;
        return head;
    }
    while (head.next && head.next.next) {   // ❌ Java 里 && 不能用于对象
        l.next = head;
        r.next = head.next.next;
        head.next.next = l.next;       // ❌ 这里直接成环
        head.next = l;
        head = r.next;
    }
    return head;
}
```

### 三个层次的问题

| # | 问题 | 后果 |
|---|---|---|
| 1 | `while (head.next && head.next.next)` | **编译不过**，Java 的 `&&` 只接受 boolean |
| 2 | `if (head.next.next == null)` 没判 null | 空链表 / 单节点时 **NPE** |
| 3 | **指针改完变成环** | 逻辑死掉 |

### 为什么一定会成环

以 `1 → 2 → 3` 为例，执行到第三行：

```java
head.next.next = l.next;
```

此时 `l.next == head == 节点1`，所以这行等价于：

```java
2.next = 1;
```

但 `1.next` 还是 `2` —— 于是：

```
1 → 2 → 1 → 2 → ...
```

**成了环。** 后面所有操作都在环上打转，永远出不来。

### 根因：想一步改两条边

你的思路是「同时调整 `l` 和 `r` 两个方向」，但链表的指针**只能一条一条改，而且改之前必须先把「后路」存下来**。

---

## 正确版本：三指针

```java
class Solution {
    public ListNode reverseList(ListNode head) {
        ListNode pre = null;
        ListNode cur = head;

        while (cur != null) {
            ListNode ne = cur.next;   // ① 先存后继（后路）
            cur.next = pre;           // ② 改指针，反向
            pre = cur;                // ③ pre 前进
            cur = ne;                 // ④ cur 前进
        }
        return pre;                   // cur 已为 null，pre 是新头
    }
}
```

### 复杂度

| 项 | 值 |
|---|---|
| 时间 | **O(n)** —— 每个节点走一次 |
| 空间 | **O(1)** —— 只用三个指针 |

---

## 关键洞察：顺序不能反

> **核心矛盾：改了 `cur.next`，就再也找不到后面的节点了。**

所以必须**先存后继，再改指针**：

```
❌ 错：cur.next = pre;  然后  ne = cur.next;   ← 此时 ne 已经是 pre 了
✅ 对：ne = cur.next;   然后  cur.next = pre;
```

**这四行代码的顺序是死的，不能调换。** 面试时如果被问"能不能先改指针"，就是考这个。

### 一个更本质的说法

反转链表做的是：**让每个节点指回它的前驱。**

要做到这点，必须同时持有「前驱」和「当前」两个信息。而改完当前节点的指针后，就得靠**提前存好的 `ne`** 才能继续往前走。

**三指针 = 前驱 + 当前 + 后继。缺任何一个都会断链或成环。**

---

## 为什么不需要虚拟头节点

虚拟头节点（dummy）用在**会改变「头」的场景**：删除、合并、插入。

反转是**整个链表翻转**，最后返回的 `pre` 自然就是新头，不需要 dummy 占位。

| 场景 | 要不要 dummy |
|---|---|
| 反转 | ❌ 不用 |
| 删除第 N 个 | ✅ 要（可能要删头） |
| 合并两个链表 | ✅ 要（新头不确定） |

**这是两个不同的模板，别混。**

---

## 递归版（面试常追问）

```java
class Solution {
    public ListNode reverseList(ListNode head) {
        if (head == null || head.next == null) return head;   // 出口：空 或 单节点

        ListNode newHead = reverseList(head.next);   // 先反转后面的

        head.next.next = head;    // 让「我的后继」指向我
        head.next = null;         // 断开我原来的方向，否则成环

        return newHead;
    }
}
```

**复杂度**：时间 O(n)，**空间 O(n)**（递归栈）。

### 递归版最容易忘的一行

```java
head.next = null;
```

**不写这行就成环。** 因为 `head.next.next = head` 之后，原来的 `head.next` 还指回去，就变成了 `2→1→2`。

> 面试追问「递归有什么问题」：栈深度等于链表长度，**链表很长时会栈溢出**。所以工程上一般用迭代。

---

## 模板速记

```java
// 反转链表：三指针，四步
ListNode pre = null, cur = head;
while (cur != null) {
    ListNode ne = cur.next;   // 存后路
    cur.next = pre;           // 反向
    pre = cur;                // 前进
    cur = ne;                 // 前进
}
return pre;
```

**记忆口诀：存、反、移、移。**

---

## 易错点清单

| # | 坑 | 后果 |
|---|---|---|
| 1 | **先改指针后存后继** | 断链，后面的节点全丢 |
| 2 | **循环写成 `cur.next != null`** | 最后一个节点没反转 |
| 3 | **返回 `head` 而不是 `pre`** | 返回的是原来的尾巴（现在是头） |
| 4 | 递归版忘写 `head.next = null` | 成环 |
| 5 | 反转用虚拟头节点 | 多余，还可能搞乱指针 |
| 6 | 没判空 | `head == null` 或单节点时 NPE |
| 7 | 用 `&&` 连对象 | `head.next && head.next.next` 编译不过，不合法的操作数类型 |

### 第 2 条要特别说

```java
while (cur != null)      // ✅ 处理到最后一个节点
while (cur.next != null) // ❌ 最后一个节点没被处理
```

**反转必须让每个节点都执行一次「反向」操作，包括最后一个。**

---

## 手推验证

`head = [1, 2, 3]`

| 轮 | cur | ne | cur.next = pre 后 | pre | cur |
|---|---|---|---|---|---|
| 初始 | 1 | — | — | null | 1 |
| 1 | 1 | 2 | 1→null | 1 | 2 |
| 2 | 2 | 3 | 2→1 | 2 | 3 |
| 3 | 3 | null | 3→2 | 3 | null |
| 结束 | null | — | — | **3** | null |

**返回 `pre = 3`，链表变成 `3 → 2 → 1 → null`** ✅

---

## 一句话总结

> **反转链表 = 让每个节点指回前驱。所以必须同时拿着前驱、当前、后继，而且「存后继」必须写在「改指针」之前。**

---

## 这道题和后面 7 道链表题的关系

反转是链表题的**地基**，后面好几题都要用它：

| 题 | 怎么用反转 |
|---|---|
| 234 回文链表 | 找中点 + **反转后半** + 比较 |
| 92 反转链表 II | 反转**指定区间**（局部反转） |
| 25 K 个一组翻转 | 每 K 个做一次局部反转 |
| 143 重排链表 | 找中点 + **反转后半** + 合并 |

**所以这题必须能闭眼默写。** 后面几题都是在它上面加条件。