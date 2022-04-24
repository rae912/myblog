---
layout: post
title: "ACW10 单链表：826. 单链表"
subtitle: "The Single Linked List"
author: "qingshan"
header-img: "img/home-bg-o.jpg"
header-mask: 0.4
tags:
  - 算法
  - ACW
---


实现一个单链表，链表初始为空，支持三种操作：

向链表头插入一个数；
删除第 k 个插入的数后面的数；
在第 k 个插入的数后插入一个数。
现在要对该链表进行 M 次操作，进行完所有操作后，从头到尾输出整个链表。

注意:题目中第 k 个插入的数并不是指当前链表的第 k 个数。例如操作过程中一共插入了 n 个数，则按照插入的时间顺序，这 n 个数依次为：第 1 个插入的数，第 2 个插入的数，…第 n 个插入的数。

输入格式
第一行包含整数 M，表示操作次数。

接下来 M 行，每行包含一个操作命令，操作命令可能为以下几种：

H x，表示向链表头插入一个数 x。
D k，表示删除第 k 个插入的数后面的数（当 k 为 0 时，表示删除头结点）。
I k x，表示在第 k 个插入的数后面插入一个数 x（此操作中 k 均大于 0）。
输出格式
共一行，将整个链表从头到尾输出。

数据范围
1≤M≤100000
所有操作保证合法。
```
输入样例：
10
H 9
I 1 1
D 1
D 0
H 6
I 3 6
I 4 5
I 4 5
I 3 4
D 6
输出样例：
6 4 6 5
```

# 解答

```python
N = 100010

class LinkList(object):
    def __init__(self):
        self.head = -1     # head表示头节点的开始索引，当head为-1时，链表为空
        self.e = [0] * N   # 存放每个链表节点的值
        self.ne = [0] * N  # 存放每个链表节点指向下一个节点的指针值
        self.idx = 1       # idx表示正在用链表中的那一个节点    

    def addToHead(self, x):  # 将x添加到头节点
        self.e[self.idx] = x
        self.ne[self.idx] = self.head
        self.head = self.idx
        self.idx+=1


    def remove(self, k):     # 删除第k个输入的数后面的数
        if k == 0: # 当k为0时，表示删除头结点
            self.head = self.ne[self.head]
            return
        self.ne[k] = self.ne[self.ne[k]]  # ne[k] 指向ne[k]的下个节点

    def insert(self, k, x):   # 在第k个输入的数后面插入一个数x
        self.e[self.idx] = x
        self.ne[self.idx] = self.ne[k]
        self.ne[k] = self.idx
        self.idx+=1

def main():
    l = LinkList()
    m = int(input())
    for _ in range(m):
        s, *p = input().split()
        if s == "I":
            k, x = map(int, p)
            l.insert(k, x)
        elif s == "H":
            x = int(p[0])
            l.addToHead(x)
        else:
            k = int(p[0])
            l.remove(k)

    tail = l.head  # 从头节点开始遍历整个链表并输出每个节点的值
    while tail != -1:
        print(l.e[tail], end=" ")
        tail = l.ne[tail]


if __name__ == "__main__":
    main()


```