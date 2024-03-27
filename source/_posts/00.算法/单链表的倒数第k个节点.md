---
title: 单链表的倒数第k个节点
date: 2023-06-16 11:21:58
tags: [leetcode,双指针]
categories: 算法
recommend: true
locate: 天津
cover: https://markdown-image-bed.oss-cn-beijing.aliyuncs.com/202306261740849.jpeg
---
# 单链表的倒数第k个节点

![a bird is perched on the side of a building](https://markdown-image-bed.oss-cn-beijing.aliyuncs.com/202306261740849.jpeg)

## 题目描述：

输入一个链表，输出该链表中倒数第k个节点。为了符合大多数人的习惯，本题从1开始计数，即链表的尾节点是倒数第1个节点。

例如，一个链表有 6 个节点，从头节点开始，它们的值依次是 1、2、3、4、5、6。这个链表的倒数第 3 个节点是值为 4 的节点。

 

示例：

给定一个链表: 1->2->3->4->5, 和 k = 2.

返回链表 4->5.

## 题解

双指针。

声明两个指针，分别是p1 和 p2；

先走p1，让p1先走k步

然后p1和p2同时走，直到p1为null的时候，停止，这时候，p2的就是待返回的结果。

```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode(int x) { val = x; }
 * }
 */
class Solution {
    public ListNode getKthFromEnd(ListNode head, int k) {
          // 先将p1指向head
        ListNode p1 = head;
        // 先让p1走k步
        for (int i = 0; i < k; i++) {
            p1 = p1.next;
        }
        ListNode p2 = head;
        // 再让p1和p2同时走n-k步
        while (p1 != null){
            p2 = p2.next;
            p1 = p1.next;
        }
        // p2 现在指向第 n - k + 1 个节点，即倒数第 k 个节点
        return p2;
    }
}
```

