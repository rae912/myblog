---
layout: post
title: "ACW05-前缀和与差分"
subtitle: "ACW Practice 05: The prefix and difference"
author: "qingshan"
header-img: "img/post-bg-os-metro.jpg"
header-mask: 0.4
tags:
  - 算法
  - ACW
---

# ACW 前缀和差分01： 795. 前缀和

输入一个长度为 n 的整数序列。

接下来再输入 m 个询问，每个询问输入一对 l,r。

对于每个询问，输出原序列中从第 l 个数到第 r 个数的和。

输入格式
第一行包含两个整数 n 和 m。

第二行包含 n 个整数，表示整数数列。

接下来 m 行，每行包含两个整数 l 和 r，表示一个询问的区间范围。

输出格式
共 m 行，每行输出一个询问的结果。

数据范围
1≤l≤r≤n,
1≤n,m≤100000,
−1000≤数列中元素的值≤1000
```
输入样例：
5 3
2 1 3 6 4
1 2
1 3
2 4
输出样例：
3
6
10
```
## 思路：
初始化一个前缀和数列，索引从 1 开始。数列中第一个位置是 s[i+1] = s[i] + a[i]，后面每个位置，都是前一个位置（实际就是前缀和）和当前原始数列 a 的和。后面题意要求输出一个区间的和，就直接从前缀和数列中查询 s[r] - s[l-1]的值就可以了。

## 答：
```python
def compute(a):
    s = [0] * 100010
    for i in range(len(a)):
        s[i+1] = s[i] + a[i]

    return s

def main():
    m, n = [5, 3]
    a = [2,1,3,6,4]

    s = compute(a)

    print(s[2] - s[1-1])
    print(s[3] - s[1-1])
    print(s[4] - s[2-1])

main()
```

------------------

# ACW 前缀和差分02： 796. 子矩阵的和

输入一个 n 行 m 列的整数矩阵，再输入 q 个询问，每个询问包含四个整数 x1,y1,x2,y2，表示一个子矩阵的左上角坐标和右下角坐标。

对于每个询问输出子矩阵中所有数的和。

输入格式
第一行包含三个整数 n，m，q。

接下来 n 行，每行包含 m 个整数，表示整数矩阵。

接下来 q 行，每行包含四个整数 x1,y1,x2,y2，表示一组询问。

输出格式
共 q 行，每行输出一个询问的结果。

数据范围
1≤n,m≤1000,
1≤q≤200000,
1≤x1≤x2≤n,
1≤y1≤y2≤m,
−1000≤矩阵内元素的值≤1000
```
输入样例：
3 4 3
1 7 2 4
3 6 2 8
2 1 2 3
1 1 2 2
2 1 3 4
1 3 3 4
输出样例：
17
27
21

```

## 思路
基本思路和前缀和一样，只是变成了二维数组，情况要复杂一些。

初始化前缀和为`s[i+1][j+1] = s[i][j+1] + s[i+1][j] - s[i][j] + a[i][j]`；

后面求值查询条件为`s[x2][y2] - s[x2][y1-1] - s[x1-1][y2] + s[x1-1][y1-1]`。

## 答
```python
def compute(n, m):
    a = [[0]*10010 for _ in range(1010)]
    s = [[0]*10010 for _ in range(1010)]

    for i in range(n):
        rows = list(map(int, input().split()))
        for j in range(m):
            a[i][j] = rows[j]

    for i in range(n):
        for j in range(m):
            s[i+1][j+1] = s[i][j+1] + s[i+1][j] - s[i][j] + a[i][j]
    
    return searchIndex


def main():
    n,m,q = list(map(int, input().split()))
    s = compute(n,m)

    while q!=0:
        x1,y1,x2,y2 = list(map(int, input().split()))
        print(s[x2][y2] - s[x2][y1-1] - s[x1-1][y2] + s[x1-1][y1-1])

main()
```

---------------

# ACW 前缀和差分03： 797. 差分

输入一个长度为 n 的整数序列。

接下来输入 m 个操作，每个操作包含三个整数 l,r,c，表示将序列中 [l,r] 之间的每个数加上 c。

请你输出进行完所有操作后的序列。

输入格式
第一行包含两个整数 n 和 m。

第二行包含 n 个整数，表示整数序列。

接下来 m 行，每行包含三个整数 l，r，c，表示一个操作。

