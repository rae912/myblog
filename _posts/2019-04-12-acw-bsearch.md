---
layout: post
title: "ACW03-二分查找01：789. 数的范围"
subtitle: "ACW Practice 02: mergeSort"
author: "qingshan"
header-img: "img/post-bg-css.jpg"
header-mask: 0.4
tags:
  - 算法
  - ACW
---

# ACW 二分01：789. 数的范围

给定一个按照升序排列的长度为 n 的整数数组，以及 q 个查询。

对于每个查询，返回一个元素 k 的起始位置和终止位置（位置从 0 开始计数）。

如果数组中不存在该元素，则返回 -1 -1。

输入格式
第一行包含整数 n 和 q，表示数组长度和询问个数。

第二行包含 n 个整数（均在 1∼10000 范围内），表示完整数组。

接下来 q 行，每行包含一个整数 k，表示一个询问元素。

输出格式
共 q 行，每行包含两个整数，表示所求元素的起始位置和终止位置。

如果数组中不存在该元素，则返回 -1 -1。

数据范围
1≤n≤100000
1≤q≤10000
1≤k≤10000
```
输入样例：
6 3
1 2 2 3 3 4
3
4
5
输出样例：
3 4
5 5
-1 -1

```

## 答：
```python
def bsearch_first(nums, k):
    """二分查找第一次出现位置"""
    l, r = 0, len(nums) - 1

    while l<r:
        mid = l+r >>1
        if nums[mid] >= k:
            r = mid
        else:
            l = mid+1

    if nums[l] != k:
        return -1
    else:
        return l

def bsearch_last(nums, k):
    """二分查找最后出现位置"""
    l, r = 0, len(nums) - 1
    while l<r:
        mid = l+r+1 >>1
        if nums[mid] <=k:
            l = mid
        else:
            r = mid - 1

    if nums[l] != k:
        return -1
    else:
        return l

def main():
    nums = [1,2,2,3,3,4]
    print(bsearch_first(nums, 3))
    print(bsearch_last(nums, 3))

if __name__ == "__main__":
    main()
    
```

## 例题

# ACW 二分02：790. 数的三次方根

给定一个浮点数 n，求它的三次方根。

输入格式
共一行，包含一个浮点数 n。

输出格式
共一行，包含一个浮点数，表示问题的解。

注意，结果保留 6 位小数。

数据范围
−10000≤n≤10000
```
输入样例：
1000.00
输出样例：
10.000000
```

## 解题思路
在数据范围内，根据二分法，查找结果。停止条件为 `r-l >= 1e-8` 由题意定义的精度。判断条件为`mid*mid*mid >=x`。此题可以类推为求任意的 N 次方根。

```python

def bsearch(x):
    l, r = -10000, 10000

    while r - l >= 1e-8:
        mid = float((l+r) / 2)
        if mid * mid * mid >= x:
            r = mid
        else:
            l = mid

    return l


def main():
    n = float(input())
    # 保留六位小数
    print("{:.6f}".format(bsearch(n)))

if __name__ == "__main__":
    main()

```