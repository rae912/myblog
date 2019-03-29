---
layout: post
title: "LeetCode002-AddTwoNumbers"
subtitle: "LeetCode002-AddTwoNumbers"
author: "qingshan"
header-img: "img/post-bg-halting.jpg"
header-mask: 0.4
tags:
  - 工作
  - 算法
  - LeetCode
---

题目：
>给出两个非空链表，代表两个非负数的整数。链表中的数字以逆序排列的同时，它们每个节点包含一个单个数字。将两个数相加并以一个链表形式返回。
 要求假设两数不包含任何以0开始的数字，除非这个数字本身就是0。
>* 示例：
   输入： (2 -> 4 -> 3) + (5 -> 6 -> 4)
>* 输出： 7 -> 0 -> 8

解题思路：题目有点绕，其实意思就是，有两个数342和465，把它们相加得出807，再以逆序显示出来。但是整个过程是以逆序和链表的形式来完成的。
链表：一种数据结构，包括一个值value，和一个下一访问数据单元（同样的结构）的索引。像链条一样环环相扣。

```python
# coding=utf-8

"""
You are given two linked lists representing two non-negative numbers. The digits are stored in reverse order and each of their nodes contain a single digit. Add the two numbers and return it as a linked list.

Input: (2 -> 4 -> 3) + (5 -> 6 -> 4)
Output: 7 -> 0 -> 8
"""


class ListNode:  # 链表结构体
    def __init__(self, value=None, next=None):
        self.value = value
        self.next = next

    def __str__(self):
        return self.value


class Solution(object):
    def addTwoNumbers(self, l1, l2):
        num1 = ""
        num2 = ""
        while l1:
            num1 += str(l1.val)
            l1 = l1.next
        while l2:
            num2 += str(l2.val)
            l2 = l2.next
        add = str(int(num1[::-1])) + str(int(num2[::-1]))[::-1]  # [::-1]是翻转字符串的意思

        head = ListNode(add[0])  # 结果链表的起始位
        answer = head
        for i in range(1, len(add)):  # 生成结果链表
            node = ListNode(add[i])
            head.next = node
            head = head.next

        return answer
```