# 25. K 个一组翻转链表

[题目](https://leetcode.cn/problems/reverse-nodes-in-k-group/) · Java

## 30 秒速记

从组前驱数 k 步确认组尾。以组外第一个节点作为 prev，翻转到组边界就停。接回新组头，再把原组头用作下一组的前驱。

## 推导与手推

以 1→2→3→4→5→6→7、k=3 为例。

1. 第一组要翻成 3→2→1，因此组前的 dummy 要接到 3，原组头 1 要接到 4。后面的完整组用同样的连接规则。
2. 让 groupEnd 从 groupPrev 出发走 k 步。第一轮从 dummy 到 3。若走到 null，当前组不足 k 个，直接返回。查找阶段还未改动任何连接。
3. 确认组尾后，保存 groupNext=4 和 oldGroupHead=1。令 current=1、prev=4。每轮先保存 current 的原后继，再反转 next 并移动两个引用。
4. 处理完 1、2、3 后，已翻转部分为 3→2→1→4。current 到达 4 时停止，prev 指向 3。令 groupPrev.next 指向 3，再把 groupPrev 移到 1。
5. 第二轮从 1 数到 6，翻转得到 3→2→1→6→5→4→7。第三轮从 4 往后无法数够三个节点，返回当前结果。

## Java

~~~java
class Solution {
    public ListNode reverseKGroup(ListNode head, int k) {
        ListNode dummy = new ListNode(0, head);
        ListNode groupPrev = dummy;

        while (true) {
            ListNode groupEnd = groupPrev;
            for (int i = 0; i < k; i++) {
                groupEnd = groupEnd.next;
                if (groupEnd == null) {
                    return dummy.next;
                }
            }

            ListNode groupNext = groupEnd.next;
            ListNode oldGroupHead = groupPrev.next;
            ListNode current = oldGroupHead;
            ListNode prev = groupNext; // 让原组头在第一轮接到组外节点

            while (current != groupNext) {
                ListNode next = current.next;
                current.next = prev;
                prev = current;
                current = next;
            }

            groupPrev.next = prev;
            groupPrev = oldGroupHead;
        }
    }
}
~~~

## 易错点

- 找到组尾之后再取 groupNext。若提前读取，此时 groupEnd 仍是 groupPrev，groupNext 和 current 会从同一个节点开始。
- prev 初始化为 groupNext。若初始化为原组头，第一轮会让组头指向自身。
- 每轮既要执行 current.next = prev，也要推进两个引用。先保存 next 才能沿原顺序走完这一组。
- 整组翻完后才接回 groupPrev，并推进到 oldGroupHead。接 prev.next 会漏掉新组头。
- while (true) 通过组尾判空返回。若末组恰好完整，groupNext 可以为 null，下一轮查找时退出。

## 复杂度与自测

链表长度为 n。每个节点最多参与一次确认组长和一次反转，时间 O(n)，额外空间 O(1)。

1. 为什么从组前驱出发要走 k 步，而从组头出发只需 k-1 步？
2. 为什么 prev 从 groupNext 开始就能自动接好组尾？
3. 手推 k=1、k=n，以及末组不足 k 的情况。

已静态检查和手推，未在力扣提交。

## 补充写法：组内头插

确认够 k 个节点后，固定 first 指向原组头。每轮把 first 后面的节点摘下，插到 groupPrev 后面。以 1→2→3→4、k=3 为例，先把 2 移到组头得到 2→1→3→4，再把 3 移到组头得到 3→2→1→4。first 始终指向 1，完成 k-1 次头插后成为下一组的前驱。

~~~java
class Solution {
    public ListNode reverseKGroup(ListNode head, int k) {
        ListNode dummy = new ListNode(0, head);
        ListNode groupPrev = dummy;

        while (true) {
            ListNode end = groupPrev;
            for (int i = 0; i < k; i++) {
                end = end.next;
                if (end == null) {
                    return dummy.next;
                }
            }

            ListNode first = groupPrev.next;
            for (int i = 1; i < k; i++) {
                // 摘下 first 后面的节点，再插到组头。
                ListNode moved = first.next;
                first.next = moved.next;
                moved.next = groupPrev.next;
                groupPrev.next = moved;
            }
            groupPrev = first;
        }
    }
}
~~~

注意 first 在组内始终指向原组头。moved.next 接的是当前组头 groupPrev.next，每轮头插后组头都会变。找组尾只用于确认组长，头插次数固定为 k-1，因此无需另外保存组外节点。时间 O(n)，额外空间 O(1)。已静态检查并手推 k=1、完整末组和不足 k 的末组。
