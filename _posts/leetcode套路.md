---
title: leetcode套路
date: 2020-08-24 09:41:49
tags: 算法
categories: 面试
---



# 1. 刷题策略

## 思路清晰，目标明确



### 数据存储方式

数组和链表。

队列、栈：数组或链表实现，数组需要考虑扩容
散列：数组+链表
图：邻接表(链表)或邻接矩阵(二维数组)
树：数组实现(堆)，是一个完全二叉树；链表实现如 二叉搜索树、AVL树、红黑树、区间树、B树等

// 二叉搜索树：左子树全部节点 小于根，右子树全部节点大于根，并以此类推
// 满二叉树: 叶子节点在同一层
// 完全二叉树：除了最后一层，其他层节点必须是满的

### 数据结构基本操作

递归：

```c++
// C/C++
void recursion(int level, int param) { 
  // recursion terminator
  if (level > MAX_LEVEL) { 
    // process result 
    return ; 
  }

  // process current logic 
  process(level, param);

  // drill down 
  recursion(level + 1, param);

  // reverse the current level status if needed
}
```



数组遍历：

```c++
// 迭代
void traverse(int[] arr) {
    for (int i = 0; i < arr.length; i++) {
        // 迭代访问 arr[i]
    }
}
```



链表遍历：

```c++
/* 基本的单链表节点 */
class ListNode {
    int val;
    ListNode* next;
}

void traverse(ListNode* head) {
    for (ListNode* p = head; p != null; p = p->next) {
        // 迭代访问 p->val
    }
}

void traverse(ListNode* head) {
    // 递归访问 head.val
    traverse(head->next)
}
```



二叉树遍历：

```c++
// 前序：根-左-右 ， 中序：左-根-右 ， 后序：左-右-根，递归
```



N叉树遍历：

```

```



### 刷题指南

先刷二叉树

解题5～10分钟，不行看题解，刷5遍

## 动态规划



## DSF 深度优先



## BSF 广度优先

