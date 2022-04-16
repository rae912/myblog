---
layout: post
title: "ACW01-快速排序01：785. 快速排序"
subtitle: "ACW Practice 01: quickSort"
author: "qingshan"
header-img: "img/post-sample-image.jpg"
header-mask: 0.4
tags:
  - 算法
  - ACW
---

# ACW 快速排序01：785. 快速排序

给定你一个长度为 n 的整数数列。

请你使用快速排序对这个数列按照从小到大进行排序。

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
def quickSort(nums, l, r):
    if l>=r: return

    i, j = l, r
    # mid 取中间索引，等同于 (l+r) // 2, 向上取整
    mid = l+r>>1
    # 中值数
    x = nums[mid]

    while i<=j:
        # 左指针从左到右移到中间索引附近
        while nums[i] < x:
            i+=1

        # 右指针从右到左移到中间索引附近
        while nums[j] > x:
            j-=1

        # 如果左右指针不重叠，交换对应指针指向的位置
        if i<=j:
            nums[i], nums[j] = nums[j], nums[i]
            i+=1
            j-=1

    # 继续交换
    quickSort(nums, l, j)
    quickSort(nums, i, r)

def main():
    nums = [10,0,6,4,3,2,9,7,8,1]
    quickSort(nums, 0, len(nums) - 1)
    
    print(nums)

if __name__ == "__main__":
    main()

```

### template 2
```python
def quickSort(nums, l, r):
    if l>=r: return
    i = l-1
    j = r+1
    x = nums[l+r>>1]

    while i<j:
        while True:
            i+=1
            if nums[i] >=x: break
        
        while True:
            j-=1
            if nums[j] <=x: break
        
        if i<j:
            tmp = nums[i]
            nums[i] = nums[j]
            nums[j] = tmp
            
    quickSort(nums, l, j)
    quickSort(nums, j+1, r)
```

----------

# 例题：第k个数

给定一个长度为 n 的整数数列，以及一个整数 k，请用快速选择算法求出数列从小到大排序后的第 k 个数。

输入格式
第一行包含两个整数 n 和 k。

第二行包含 n 个整数（所有整数均在 1∼109 范围内），表示整数数列。

输出格式
输出一个整数，表示数列的第 k 小数。

数据范围
1≤n≤100000,
1≤k≤n
```
输入样例：
5 3
2 4 1 5 3
输出样例：
3
```

### 答：

```python
quickSort(nums)
print(nums[k-1])
```