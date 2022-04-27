---
layout: post
title: "ACW18-trie树：835. Trie字符串统计"
subtitle: "The trie tree"
author: "qingshan"
header-img: "img/home-bg-o.jpg"
header-mask: 0.4
tags:
  - 算法
  - ACW
---


维护一个字符串集合，支持两种操作：

1. I x 向集合中插入一个字符串 x；
2. Q x 询问一个字符串在集合中出现了多少次。

共有 N 个操作，输入的字符串总长度不超过 105，字符串仅包含小写英文字母。

### 输入格式
第一行包含整数 N，表示操作数。

接下来 N 行，每行包含一个操作指令，指令为 I x 或 Q x 中的一种。

### 输出格式
对于每个询问指令 Q x，都要输出一个整数作为结果，表示 x 在集合中出现的次数。

每个结果占一行。

数据范围
1≤N≤2∗1e4
```

输入样例：
5
I abc
Q abc
Q ab
I ab
Q ab
输出样例：
1
0
1
```

# 解答
```python
N = 100010

class Trie(object):
    def __init__(self):
        """初始化"""
        self.idx = 0
        self.cnt = [0] * N
        self.son = [[0] * 26 for _ in range(N)]


    def insert(self, characters):
        p = 0
        for i in characters:
            # u 值取字母表中字母的顺序，a=0, b=1, c=2以此类推
            u = ord(i) - ord('a')
            # son[p][u]=0 表示当前节点不存在
            if self.son[p][u] == 0:
                # 新建节点
                self.idx +=1
                self.son[p][u] = self.idx

            # 指向下一个节点
            p = self.son[p][u]

        # 结束，记录以此节点结束的字符串出现次数
        self.cnt[p] += 1


    def query(self, characters):
        p = 0
        for i in characters:
            u = ord(i) - ord('a')
            # 如果该节点为 0，表示没有创建过，不存在
            if self.son[p][u] == 0:
                return 0

            p = self.son[p][u]

        # 返回该节点出现的次数
        return self.cnt[p]


def main():
    n = int(input())

    t = Trie()

    for _ in range(n):
        op, characters = list(input().split())

        if op == "I":
            t.insert(characters)
        else:
            print(t.query(characters))


if __name__ == "__main__":
    main()  

```