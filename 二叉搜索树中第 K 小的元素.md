# 230. 二叉搜索树中第 K 小的元素

[力扣题目](https://leetcode.cn/problems/kth-smallest-element-in-a-bst/) · Java

## 30 秒速记

BST 的中序访问顺序是「左子树 → 当前节点 → 右子树」，节点值按升序出现。因此只在中序访问当前节点时更新排名。可以从 `count = 0` 往上数至 `k`，也可以从 `remaining = k` 往下数至 `0`。两种方法的计数位置相同。

## 手推计数位置

```text
      3
     / \
    1   4
     \
      2
```

以 `k = 2` 为例。先进入根的递归，把根的中序访问留到左子树返回之后。节点 `1` 等空的左子树返回才被数到，剩余次数变成 `1`。随后进入 `2`，等它的空左子树返回才把剩余次数减到 `0`，这时答案是 `2`。

## 写法一：用字段保存答案

`remaining` 由所有递归调用共用，`answer` 保存命中的值。左递归结束后先看剩余次数。若答案已在左侧找到，这一层立即退出。否则才轮到当前节点执行 `--remaining`。减到 `0` 就保存当前值并返回。[灵茶山艾府的题解](https://leetcode.cn/problems/kth-smallest-element-in-a-bst/solutions/2952810/zhong-xu-bian-li-pythonjavaccgojsrust-by-wc02/)

```java
class Solution {
    private int remaining;
    private int answer;

    public int kthSmallest(TreeNode root, int k) {
        remaining = k;
        recordInorder(root);
        return answer;
    }

    private void recordInorder(TreeNode node) {
        if (node == null) {
            return;
        }

        recordInorder(node.left);
        if (remaining == 0) {
            return;
        }

        if (--remaining == 0) {
            answer = node.val;
            return;
        }
        recordInorder(node.right);
    }
}
```

原题解在递归入口用 `k <= 0` 剪枝。已经进入的祖先调用从左子树返回后仍可能执行一次 `--k`，所以原写法的计数可以降到负数。这里把停止检查放在左递归返回之后，并在命中时立即返回，因此剩余次数停在 `0`，便于观察计数过程。[原题解的写法一](https://leetcode.cn/problems/kth-smallest-element-in-a-bst/solutions/2952810/zhong-xu-bian-li-pythonjavaccgojsrust-by-wc02/)

## 写法二：用递归返回值传回答案

每次调用返回两种结果之一：找到的节点值，或表示这次调用没有找到的 `-1`。题目保证节点值非负，因此 `0` 也是可能的答案，可以安全用 `-1` 表示未命中。左子树返回有效答案时，直接把答案向上传。左侧返回 `-1` 时，再数当前节点，然后视情况进入右子树。[原题解的写法二](https://leetcode.cn/problems/kth-smallest-element-in-a-bst/solutions/2952810/zhong-xu-bian-li-pythonjavaccgojsrust-by-wc02/)

```java
class Solution {
    private int remaining;

    public int kthSmallest(TreeNode root, int k) {
        remaining = k;
        return findInorder(root);
    }

    private int findInorder(TreeNode node) {
        if (node == null) {
            return -1;
        }

        int leftResult = findInorder(node.left);
        if (leftResult != -1) {
            return leftResult;
        }

        if (--remaining == 0) {
            return node.val;
        }
        return findInorder(node.right);
    }
}
```

样例中，节点 `2` 命中后返回 `2`。节点 `1` 的最后一行是 `return findInorder(node.right)`，直接把 `2` 传回。根 `3` 收到的 `leftResult` 是 `2`，于是直接返回。整个过程跳过了对 `3` 和 `4` 的计数。

## 复杂度与自测

两种写法的最坏时间都是 **O(n)**，递归栈空间都是 **O(h)**，其中 `h` 为树高。

1. 如果把 `--remaining` 放在左递归之前，样例中哪个值会被最先数到？
2. 若样例取 `k = 3`，根节点 `3` 的左子树会返回 `-1`。此时 `remaining` 还剩多少？
3. 如果要找第 `k` 大，应按什么顺序访问左右子树？

[常用代码：二叉树遍历](算法题总结/08%20二叉树遍历.md)
