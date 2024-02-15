---
title: 二叉树的层序遍历-java引用问题解决
abbrlink: 8
categories: 学习笔记
tags: 算法
updated: 2023-02-13
data: 2023-02-13
---



### 题目

[102. 二叉树的层序遍历](https://leetcode.cn/problems/binary-tree-level-order-traversal/)

给你二叉树的根节点 `root` ，返回其节点值的 **层序遍历** 。 （即逐层地，从左到右访问所有节点）。

**示例 1：**

![img](https://assets.leetcode.com/uploads/2021/02/19/tree1.jpg)

```
输入：root = [3,9,20,null,null,15,7]
输出：[[3],[9,20],[15,7]]
```

**示例 2：**

```
输入：root = [1]
输出：[[1]]
```

**示例 3：**

```
输入：root = []
输出：[]
```

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode() {}
 *     TreeNode(int val) { this.val = val; }
 *     TreeNode(int val, TreeNode left, TreeNode right) {
 *         this.val = val;
 *         this.left = left;
 *         this.right = right;
 *     }
 * }
 */
```

### 原本思路

```java
class Solution {
    public List<List<Integer>> levelOrder(TreeNode root) {
        Queue<TreeNode> q = new ArrayDeque<>();
        if (root == null) {
            return new ArrayList<>();
        }
        List<List<Integer>> list = new ArrayList<>();
        List<Integer> list1 = new ArrayList<>();
        q.add(root);
        while (!q.isEmpty()) {
            int n = q.size();
            list1.clear();
            for (int i = 0; i < n; i++) {
                TreeNode treeNode = q.poll();
                list1.add(treeNode.val);
                if (treeNode.left != null) {
                    q.add(treeNode.left);
                }
                if (treeNode.right != null) {
                    q.add(treeNode.right);
                }
            }
            list.add(list1);
        }
        System.out.println(list);
        return list;
    }
}
```

输入

> root = [3,9,20,null,null,15,7]

标准输出

>  [[15, 7], [15, 7], [15, 7]]

输出

>  [[15,7],[15,7],[15,7]]

预期结果

>  [[3],[9,20],[15,7]]



可以看到这里的输出每一个元素都变成了最后一个元素。

### 原因分析

原因就出在 `list1.clear();`这行代码上。

这里的clear只会清空list1中的数据，不会创建一个新的对象。由于每次迭代添加到list中的都是list1对象，

所以在清空list1后，向list中添加的仍然是同一个list1对象的引用，这就导致list1变化的时候list中的其他元素也都跟着list1变化。

### 解决方案

在 `list1.clear();`处重新创建一个list1对象

### 完整代码

```java
class Solution {
    public List<List<Integer>> levelOrder(TreeNode root) {
        Queue<TreeNode> q = new ArrayDeque<>();
        if (root == null) {
            return new ArrayList<>();
        }
        List<List<Integer>> list = new ArrayList<>();
        q.add(root);
        while (!q.isEmpty()) {
            int n = q.size();
            List<Integer> list1 = new ArrayList<>();
            for (int i = 0; i < n; i++) {
                TreeNode treeNode = q.poll();
                list1.add(treeNode.val);
                if (treeNode.left != null) {
                    q.add(treeNode.left);
                }
                if (treeNode.right != null) {
                    q.add(treeNode.right);
                }
            }
            list.add(list1);
        }
        System.out.println(list);
        return list;
    }
}
```



### 附录

另外附上通过数组构建数的完整代码

```java
class Tests {
    public static void main(String[] args) {
        Integer[] nums = {3, 9, 20, null, null, 15, 7};
        TreeNode root = buildTree(nums, 0);
        levelOrder(root);
    }

    private static List<List<Integer>> levelOrder(TreeNode root) {
        List<List<Integer>> list = new ArrayList<>();
        Queue<TreeNode> q = new ArrayDeque<>();
        if (root == null) {
            return list;
        }
        q.add(root);
        List<Integer> list1 = new ArrayList<>();

        while (!q.isEmpty()) {
            int n = q.size();
            list1.clear();
            for (int i = 0; i < n; i++) {
                TreeNode treeNode = q.poll();
                list1.add(treeNode.val);
                if (treeNode.left != null) {
                    q.add(treeNode.left);
                }
                if (treeNode.right != null) {
                    q.add(treeNode.right);
                }
            }
            System.out.println(list1);
            list.add(list1);
        }
        System.out.println(list);
        return list;
    }

    private static TreeNode buildTree(Integer[] nums, int index) {
        if (index >= nums.length || nums[index] == null) {
            return null;
        }
        TreeNode root = new TreeNode(nums[index]);
        root.left = buildTree(nums, 2 * index + 1);
        root.right = buildTree(nums, 2 * index + 2);
        return root;

    }
}


class TreeNode {
    int val;
    TreeNode left;
    TreeNode right;

    TreeNode() {
    }

    TreeNode(int val) {
        this.val = val;
    }

    TreeNode(int val, TreeNode left, TreeNode right) {
        this.val = val;
        this.left = left;
        this.right = right;
    }
}
```



