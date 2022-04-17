---
layout: post
title: "ACW02-归并排序01：787. 归并排序"
subtitle: "ACW Practice 02: mergeSort"
author: "qingshan"
header-img: "img/post-bg-css.jpg"
header-mask: 0.4
tags:
  - 算法
  - ACW
---

# ACW 归并查询01： 787. 归并排序

给定你一个长度为 n 的整数数列。

请你使用归并排序对这个数列按照从小到大进行排序。

并将排好序的数列按顺序输出。

输入格式
输入共两行，第一行包含整数 n。

第二行包含 n 个整数（所有整数均在 1∼109 范围内），表示整个数列。

输出格式
输出共一行，包含 n 个整数，表示排好序的数列。

数据范围
1≤n≤100000
```
输入样例：
5
3 1 2 4 5
输出样例：
1 2 3 4 5
```

## 答

### template 1
```python
def mergeSort(nums):
    if len(nums) <=1: return
    
    mid = len(nums) >>1
    # 把 nums 从中间劈成两半
    L = nums[:mid]
    R = nums[mid:]

    # 继续劈
    mergeSort(L)
    mergeSort(R)

    i=j=k=0
    while i<len(L) and j<len(R):
        # 对比 L[i], R[j], 把小的放到 nums[k]位置上
        if L[i] < R[j]:
            nums[k] = L[i]
            i+=1
        else:
            nums[k] = R[j]
            j+=1
        k+=1

    # 运行到这里说明 L 和 R 其中有一个没有遍历完
    # 但是也不用判断了，直接把没遍历完的接到 nums 后面
    while i<len(L):
        nums[k] = L[i]
        i+=1
        k+=1

    while j<len(R):
        nums[k] = R[j]
        j+=1
        k+=1

def main():
    nums = [10,0,6,4,3,2,9,7,8,1]
    mergeSort(nums)
    print(nums)


if __name__ == "__main__":
    main()

```

----------

# 例题：ACW 归并查询02：788. 逆序对的数量

给定一个长度为 n 的整数数列，请你计算数列中的逆序对的数量。

逆序对的定义如下：对于数列的第 i 个和第 j 个元素，如果满足 i<j 且 a[i]>a[j]，则其为一个逆序对；否则不是。

输入格式
第一行包含整数 n，表示数列的长度。

第二行包含 n 个整数，表示整个数列。

输出格式
输出一个整数，表示逆序对的个数。

数据范围
1≤n≤100000，
数列中的元素的取值范围 [1,109]。
```
输入样例：
6
2 3 4 5 6 1
输出样例：
5
```

## 答

```python
def mergeSort(nums):
    if len(nums) <=1: return 0
    
    mid = len(nums) >>1
    # 把 nums 从中间劈成两半
    L = nums[:mid]
    R = nums[mid:]

    # 继续劈
    ans = mergeSort(L) + mergeSort(R)

    # 归并的过程
    i = j = k = 0
    while i < len(L) and j < len(R):
        # 如果 L[i] 小于等于R[j]，表示有序，不用换位
        # 如果有重复的数字，那么等号不能省
        if L[i] <= R[j]:
            nums[k] = L[i]
            k += 1
            i += 1
        else:
        # 否则表示需要更换数字位置
            nums[k] = R[j]
            # len(L) - i 表示本次交换的数字次数，也即是逆序对
            ans += len(L) - i
            k += 1
            j += 1
    while i < len(L):
        nums[k] = L[i]
        k += 1
        i += 1
    while j < len(R):
        nums[k] = R[j]
        k += 1
        j += 1

    # 这里 ans 是交换过的次数，也即是题意中逆序对的数量
    return ans


def main():
    nums = [88,71,16,2,72,38,94,50,72,67]
    # nums = [10,0,6,4,3,2,9,7,8,1,11]
    print(mergeSort(nums))
    print(nums)


if __name__ == "__main__":
    main()


```