# 146. LRU 缓存

[题目](https://leetcode.cn/problems/lru-cache/) · Java

## 30 秒速记

HashMap 通过键找到节点。双向链表从头到尾按最近到最久使用排列。读取命中、更新旧键和插入新键都会把节点放到头部。超容量时淘汰 tail.prev，并从哈希表移除它的键。

## 从题意一步步推导

1. 只用链表时，按键查找需要遍历。HashSet 只能验证键存在，HashMap 可以从键直接定位节点并读取 value。
2. 命中后要把节点移到头部。若节点只有 next，要从头寻找前驱。加上 prev 后，摘除只需把两侧重新接起来。
3. 用 head 和 tail 两个哑节点统一边界。初始化令 head.next 指向 tail，tail.prev 指向 head。空缓存第一次插入时，head.next 也有节点可以更新。
4. get 未命中返回 -1。命中则先摘除，再放到头部。
5. put 命中时复用已有节点，更新 value 并刷新位置。新键则新建节点，同时加入哈希表和链表。只有插入新键才可能超容量。
6. 链尾节点保存 key，淘汰时可直接执行 map.remove(lru.key)。这样链表中的真实节点与哈希表条目始终一一对应。

## 手推

容量为 2。表中顺序为最近使用到最久未使用。

| 操作 | 链表中的键 | 结果 |
| --- | --- | --- |
| put(1, 10) | 1 | 插入 1 |
| put(2, 20) | 2 → 1 | 插入 2 |
| put(1, 100) | 1 → 2 | 更新 1，条目数仍为 2 |
| put(3, 30) | 3 → 1 | 先插入 3，再淘汰 2 |
| get(1) | 1 → 3 | 返回 100 |
| get(2) | 1 → 3 | 返回 -1 |

若把第三步换成 get(1)，也会淘汰 2，只是键 1 的值仍为 10。

## Java

~~~java
import java.util.HashMap;
import java.util.Map;

class LRUCache {
    private static class Node {
        int key;
        int value;
        Node prev;
        Node next;

        Node(int key, int value) {
            this.key = key;
            this.value = value;
        }
    }

    private final int capacity;
    private final Map<Integer, Node> map = new HashMap<>();
    private final Node head = new Node(0, 0);
    private final Node tail = new Node(0, 0);

    public LRUCache(int capacity) {
        this.capacity = capacity;
        head.next = tail;
        tail.prev = head;
    }

    public int get(int key) {
        Node node = map.get(key);
        if (node == null) {
            return -1;
        }
        unlink(node);
        addFirst(node);
        return node.value;
    }

    public void put(int key, int value) {
        Node node = map.get(key);
        if (node == null) {
            node = new Node(key, value);
            map.put(key, node);
            addFirst(node);
            if (map.size() > capacity) {
                Node lru = tail.prev;
                unlink(lru);
                map.remove(lru.key);
            }
        } else {
            node.value = value;
            unlink(node);
            addFirst(node);
        }
    }

    private void unlink(Node node) {
        node.prev.next = node.next;
        node.next.prev = node.prev;
    }

    private void addFirst(Node node) {
        node.next = head.next;
        node.prev = head;
        head.next.prev = node;
        head.next = node;
    }
}
~~~

## 易错点

- Map 的值类型是 Node。新键先创建节点，再执行 map.put(key, node)。整数 value 已保存在节点内。
- 已有键要同时更新值和使用顺序。写 node.value = value 会修改 map 与链表共同引用的那个对象。
- addFirst 先给新节点保存旧头，再更新旧头的 prev，最后更新 head.next。提前覆盖 head.next 会丢失从 head 到旧头的连接。
- 淘汰要同时修改链表和哈希表。漏删哈希表会导致已淘汰的键还可被查到。

## 复杂度与自测

链表操作只改固定数量的指针，get 和 put 平均时间为 O(1)，缓存额外空间为 O(capacity)。

1. 为什么节点要同时保存 key 和 value？
2. 容量为 1，连续更新同一个键会触发淘汰吗？
3. 当只有一个真实节点时，unlink 会把哪两个节点重新连起来？

本地 Java 验证通过 8 项断言，覆盖未命中、读写刷新顺序、容量为 1 和淘汰。
