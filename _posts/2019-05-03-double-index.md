---
layout: post
title: "ACW06-双指针算法"
subtitle: "The double index"
author: "qingshan"
header-img: "img/post-bg-os-metro.jpg"
header-mask: 0.4
tags:
  - 算法
  - ACW
---

# ACW双指针算法01：799. 最长连续不重复子序列

给定一个长度为 n 的整数序列，请找出最长的不包含重复的数的连续区间，输出它的长度。

输入格式
第一行包含整数 n。

第二行包含 n 个整数（均在 0∼1e5 范围内），表示整数序列。

输出格式
共一行，包含一个整数，表示最长的不包含重复的数的连续区间的长度。

数据范围
1≤n≤1e5
```
输入样例：
5
1 2 2 3 5
输出样例：
3
```

# 思路：双指针
设置一个字典，key 为整数序列的值。如果对应的值为曾经出现的次数。如果 j<=i 且 d[a[j]]出现不止一次，说明有重复，j 指针后移，并将值减一（重置），直到 d[a[j]]为 1。说明符合题意：不重复区间。

![解法](https://cdn.acwing.com/media/article/image/2020/05/21/38626_0aed0a9e9b-1.png)

```python
def main():
    n = 5
    a = [1,2,2,3,5]

    i=j=res=0
    d = dict()

    while i<n:
        if d.get(a[i]) is None:
            d[a[i]] = 1
        else:
            d[a[i]] += 1
        
        while j<=i and d[a[j]] > 1:
            d[a[j]] -= 1
            j+=1

        i+=1
        res = max(res, i-j+1)
        
    print(res)
```

--------------------
# ACW双指针算法02：800. 数组元素的目标和

给定两个升序排序的有序数组 A 和 B，以及一个目标值 x。

数组下标从 0 开始。

请你求出满足 A[i]+B[j]=x 的数对 (i,j)。

数据保证有唯一解。

输入格式
第一行包含三个整数 n,m,x，分别表示 A 的长度，B 的长度以及目标值 x。

第二行包含 n 个整数，表示数组 A。

第三行包含 m 个整数，表示数组 B。

输出格式
共一行，包含两个整数 i 和 j。

数据范围
数组长度不超过 1e5。
同一数组内元素各不相同。
1≤数组元素≤1e9
```
输入样例：
4 5 6
1 2 4 7
3 4 6 8 9
输出样例：
1 1
```

# 思路：双指针

因为AB是单调递增数列， 所以 AB 的解位置一定有最大值。那么从 A 的第一个数，B 的最后一个数开始。如果`j>0 and A[i] + B[j] >x`，则 j往前移动一位，直到`A[i] + B[j]<=x`，然后把 A 的位置往后移动。直到`A[i] + B[j]==x`。

```python
def main():
    n,m,x = list(map(int, input().split()))
    A = list(map(int, input().split()))
    B = list(map(int, input().split()))

    i=0 # i 指向 n 数组的第一个元素
    j=m-1 # j 指向m 数组的最后一个元素

    while i<n:
        while A[i] + B[j] > x and j>0:
            j-=1 # 从 m 数组最后一个元素往前找

        if A[i] + B[j] == x:
            print(i, j)
            return # 找到后退出程序

        i+=1 # 从 A 数组第一个元素开始

if __name__ == "__main__":
    main()

```

---------------

# ACW双指针算法03: 2816. 判断子序列

给定一个长度为 n 的整数序列 a1,a2,…,an 以及一个长度为 m 的整数序列 b1,b2,…,bm。

请你判断 a 序列是否为 b 序列的子序列。

子序列指序列的一部分项按原有次序排列而得的序列，例如序列 {a1,a3,a5} 是序列 {a1,a2,a3,a4,a5} 的一个子序列。

输入格式
第一行包含两个整数 n,m。

第二行包含 n 个整数，表示 a1,a2,…,an。

第三行包含 m 个整数，表示 b1,b2,…,bm。

输出格式
如果 a 序列是 b 序列的子序列，输出一行 Yes。

否则，输出 No。

数据范围
1≤n≤m≤105,
−1e9≤ai,bi≤1e9
```
输入样例：
3 5
1 3 5
1 2 3 4 5
输出样例：
Yes
```

# 思路：双指针
a 序列是短的序列，它的指针仅当元素在 b 中找到时候，才往右移；b 是长的序列，它的指针每次都后移。停止条件为其中一个指针移到尾。如果 a 的指针等于 a 的长度，说明 a 是 b 的子序列，否则不是。

```python
def main():
    n, m = list(map(int, input().split()))
    
    A = list(map(int, input().split()))
    B = list(map(int, input().split()))
    
    i=j=0
    while i<n and j<m:
        if A[i] == B[j]:
            i+=1
        j+=1
        
    if i == n: print("Yes")
    else:print("No")
    
if __name__ == "__main__":
    main() 

```


