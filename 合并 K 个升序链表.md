# 23. 合并 K 个升序链表

[题目](https://leetcode.cn/problems/merge-k-sorted-lists/) · Java

## 30 秒速记

沿用第 21 题的双链表归并。步长从 1 开始翻倍，每轮配对 i 和 i+step，结果存回 i。下一个左下标增加 2×step。没有搭档的末组留到下一轮。

## 推导与手推

每条输入链表已经有序，可以直接两两合并。第一轮每个头代表一条输入链表，第二轮每个有效头代表最多两条，后续每轮把可代表的链表数翻倍。这个过程和第 148 题迭代归并的后半段相同，这里通过数组下标访问各段的头。

以五条链表 A、B、C、D、E 为例。step=1 时合并下标 (0,1) 与 (2,3)，保留 E。step=2 时合并 (0,2)，E 仍保留在下标 4。step=4 时合并 (0,4)，所有节点的合并结果都在下标 0。

## Java

~~~java
class Solution {
    public ListNode mergeKLists(ListNode[] lists) {
        if (lists.length == 0) {
            return null;
        }
        for (int step = 1; step < lists.length; step *= 2) {
            for (int i = 0; i + step < lists.length; i += 2 * step) {
                lists[i] = mergeTwoLists(lists[i], lists[i + step]);
            }
        }
        return lists[0];
    }

    private ListNode mergeTwoLists(ListNode list1, ListNode list2) {
        ListNode dummy = new ListNode(0);
        ListNode tail = dummy;
        while (list1 != null && list2 != null) {
            if (list1.val < list2.val) {
                tail.next = list1;
                list1 = list1.next;
            } else {
                tail.next = list2;
                list2 = list2.next;
            }
            tail = tail.next;
        }
        tail.next = list1 != null ? list1 : list2;
        return dummy.next;
    }
}
~~~

## 易错点

- i 每次增加 2×step，跳过刚处理的两组。step=2 时，下一对从 4 开始。
- 判断 i+step 是否越界，末组没有搭档时自然跳过，进入下一轮继续参与合并。
- 空数组先返回 null。数组里的空链表可以交给 mergeTwoLists，两个头都为空时也能返回 null。
- 代码会改写 lists 中部分头引用，并重连输入节点。相等值时选择任意一边都能保证升序。

## 复杂度与自测

设 K 为链表条数，N 为节点总数。K≥2 时，最多有 O(log K) 轮，每轮最多遍历 N 个节点，另有总共 K−1 次配对调用，时间上界为 O(K + N log K)。额外空间 O(1)，每次归并的哑节点只在当次调用使用。

1. 六条链表在 step=2 时，哪些下标被合并？哪个下标保留？
2. 数组为 [null, null] 时返回什么？

已静态检查并手推空数组、空链表和奇数条链表的情况。

## 方法二 最小堆

每条升序链表中，当前头是该链表剩余节点的最小值。用最小堆维护这些候选。比如三条链是 1→2→5、3→6、4，取出 1 后补入 2，下一次就能取到 2。

~~~java
import java.util.Comparator;
import java.util.PriorityQueue;

class Solution {
    public ListNode mergeKLists(ListNode[] lists) {
        PriorityQueue<ListNode> heap =
                new PriorityQueue<>(Comparator.comparingInt(node -> node.val));
        for (ListNode head : lists) {
            if (head != null) {
                heap.offer(head);
            }
        }

        ListNode dummy = new ListNode(0);
        ListNode tail = dummy;
        while (!heap.isEmpty()) {
            ListNode node = heap.poll();
            if (node.next != null) {
                heap.offer(node.next);
            }
            tail.next = node;
            tail = node;
        }
        return dummy.next;
    }
}
~~~

空头不能入堆。每次只补入弹出节点的后继，因为其余链表的候选仍在堆中。设 K 为链表条数、M 为非空链表条数、N 为总节点数，时间 O(K + N log(M+1))，额外空间 O(M+1)。

## 方法三 递归分治

沿用方法一的 mergeTwoLists。先按下标将数组范围分成两半，分别合并，再把两份结果归并。例如 [0,4] 拆成 [0,2] 和 [3,4]，其中 [0,2] 继续拆成 [0,1] 和 [2,2]。范围只剩一条链表时直接返回它。

用下面两个方法替换方法一的 mergeKLists，保留原来的 mergeTwoLists。

~~~java
public ListNode mergeKLists(ListNode[] lists) {
    return mergeRange(lists, 0, lists.length - 1);
}

private ListNode mergeRange(ListNode[] lists, int left, int right) {
    if (left > right) {
        return null;
    }
    if (left == right) {
        return lists[left];
    }

    int mid = left + (right - left) / 2;
    ListNode first = mergeRange(lists, left, mid);
    ListNode second = mergeRange(lists, mid + 1, right);
    return mergeTwoLists(first, second);
}
~~~

空数组对应 [0,-1]。右半从 mid+1 开始，两个子区间才会缩小且互不重叠。K≥2 时，时间上界 O(K + N log K)，递归栈占 O(log K)。原来的步长翻倍法从单条链表开始逐轮归并，额外空间为 O(1)。

自测：堆为什么每条链表只保留一个节点？分治时 [0,1] 会拆成哪两个区间？

两种 Java 写法均已静态检查并手推空输入和奇数条链表。 [灵茶山艾府题解](https://leetcode.cn/problems/merge-k-sorted-lists/solutions/2384305/liang-chong-fang-fa-zui-xiao-dui-fen-zhi-zbzx/)