输出格式
共一行，包含 n 个整数，表示最终序列。

数据范围
1≤n,m≤100000,
1≤l≤r≤n,
−1000≤c≤1000,
−1000≤整数序列中元素的值≤1000
```
输入样例：
6 3
1 2 2 1 2 1
1 3 1
3 5 1
1 6 1
输出样例：
3 4 5 3 4 2
```

## 思路
初始化一个差分数组 s, 用于存放差分数组。初始化为 s[l] +=c , s[r+1] -=c。这里初始化时候是 l=r的。所以初始化以后，除了 s[0] = a[i] 以外，其他的位置在下次计算时候会被抵消，还是为 a[i] - a[i+1] (差分)。后面每一次 insert 操作，也只变化对应 l,r+1 两个位置上的值。最后，s[i] += s[i-1]即可还原出题意要求的原始数组。

## 答
```python
def insert(nums, l ,r, c):
    # b 查分数组， l起始位置，r结束位置，c加数
    nums[l] += c
    nums[r+1] -= c


def main():
    n, m = map(int, input().split())
    a = [0] * (n+10) # 原值数组
    b = [0] * (n+10) # 差分数组

    nums = list(map(int, input().split()))
    index=0
    for v in nums:
        a[index+1] = v
        index+=1
    for i in range(1, n+1):
        insert(b, i, i, a[i]) # 构建差分数组

    while m>0:
        m-=1
        l,r,c = map(int, input().split())

        insert(b, l, r, c)  # 变更差分数组

    for i in range(1, n+1):
        b[i] += b[i-1] # 还原原始数组
        print(b[i], end = " ")

main()
        
```
------------------

# ACW 前缀和差分04： 798. 差分矩阵

输入一个 n 行 m 列的整数矩阵，再输入 q 个操作，每个操作包含五个整数 x1,y1,x2,y2,c，其中 (x1,y1) 和 (x2,y2) 表示一个子矩阵的左上角坐标和右下角坐标。

每个操作都要将选中的子矩阵中的每个元素的值加上 c。

请你将进行完所有操作后的矩阵输出。

输入格式
第一行包含整数 n,m,q。

接下来 n 行，每行包含 m 个整数，表示整数矩阵。

接下来 q 行，每行包含 5 个整数 x1,y1,x2,y2,c，表示一个操作。

输出格式
共 n 行，每行 m 个整数，表示所有操作进行完毕后的最终矩阵。

数据范围
1≤n,m≤1000,
1≤q≤100000,
1≤x1≤x2≤n,
1≤y1≤y2≤m,
−1000≤c≤1000,
−1000≤矩阵内元素的值≤1000
```
输入样例：
3 4 3
1 2 2 1
3 2 2 1
1 1 1 1
1 1 2 2 1
1 3 2 3 2
3 1 3 4 1
输出样例：
2 3 4 1
4 3 4 1
2 2 2 2
```

## 思路
思路参考差分数组。在二维数组情况下更为复杂一点。insert函数为
```python
b[x1][y1] +=c
b[x1][y2+1] -= c
b[x2+1][y1] -= c
b[x2+1][y2+1] += c
```
初始状态：`insert(b, i+1, j+1, i+1, j+1, a[i+1][j+1])`
最后还原：`b[i+1][j+1] = b[i][j+1] + b[i+1][j] + b[i+1][j+1] - b[i][j]`

## 答
```python
a = [[0] * 1010 for _ in range(1010)]
b = [[0] * 1010 for _ in range(1010)]

def insert(b, x1, y1, x2, y2, c):
    b[x1][y1] += c
    b[x1][y2+1] -= c
    b[x2+1][y1] -= c
    b[x2+1][y2+1] += c

def main():
    n,m,q = list(map(int, input().split()))
    for i in range(n):
        nums = list(map(int, input().split()))
        for j in range(m):
            a[i+1][j+1] = nums[j]
            insert(b, i+1, j+1, i+1, j+1, a[i+1][j+1])

    while q!=0:
        q-=1
        x1, y1, y1, y2, c = list(map(int, input().split()))
        insert(b, x1, y1, x2, y2, c)

    for i in range(n):
        for j in range(m):
            b[i+1][j+1] = b[i][j+1] + b[i+1][j] - b[i][j] + b[i+1][j+1]
            print(b[i+1][j+1], end = " ")
        print()

main()

```
