---
layout: post
title: "ACW17 KMP算法：831.KMP字符串"
subtitle: "The KMP Algorithm"
author: "qingshan"
header-img: "img/post-bg-halting.jpg"
header-mask: 0.4
tags:
  - 算法
  - ACW
---


给定一个模式串 S，以及一个模板串 P，所有字符串中只包含大小写英文字母以及阿拉伯数字。

模板串 P 在模式串 S 中多次作为子串出现。

求出模板串 P 在模式串 S 中所有出现的位置的起始下标。

输入格式
第一行输入整数 N，表示字符串 P 的长度。

第二行输入字符串 P。

第三行输入整数 M，表示字符串 S 的长度。

第四行输入字符串 S。

输出格式
共一行，输出所有出现位置的起始下标（下标从 0 开始计数），整数之间用空格隔开。

数据范围
1≤N≤105
1≤M≤106
```
输入样例：
3
aba
5
ababa
输出样例：
0 2
```

# 解法步骤：
1. 生成匹配字符串 p 的字符重复情况表 next[];
2. next[] 的元素说明：下标对应的元素值，表示之前在本字符串中已经出现过。例如[0,1,2]表示第二个元素和第三个元素之前都出现过，并且连续；
3. next[] 的元素值在非零的区间单调递增的；为 0 则表示当前位置未匹配（跳过）；
4. 分别对比被匹配字符串s和匹配字符串 p从 0 开始下标开始的元素；
5. 如果当前匹配上了，tar 和 pos 指针分别往后移动；
6. 如果没匹配上且 pos 不是在 0 位置，pos 就回退到 ne[pos-1] 位置；
7. 如果 5，6 都没成功，就说明没匹配上且 pos 在 0 位置。匹配失败，tar 往后移动一个；
8. 当某一时刻 pos==len(p)，说明 p 被完全匹配上。此时 tar-pos 就是在 s 中匹配成功的位置;
9. 结果返回


# 解答
```python
def getNext(p):
    ne=[0]        # next[0] 必然是 0
    x=1           # next[1] 开始
    now=0

    while x<len(p):
        if p[now] == p[x]:  # 如果p[now] == p[x], 向右拓展一位
            x+=1
            now+=1;
            ne.append(now)
        elif now != 0:            
            now = ne[now-1]  # 缩小 now, 改为 next[now-1]
        else:
            ne.append(0)     # now 已经为 0， 无法再缩小，next[x]=0
            x +=1

    return ne

def search(s, p, ne):
    tar=0               # 主串中将要匹配的位置
    pos=0               # pos: 模式串中将要匹配的位置

    while tar < len(s):
        if s[tar] == p[pos]:  # 若两个字符相等，则 tar, pos 各进一步
            tar +=1
            pos +=1
        elif pos != 0:        # 失配了，post 不为 0，继续依据 next 数组移动
            pos = ne[pos-1]
        else:
            tar +=1           # pos[0] 失配， 将标尺右移

        if pos==len(p):       # 匹配结束且成功
            print(tar - pos, end=" ")
            pos = ne[pos-1]   # 移动标尺



def main():
    n = int(input())
    p = input()
    m = int(input())
    s = input()

    ne = getNext(p)
    # print(ne)
    search(s, p, ne)

if __name__ == "__main__":
    main()

```
