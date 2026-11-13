# 19. 删除链表的倒数第 N 个结点

[题目](https://leetcode.cn/problems/remove-nth-node-from-end-of-list/) · Java

## 30 秒速记

快慢指针从 dummy 出发。快指针先走 n 步，再一起走到快指针停在尾节点。此时慢指针是待删节点的前驱。绕过目标，返回 dummy.next。

## 从题意一步步推导

1. 删除节点需要改它前驱的 next。例如删掉 4，要让 3 的 next 指向 5。
2. 从目标的前驱到尾节点恰好有 n 步。让快指针先走 n 步，再一起走，就能保持这个距离。快指针到尾节点时，慢指针便到了所需的前驱。
3. 头节点也可能被删。把 dummy 接在 head 前面，让两个指针都从 dummy 出发。若 n 等于链表长度，快指针先走 n 步后刚好到尾节点，慢指针留在 dummy。
4. 执行 slow.next = slow.next.next 绕过目标，返回 dummy.next。删头时它会指向新的头节点。

## 手推例子

链表为 1→2→3→4→5，n = 2。

| 阶段 | fast | slow |
| --- | --- | --- |
| 初始 | dummy | dummy |
| fast 先走两步 | 2 | dummy |
| 同步走一步 | 3 | 1 |
| 再走一步 | 4 | 2 |
| 停在尾节点 | 5 | 3 |

将 3 的 next 指向 5，得到 1→2→3→5。

## Java 代码

~~~java
class Solution {
    public ListNode removeNthFromEnd(ListNode head, int n) {
        ListNode dummy = new ListNode();
        dummy.next = head;
        ListNode fast = dummy;
        ListNode slow = dummy;

        for (int i = 0; i < n; i++) {
            fast = fast.next;
        }

        while (fast.next != null) {
            fast = fast.next;
            slow = slow.next;
        }

        slow.next = slow.next.next;
        return dummy.next;
    }
}
~~~

## 易错点

- 两个指针从同一位置同时出发会一直重合。先让 fast 走 n 步。
- 这套写法的停止点是尾节点。用 fast != null 会多走一步，使 slow 到达目标节点。
- 删头时 head 仍指向原头节点，返回 dummy.next 才能得到新头节点。
- 临时引用置空不改变链表结构。实际删除由 slow.next 的重新连接完成。

## 复杂度与自测

链表长度为 L，时间 O(L)，额外空间 O(1)。

1. 为什么目标的前驱距尾节点恰好 n 步？
2. 只有一个节点且 n = 1 时，循环在哪一步结束，最终返回什么？
3. n = 1 且链表有多个节点时，slow 最终指向哪里？

静态审阅并手推普通删除、删头、删尾及单节点情形。
