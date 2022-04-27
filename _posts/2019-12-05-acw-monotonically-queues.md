---
layout: post
title: "ACW16-单调队列"
subtitle: "ACW16-Monotonically Queues"
author: "qingshan"
header-img: "img/home-bg-o.jpg"
header-mask: 0.4
tags:
  - 算法
  - ACW
---

# ACW单调队列01: 154. 滑动窗口

给定一个大小为 n≤106 的数组。

有一个大小为 k 的滑动窗口，它从数组的最左边移动到最右边。

你只能在窗口中看到 k 个数字。

每次滑动窗口向右移动一个位置。

以下是一个例子：

```
该数组为 [1 3 -1 -3 5 3 6 7]，k 为 3。

窗口位置	最小值	最大值
[1 3 -1] -3 5 3 6 7	-1	3
1 [3 -1 -3] 5 3 6 7	-3	3
1 3 [-1 -3 5] 3 6 7	-3	5
1 3 -1 [-3 5 3] 6 7	-3	5
1 3 -1 -3 [5 3 6] 7	3	6
1 3 -1 -3 5 [3 6 7]	3	7
```
你的任务是确定滑动窗口位于每个位置时，窗口中的最大值和最小值。

输入格式
输入包含两行。

第一行包含两个整数 n 和 k，分别代表数组长度和滑动窗口的长度。

第二行有 n 个整数，代表数组的具体数值。

同行数据之间用空格隔开。

输出格式
输出包含两个。

第一行输出，从左至右，每个位置滑动窗口中的最小值。

第二行输出，从左至右，每个位置滑动窗口中的最大值。

```
输入样例：
8 3
1 3 -1 -3 5 3 6 7
输出样例：
-1 -3 -3 -3 3 3
3 3 5 5 6 7
```

## 解答

```python

class Window(object):
    def __init__(self, a):
        self.a = a
        self.q = [0 for _ in range(len(a) + 10)]
        
    def getMinimum(self, n, k):
        """单调递增数列, q中最左边就是最小值"""
        hh, tt = 0, -1
        for i in range(n):
            # 如果头指针指向的位置超出了窗口宽度，头指针后移 1
            if hh<=tt and i-k+1 > self.q[hh]:
                hh +=1
            
            # 如果q最后位置指向 a 的值大于当前值，那么 tt 往前移动 1（等价替代掉较大值）
            while hh<=tt and self.a[self.q[tt]] >= self.a[i]:
                tt -=1
                
            tt +=1
            self.q[tt] = i
            # 当窗口即将往后移动时候，
            if i>=k-1:
                # 输出当前 q 中头部指针位置，输出 a 中最小值
                print(self.a[self.q[hh]], end = " ")
                
        print()
        
    def getMaximum(self, n, k):
        """单调递减数列，q 中最左边就是最大值"""
        hh, tt = 0, -1
        for i in range(n):
            # 如果头指针指向的位置超出了窗口宽度，头指针后移 1
            if hh<=tt and i-k+1>self.q[hh]:
                hh +=1
            
            # 如果q最后位置指向 a 的值大于当前值，那么 tt 往前移动 1（等价替代掉较小值）
            while hh<=tt and self.a[self.q[tt]] <= self.a[i]:
                tt -=1
                
            tt +=1
            self.q[tt] = i
            if i>=k-1:
                print(self.a[self.q[hh]], end = " ")
                
        print()
        
if __name__ == "__main__":
    n, k = list(map(int, input().split()))
    a = list(map(int, input().split()))
    
    w = Window(a)
    w.getMinimum(n, k)
    w.getMaximum(n, k)
                
```