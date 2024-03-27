---
title: 反转链表I
date: 2023-07-07 10:50:06
tags: [递归, 链表, leetcode]
categories: [算法]
recommend: true
locate: [天津]
cover: https://markdown-image-bed.oss-cn-beijing.aliyuncs.com/202307071047886.jpeg
comment: 是
keywords: 
---
---
title: 反转链表I
date: 2023-07-07 10:48:21
tags: [递归, 链表, leetcode]
categories: [算法]
recommend: true
locate: [天津]
cover: https://markdown-image-bed.oss-cn-beijing.aliyuncs.com/202307071047886.jpeg
comment: 是
keywords: 
---
# 反转链表I

![a view of a mountain range at sunset](https://markdown-image-bed.oss-cn-beijing.aliyuncs.com/202307071047886.jpeg)

给定单链表的头节点 head ，请反转链表，并返回反转后的链表的头节点。

 ![img](https://assets.leetcode.com/uploads/2021/02/19/rev1ex1.jpg)

示例 1：

输入：head = [1,2,3,4,5]
输出：[5,4,3,2,1]
示例 2：


输入：head = [1,2]
输出：[2,1]
示例 3：

输入：head = []
输出：[]

## 题解

![img](https://markdown-image-bed.oss-cn-beijing.aliyuncs.com/202307071037561.png)

```java
class Solution {
    public ListNode reverseList(ListNode head) {
        ListNode cur = head, pre = null;
        while(cur != null) {
            ListNode tmp = cur.next; // 暂存后继节点 cur.next
            cur.next = pre;          // 修改 next 引用指向
            pre = cur;               // pre 暂存 cur
            cur = tmp;               // cur 访问下一节点
        }
        return pre;
    }
}
```

### 递归

不太懂递归：

```java
ListNode reverse(ListNode head){
        if (head == null || head.next == null){
            return head;
        }
        ListNode last = reverse(head.next);
        head.next.next = head;
        head.next = null;
        return last;
    }
```

第二种回溯方法：

```java
class Solution {
    public ListNode reverseList(ListNode head) {
        return recur(head, null);    // 调用递归并返回
    }
    private ListNode recur(ListNode cur, ListNode pre) {
        if (cur == null) return pre; // 终止条件
        ListNode res = recur(cur.next, cur);  // 递归后继节点
        cur.next = pre;              // 修改节点引用指向
        return res;                  // 返回反转链表的头节点
    }
}
```

