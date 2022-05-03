---
layout: post
title: "ACW19 并查集2： 837. 连通块中点的数量"
subtitle: "The set merge 02"
author: "qingshan"
header-img: "img/home-bg-o.jpg"
header-mask: 0.4
tags:
  - 算法
  - ACW
---


# ACW 并查集02： 837. 连通块中点的数量

给定一个包含 n 个点（编号为 1∼n）的无向图，初始时图中没有边。

现在要进行 m 个操作，操作共有三种：

1. `C a b`，在点 a 和点 b 之间连一条边，a 和 b 可能相等；
2. `Q1 a b`，询问点 a 和点 b 是否在同一个连通块中，a 和 b 可能相等；
3. `Q2 a`，询问点 a 所在连通块中点的数量；
输入格式
第一行输入整数 n 和 m。

接下来 m 行，每行包含一个操作指令，指令为 C a b，Q1 a b 或 Q2 a 中的一种。

输出格式
对于每个询问指令 Q1 a b，如果 a 和 b 在同一个连通块中，则输出 Yes，否则输出 No。

对于每个询问指令 Q2 a，输出一个整数表示点 a 所在连通块中点的数量

每个结果占一行。

数据范围
1≤n,m≤1e5
```
输入样例：
5 5
C 1 2
Q1 1 2
Q2 1
C 2 5
Q2 5

输出样例：
Yes
2
3
```


# 解答
```python
N = 100010

class trie(object):
    def __init__(self, n):
        """初始化"""
        self.p = [0] * N
        self.size= [0] * N
        for i in range(n):
            # 初始状态：每个索引位值是自己。即节点的父节点是自己
            self.p[i] = i
            self.size[i] = 1

    def find(self, x):
        """查找祖先节点"""
        if self.p[x] != x:
            # 递归。注意这里等号左边p[x]的值在递归结束时候，会被覆盖
            # 即所有动态 p[x] 最终都会被覆盖成那个 p[x] = x 的  p[x] 值，等同 x
            self.p[x] = self.find(self.p[x])

        return self.p[x]

    def merge(self, a, b):
        """合并两个值"""
        # 意思就是将 a 值的祖先节点，指向 b 的祖先节点。
        # 两个值有同一个祖先节点，说明就是在同一个集合
        self.p[self.find(a)] = self.find(b)


def main():
    n, m = list(map(int, input().split(" ")))
    t = trie(n)

    for _ in range(m):
        s = list(input().split(" "))
        opt = s[0]
        if opt == 'C':
            a = int(s[1])
            b = int(s[2])
            if t.find(a) == t.find(b):
                continue
            else:
                t.size[t.find(b)] += t.size[t.find(a)]
                t.p[t.find(a)] = t.find(b)

        elif opt == 'Q1':
            a = int(s[1])
            b = int(s[2])
            if t.find(a) == t.find(b):
                print("Yes")
            else:
                print("No")

        else:
            a = int(s[1])
            print(t.size[t.find(a)])

if __name__ == "__main__":
    main()

```