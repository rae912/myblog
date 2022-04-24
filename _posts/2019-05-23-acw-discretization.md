---
layout: post
title: "ACW08-离散化"
subtitle: "The Discretization"
author: "qingshan"
header-img: "img/post-bg-os-metro.jpg"
header-mask: 0.4
tags:
  - 算法
  - ACW
---

# ACW离散化算法01: 802. 区间和

假定有一个无限长的数轴，数轴上每个坐标上的数都是 0。

现在，我们首先进行 n 次操作，每次操作将某一位置 x 上的数加 c。

接下来，进行 m 次询问，每个询问包含两个整数 l 和 r，你需要求出在区间 [l,r] 之间的所有数的和。

输入格式
第一行包含两个整数 n 和 m。

接下来 n 行，每行包含两个整数 x 和 c。

再接下来 m 行，每行包含两个整数 l 和 r。

输出格式
共 m 行，每行输出一个询问中所求的区间内数字和。

数据范围
−109≤x≤109,
1≤n,m≤105,
−109≤l≤r≤109,
−10000≤c≤10000

```
输入样例：
3 3
1 2
3 6
7 5
1 3
4 6
7 8
输出样例：
8
0
5
```

# 整体思路
这个题涉及多个基础算法：
1. 排序
2. 数组去重（双指针）
3. 二分查找
4. 离散化
5. 前缀和

题目拿到后，对数据依次进行处理：
1. 初始化数轴，最大长度是 n+2*m; （原因见下面）
2. 将x 位置和查询位置 l, r 三个值全部放入索引数组index；
3. 将索引数组排序并去重；
4. 根据题意，将x位置坐标从索引数组中查出下标 y，放入离散数组 a;
5. 根据题意加上c: 表达式a[y+1] += c；** 注意要加 1 **
6. 生成离散数组 a 的前缀和数组 s；长度都为 n+2m;
7. 根据题意，找出 l, r 分别在索引数组的位置 l_index, r_index; 
8. 输出结果就为 s[r_index] - s[l_index-1] ** 注意要减 1 **


# 需要避免的坑
1. N=n+2*m 因为n操作每次包含一个坐标，m 操作每次包含两个坐标。极端情况下，这三个坐标全部都不一样。所以初始化数组时候，需要大于 n+2*m
2. find 操作会返回目标元素的坐标值。但是实际上放入离散数组 a 时候需要坐标+1，因为 a[1]才是第一个映射值；
3. s 同理，s[1]才是第一个前缀和的值；

```python
def quickSort(nums, l, r):
    if l >= r: return

    i = l - 1
    j = r + 1
    x = nums[l+r>>1]

    while i<j:
        while True:
            i += 1
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


def unique(nums):
    i=j=0

    while i<len(nums):
        if i==0 or nums[i] != nums[i-1]:
            nums[j] = nums[i]
            j+=1

        i+=1

    return nums[:j]


def find(nums, x):
    l = 0
    r = len(nums) - 1

    while l<r:
        mid = l+r>>1
        if nums[mid] >=x: r=mid
        else: l = mid+1

    return l

def main():
    n, m = list(map(int, input().split()))
    N = n + 2*m + 10

    index_list = []
    add_list = [[0,0] for i in range(N)]
    for i in range(n):
        x, c = list(map(int, input().split()))
        index_list.append(x)
        add_list[i][0] = x
        add_list[i][1] = c


    query_list = [[0,0] for i in range(N)]
    for i in range(m):
        l, r = list(map(int, input().split()))
        index_list.append(l)
        index_list.append(r)
        query_list[i][0] = l
        query_list[i][1] = r

    quickSort(index_list, 0, len(index_list) - 1)
    index_list = unique(index_list)

    a = [0 for i in range(N)]
    for x, c in add_list:
        # 注意需要加 1
        a[find(index_list, x) + 1] += c

    # 前缀和
    s = [0 for i in range(N)]
    for i in range(len(index_list)):
        s[i+1] = s[i] + a[i+1]

    for i in range(m):
        # 注意需要加 1
        l = find(index_list, query_list[i][0]) + 1
        r = find(index_list, query_list[i][1]) + 1

        print(s[r] - s[l-1])


if __name__ == '__main__':
    main()

```

# 总结
这个题目非常好，将之前几次学习的知识点全盘复习了一遍，并且应用到了实际。