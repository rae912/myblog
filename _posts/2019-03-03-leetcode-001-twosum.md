---
layout: post
title: "算法练习001-TwoSum"
subtitle: "LeetCode001-TwoSum"
author: "qingshan"
header-img: "img/post-bg-halting.jpg"
header-mask: 0.4
tags:
  - 工作
  - 算法
  - LeetCode
---

题目：
>给出一个包含一系列整数的数组和一个目标整数值，要求找出能够使数组内元素之和为目标整数值的索引，返回一个包含两个索引的结果数组。
>* 示例：
   数组 nums = [2, 7, 11, 15], 目标值 9,
   因为 nums[0] + nums[1] = 2 + 7 = 9,
>* 返回 [0, 1].

解题思路：从结果入手。题意需要求a+b=c，而c是已知且固定的，那么就可以将题意转换为b=c-a。其中a为nums中依次遍历，遍历的次数就是a的索引，如果b存在nums中，那么b的索引就是另一个需要求的值。

优化点：有的语言没有nums.index的功能，为了节省计算开销，可以将每次c-a的值的索引存储进dict，方便后面查找

```python
# coding=utf-8

"""
Given an array of integers, return indices of the two numbers such that they add up to a specific target.
You may assume that each input would have exactly one solution, and you may not use the same element twice.
Example:
Given nums = [2, 7, 11, 15], target = 9,
Because nums[0] + nums[1] = 2 + 7 = 9,
return [0, 1].
"""

class Solution(object):
    def twoSum(self, nums, target):
        for index, value in enumerate(nums):
            b = target - value
            if b in nums:
                return [index, nums.index(b)]


if __name__ == "__main__":
    nums = [2, 7, 11, 15]
    target = 18
    s = Solution()
    print(s.twoSum(nums, target))
```

==END==