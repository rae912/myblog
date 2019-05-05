---
layout: post
title: "算法练习003-LongestSubstringWithoutRepeatingCharacters"
subtitle: "LeetCode003-LongestSubstringWithoutRepeatingCharacters"
author: "qingshan"
header-img: "img/post-bg-halting.jpg"
header-mask: 0.4
tags:
  - 工作
  - 算法
  - LeetCode
---

题目：
>给定两个大小为 m 和 n 的有序数组 nums1 和 nums2 。
请找出这两个有序数组的中位数。要求算法的时间复杂度为 O(log (m+n)) 。
你可以假设 nums1 和 nums2 不同时为空。

示例：
>* nums1 = [1, 3], nums2 = [2], result = 2.0
>* nums1 = [1, 2], nums2 = [3, 4], result = 2.5

解题思路：利用vector，因为此次考虑的字符串中只包含字母，所以可以创建一个256大小，初始化为-1的vector，负责记录字符串中每个字符所在的坐标。
并初始化一个first为-1，记录最长无重复子串的起始坐标。在每次遍历字符串时，如果vector中记录的该元素的坐标大于first，说明之前子串已经用到过
相同的元素值，发现重复，则更新first为该元素坐标。然后将该元素坐标记录到vector中。最后更新一下count，根据当前遍历的坐标i-first与count比
较。

```python
# coding=utf-8

"""
Given a string, find the length of the longest substring without repeating characters.
Example:
Input: "abcabcbb"
Output: 3
Explanation: The answer is "abc", with the length of 3.

"""


class Solution(object):
    def lengthOfLongestSubstring(self, s):
        """
        :type s: str
        :rtype: int
        """
        if not s:
            return 0
        if len(s) <= 1:
            return len(s)
        locations = [-1 for i in range(256)]
        index = -1
        m = 0
        for i, v in enumerate(s):
            l = locations[ord(v)]
            if l > index:
                index = locations[ord(v)]
            m = max(m, i - index)
            locations[ord(v)] = i
        return m


class Solution2(object):
    def lengthOfLongestSubstring(self, s):
        a = {}
        count = 0
        index = -1
        for i in range(len(s)):
            if s[i] in a and a[s[i]] > index:
                index = a[s[i]]
            a[s[i]] = i
            count = max(count, i-index)
        return count


if __name__ == "__main__":
    print(Solution().lengthOfLongestSubstring("abceabcedfg"))
    print(Solution2().lengthOfLongestSubstring("abceabcedfg"))

```