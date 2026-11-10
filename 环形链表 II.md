# 142. 环形链表 II

[题目](https://leetcode.cn/problems/linked-list-cycle-ii/) · Java · 2026-11-10

## 30 秒速记

先快慢指针找环内相遇点。相遇后一个指针从 head 出发，另一个从相遇点出发，都每轮走一步，在入口相遇。
无环返回 null。比较引用，保持节点 next 不变。

## 从题意一步步推导

### 1. 环入口与相遇点分别是什么

~~~text
A→B→C→D→E
    ↑   |
    └───┘
~~~

E.next=C，所以入口为 C。
HashSet 沿 A、B、C、D、E、C 的顺序记录节点，第一个重复出现的是 C，可直接返回。
快慢指针则不一定第一次就在入口相遇。

### 2. 手推第一阶段

两指针初始均在 A，先移动再比较：

| 轮数 | slow | fast |
| --- | --- | --- |
| 0 | A | A |
| 1 | B | C |
| 2 | C | E |
| 3 | D | D |

在 D 相遇，说明存在环，但 D 不是入口。

### 3. 从两处同步走如何找到入口

让 p 从 A 出发，slow 留在 D，每轮都走一步：

| 轮数 | p | slow |
| --- | --- | --- |
| 0 | A | D |
| 1 | B | E |
| 2 | C | C |

两者到入口的距离刚好相同。一般情况可能包含整圈，需要进一步证明。

### 4. 用路程关系解释

设头到入口距离为 a，入口沿 next 到相遇点距离为 b，环长为 L。
慢指针走了 s 步，快指针走了 2s 步。
同一时刻在环内同一位置，快比慢多走整数圈，所以 2s-s=qL，即 s=qL。

慢指针到相遇点可能已经绕了 k 圈，s=a+b+kL。
因此 a+b=(q-k)L，a+b 是 L 的整数倍。

从相遇点出发再走 a 步，按入口作为坐标原点，其位置为 (b+a)%L=0，正好到入口。
从 head 出发走 a 步也到入口，所以第二阶段会在那里相遇。

在走够 a 步前，p 还在环外，而 slow 始终在环内，不可能提前在其他节点相遇。
a=0 时，第一阶段的相遇点也是入口，第二阶段条件一开始就不成立，直接返回。

### 5. 为什么第二阶段不能按 p!=null 循环

你初稿写 while(p!=null)，但第二阶段已确定有环，p 进入环后会无限绕圈，永远不为 null。
需要判断目标状态是否出现，因此用 p!=slow，直到同一个节点时停止。
第一阶段按 fast 的边界判断，因为仍需兼顾无环情况；两个阶段的停止目的不同。

## Java 代码：你完成的版本

~~~java
public class Solution {
    public ListNode detectCycle(ListNode head) {
        ListNode fast = head;
        ListNode slow = head;
        ListNode p = head;
        while (fast != null && fast.next != null) {
            fast = fast.next.next;
            slow = slow.next;
            if (slow == fast) {
                while (p != slow) {
                    p = p.next;
                    slow = slow.next;
                }
                return p;
            }
        }
        return null;
    }
}
~~~

p 在方法开头定义也正确，因为第一阶段没有移动它，它始终指向 head。
也可以在 if(slow==fast) 内再初始化 p=head，令用途集中。
ListNode 由力扣提供，返回的是节点引用。

## 补充对照：HashSet

这是你前面已理解的重复访问思路的完整实现。

~~~java
class Solution {
    public ListNode detectCycle(ListNode head) {
        java.util.Set<ListNode> visited = new java.util.HashSet<>();
        ListNode current = head;
        while (current != null) {
            if (!visited.add(current)) {
                return current;
            }
            current = current.next;
        }
        return null;
    }
}
~~~

从头遍历时，第一次重新进入访问过的节点，恰好是环入口。
默认 ListNode 的相等语义按节点对象区分，不能改成记录 val。

## 易错点与复杂度

- pos 是评测描述，不作为方法参数，不能据此直接访问入口。
- 先移动快慢指针再比较，避免初始同头被误认为已经相遇。
- 相遇后第二阶段两者都走一步，不能继续让 fast 走两步。
- 第二阶段一定在确认相遇后执行。
- 第二阶段比较引用相同，不能比较值。
- head=null、单节点无环，第一阶段跳过，返回 null。
- 单节点自环，第一次移动后相遇，第二阶段直接返回 head。
- 不需要修改任何节点的 next。

第一阶段 O(n)，第二阶段走 a 步，合计时间 O(n)，额外空间 O(1)。
HashSet 平均时间 O(n)，额外空间 O(n)。

## 合上笔记后自测

1. 上例为什么入口为 C，而第一阶段在 D 相遇？
2. 为什么快慢路程差一定是整数圈？
3. 如何推出 a+b 是环长整数倍？
4. 为什么从相遇点再走 a 步就到入口？
5. 第二阶段为什么不可能在到达入口前相遇？
6. a=0 时为何不用移动？
7. 为什么第二阶段写 p!=null 会死循环？

## 学习与验证

2026-11-10，你推导环内相遇与同步找入口，完成完整实现。
重点纠正入口与相遇点的区别，以及第二阶段应用引用相遇作为停止条件。
本轮静态审阅并手推，尚无 Java 运行或力扣提交结果。
