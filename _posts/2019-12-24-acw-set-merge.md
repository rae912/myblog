---
layout: post
title: "ACW19-并查集: 836. 合并集合"
subtitle: "The set merge"
author: "qingshan"
header-img: "img/home-bg-o.jpg"
header-mask: 0.4
tags:
  - 算法
  - ACW
---


# ACW 并查集01： 836. 合并集合
一共有 n 个数，编号是 1∼n，最开始每个数各自在一个集合中。

现在要进行 m 个操作，操作共有两种：

`M a b`，将编号为 a 和 b 的两个数所在的集合合并，如果两个数已经在同一个集合中，则忽略这个操作；
`Q a b`，询问编号为 a 和 b 的两个数是否在同一个集合中；
输入格式
第一行输入整数 n 和 m。

接下来 m 行，每行包含一个操作指令，指令为 `M a b` 或 `Q a b` 中的一种。

输出格式
对于每个询问指令 `Q a b`，都要输出一个结果，如果 a 和 b 在同一集合内，则输出 Yes，否则输出 No。

每个结果占一行。

数据范围
1≤n,m≤1e5
```
输入样例：
4 5
M 1 2
M 3 4
Q 1 2
Q 1 3
Q 3 4
输出样例：
Yes
No
Yes
```


# 解答
```python
N = 100010

class trie(object):
    def __init__(self, n):
        """初始化"""
        self.p = [0] * N
        for i in range(n):
            # 初始状态：每个索引位值是自己。即节点的父节点是自己
            self.p[i] = i

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
    n, m = list(map(int, input().split()))
    t = trie(n)

    for _ in range(m):
        opt, a, b = list(input().split())
        a = int(a)
        b = int(b)
        if opt == "M":
            t.merge(a, b)
        else:
            # 两个值如果有同一个祖先节点，就是在同一个集合
            if t.find(a) == t.find(b):
                print("Yes")
            else:
                print("No")

if __name__ == "__main__":
    main()

```