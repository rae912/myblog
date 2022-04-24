---
layout: post
title: "ACW11 双链表：827. 双链表"
subtitle: "The Single Linked List"
author: "qingshan"
header-img: "img/home-bg-o.jpg"
header-mask: 0.4
tags:
  - 算法
  - ACW
---


827. 双链表

实现一个双链表，双链表初始为空，支持 5 种操作：

在最左侧插入一个数；
在最右侧插入一个数；
将第 k 个插入的数删除；
在第 k 个插入的数左侧插入一个数；
在第 k 个插入的数右侧插入一个数
现在要对该链表进行 M 次操作，进行完所有操作后，从左到右输出整个链表。

注意:题目中第 k 个插入的数并不是指当前链表的第 k 个数。例如操作过程中一共插入了 n 个数，则按照插入的时间顺序，这 n 个数依次为：第 1 个插入的数，第 2 个插入的数，…第 n 个插入的数。

输入格式
第一行包含整数 M，表示操作次数。

接下来 M 行，每行包含一个操作命令，操作命令可能为以下几种：

L x，表示在链表的最左端插入数 x。
R x，表示在链表的最右端插入数 x。
D k，表示将第 k 个插入的数删除。
IL k x，表示在第 k 个插入的数左侧插入一个数。
IR k x，表示在第 k 个插入的数右侧插入一个数。
输出格式
共一行，将整个链表从左到右输出。

数据范围
1≤M≤100000
所有操作保证合法。
```
输入样例：
10
R 7
D 1
L 3
IL 2 10
D 3
IL 2 7
L 8
R 9
IL 4 7
IR 2 2
输出样例：
8 7 7 3 2 9
```

# 解答

```python
N = int(input().strip()) + 10

class DoubleLinkList(object):
    def __init__(self):
        # 初始化双链表
        self.e = [0 for _ in range(N)]
        self.r = [0 for _ in range(N)]
        self.l = [0 for _ in range(N)]
        self.r[0] = 1
        self.l[1] = 0   # l[1] 表示最后一个节点
        self.idx = 2

    def insert(self, k, x):
        """在第k个插入的数后面添加数x"""
        self.e[self.idx] = x
        self.r[self.idx] = self.r[k]
        self.l[self.idx] = k
        self.l[self.r[k]] = self.idx
        self.r[k] = self.idx
        self.idx +=1

    def remove(self, k):
        """删除第k个插入的数后面的数"""
        self.r[self.l[k]] = self.r[k]
        self.l[self.r[k]] = self.l[k]

if __name__ == "__main__":
    d = DoubleLinkList()
    for i in range(N-10):
        in_li = list(input().split())
        op = in_li[0]

        if op=="L":
            x = int(in_li[1])
            d.insert(0,x)     # 添加头结点时应该直接在0号点的右侧添加，而不应该在r[0]的右侧添加
        elif op=="R":
            x = int(in_li[1])
            d.insert(d.l[1], x)  # l[1]表示最后一个节点
        elif op=="D":
            k = int(in_li[1])
            d.remove(k+1)
        elif op=="IL":
            k, x = map(int, in_li[1:])
            d.insert(d.l[k+1], x)
        else:
            k, x = map(int, in_li[1:])
            d.insert(k+1, x)

    i = d.r[0]
    # print(d.r)
    while i!=1:
        print(d.e[i], end=" ")
        i = d.r[i]

```