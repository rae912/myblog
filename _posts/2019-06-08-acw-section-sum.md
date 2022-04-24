---
layout: post
title: "ACW09-区间合并"
subtitle: "The Section Sum"
author: "qingshan"
header-img: "img/post-bg-os-metro.jpg"
header-mask: 0.4
tags:
  - 算法
  - ACW
---


# ACW区间合并算法01: 803. 区间合并

给定 n 个区间 [li,ri]，要求合并所有有交集的区间。

注意如果在端点处相交，也算有交集。

输出合并完成后的区间个数。

例如：[1,3] 和 [2,6] 可以合并为一个区间 [1,6]。

输入格式
第一行包含整数 n。

接下来 n 行，每行包含两个整数 l 和 r。

输出格式
共一行，包含一个整数，表示合并区间完成后的区间个数。

数据范围
1≤n≤100000,
−109≤li≤ri≤109
```
输入样例：
5
1 2
2 4
5 6
7 8
7 9
输出样例：
3
```

# 解答

```python
def merge(num_list):
    start=end=0
    res = []
    
    for l, r in num_list:
        if end<l: # 如果当前区间的 end, 小于新的区间，说明两个区间独立，要新开一个
            if start != 0:  # 如果为 0，说明是第一个区间，不用初始化
                res.append([start,end])  # 存入结果
            
            # 覆盖 start, end，用新的区间值，用于下次对比
            start, end = l, r
        else:
            # 合并区间，主要就是 end 取较大值
            end = max(end, r)

    # 最后一次区间合并  
    res.append([start, end])
    return res
    
    
def main():
    n = int(input())
    nums_list = []
    for _ in range(n):
        l, r = list(map(int, input().split()))
        nums_list.append([l, r])
        
    # 排序：按照左边界
    nums_list.sort(key=lambda x:x[0])
    
    print(len(merge(nums_list)))
    
if __name__ == "__main__":
    main()
```
