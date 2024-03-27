---
title: 删除链表的倒数第N个节点
date: 2023-07-07 11:06:39
tags: [链表, leetcode]
categories: [算法]
recommend: true
locate: [天津]
cover: https://markdown-image-bed.oss-cn-beijing.aliyuncs.com/202307071106271.jpeg
comment: 是
keywords: 
---
# 删除链表的倒数第N个节点

![an aerial view of a sandy beach and ocean](https://markdown-image-bed.oss-cn-beijing.aliyuncs.com/202307071106271.jpeg)

给你一个链表，删除链表的倒数第 `n` 个结点，并且返回链表的头结点。

**示例 1：**

![img](https://assets.leetcode.com/uploads/2020/10/03/remove_ex1.jpg)

```
输入：head = [1,2,3,4,5], n = 2
输出：[1,2,3,5]
```

**示例 2：**

```
输入：head = [1], n = 1
输出：[]
```

**示例 3：**

```
输入：head = [1,2], n = 1
输出：[1]
```

与链表的倒数第K个节点类似，只不过在得到倒数第K个节点之后，将该节点删除。

```java
 public ListNode removeNthFromEnd(ListNode head, int n) {
        ListNode dummy = new ListNode(-1);
        dummy.next = head;
        // 先利用双指针找到倒数第N个节点
        ListNode p1 = dummy;
        // p1 先走n步
        for (int i = 0; i < n + 1; i++) {
            p1 = p1.next;
        }
        ListNode p2 = dummy;
        // 再一起走
        while (p1 != null){
            p2 = p2.next;
            p1 = p1.next;
        }
        // 现在p2指的节点就是倒数第N个节点
        // 开始删除p2节点
        p2.next = p2.next.next;
        return dummy.next;
    }
```

