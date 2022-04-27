---
layout: post
title: "ACW04-高精度运算：加减乘除"
subtitle: "ACW Practice 04: The High Precision Compution"
author: "qingshan"
header-img: "img/post-bg-css.jpg"
header-mask: 0.4
tags:
  - 算法
  - ACW
---

# ACW 高精度01：791. 高精度加法 

给定两个正整数（不含前导 0），计算它们的和。

输入格式
共两行，每行包含一个整数。

输出格式
共一行，包含所求的和。

数据范围
1≤整数长度≤100000

```
输入样例：
12
23
输出样例：
35
```

## 整体思路
将 AB 数列倒置，从第0位开始依次运算加法。超过 10 就进 1 存在 t 中。依次运算后结果存放 C，再倒置输出

## 答：

```python
def compare(A, B):
    """获取 A 或 B 中长的那个"""
    if len(A) != len(B):
        return len(A) > len(B)
    else:
        return True


def add(A, B):
    C=[]
    i=t=0
    while i<len(A):
        t += A[i]

        if i<len(B):
            t+=B[i]

        # 计算当位上面的数字
        C.append((t+10) % 10)

        # 判断是否进位
        if t>=10:
            t=1
        else:
            t=0

        i+=1

    # 最后一位，判断是否进 1
    if t==1:
        C.append(1)

    return C[::-1]


def main():
    a = [3,2,1,2]
    b = [1,2,3,8]

    A = a[::-1]
    B = b[::-1]

    if compare(A, B):
        return "".join(map(str, add(A,B)))
    else:
        return "".join(map(str, add(B,A)))

main()
```


----------
# ACW 高精度02：792. 高精度减法

给定两个正整数（不含前导 0），计算它们的差，计算结果可能为负数。

输入格式
共两行，每行包含一个整数。

输出格式
共一行，包含所求的差。

数据范围
1≤整数长度≤105
```
输入样例：
32
11
输出样例：
21
```

## 整体思路
将 AB 数列倒置，从第0位开始依次运算减法。超过 10 就进 1 存在 t 中。依次运算后结果存放 C，再倒置输出

## 答：

```python
def compare(A, B):
    """取 A，B 中长度长的哪一个"""
    if len(A) != len(B):
        return len(A) > len(B)

    # 倒序
    for i in range(len(A) - 1, -1, -1):
        if A[i] != B[i]:
            return A[i] > B[i]

    return True


def sub(A, B):
    C = []
    i=t=0
    while i<len(A):
        # t 是上次计算的借位
        t = A[i] - t
        if i < len(B):
            # t 是计算借位后，和减数相减的结果
            t -= B[i]
        # 留下当位的值
        C.append((t+10) % 10)
        if t<0:
            t = 1  # 借位
        else:
            t = 0  # 不借位
        i += 1
    # C 倒序重排
    C = C[::-1]
    while len(C) > 1 and C[0] == 0:
        # 如果结果中最高位是 0， 就弹出最高位的 0
        C.pop(0)

    return C

def main():
    a = [3,2,1,2]
    b = [1,2,3,8]

    A = a[::-1]
    B = b[::-1]

    if compare(A, B):
        return "".join(map(str, sub(A,B)))
    else:
        return "".join(map(str, sub(B,A)))

main()    

```

----------------
# ACW 高精度03：793. 高精度乘法

给定两个非负整数（不含前导 0） A 和 B，请你计算 A×B 的值。

输入格式
共两行，第一行包含整数 A，第二行包含整数 B。

输出格式
共一行，包含 A×B 的值。

数据范围
1≤A的长度≤100000,
0≤B≤10000
```
输入样例：
2
3
输出样例：
6

```

## 整体思路
将 A 数列倒置，从第0位开始依次乘以 b。超过 10 就进 1 存在 t 中。依次运算后结果存放 C，再倒置输出


## 答：
```python
def multiplication(A, b):
    if b == 0: return [0]

    C = []
    i=t=0
    while i<len(A):
        # A 上每一位都乘以 b
        t += A[i] * b
        # C 结果数列存入个位数
        C.append(t%10)
        # t 除以 10 取整数
        t = t // 10
        i+=1

    # 最高位如果不为 0 则存入
    if t!=0:
        C.append(t)
    
    # while len(C) > 1 and C[-1] == 0:
    #     C.pop()
    return C[::-1]

def main():
    a = [3,2,1,2]
    b = 12

    A = a[::-1]

    print("".join(map(str, multiplication(A,b))))

main()

```

----------------

# ACW高精度04：794. 高精度除法 

给定两个非负整数（不含前导 0） A，B，请你计算 A/B 的商和余数。

输入格式
共两行，第一行包含整数 A，第二行包含整数 B。

输出格式
共两行，第一行输出所求的商，第二行输出所求余数。

数据范围
1≤A的长度≤100000,
1≤B≤10000,
B 一定不为 0
```
输入样例：
7
2
输出样例：
3
1
```

## 答

## 整体思路
(类似高精度减法)

```python
def div(A, b):
    """返回结果[商， 余数]"""
    if b==0: return "ERROR"
    if b==1: return  list(A[::-1], 0)

    C = []
    i=t=0

    for i in range(len(A)-1, -1, -1):
        t = t*10 + A[i]
        # C 存入结果
        C.append(t//b)
        # t 存储余数
        t = t%b

    # 如果结果最高位是0，就弹出
    while len(C) >1 and C[0] == 0:
        C.pop(0)

    return [C, t]


def main():
    a = [3,2,1,2]
    b = 12

    A = a[::-1]

    result = div(A, b)
    print("".join(map(str, result[0])))
    print(result[1])

main()
```