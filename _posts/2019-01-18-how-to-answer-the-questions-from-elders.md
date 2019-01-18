---
layout: post
title: "过年走亲戚应答指南（数学版）——最优截至策略"
subtitle: "How to answer the questions from elders"
author: "qingshan"
header-img: "img/post-bg-infinity.jpg"
header-mask: 0.4
tags:
  - 数学
  - 生活
---


> 怎么还不找对象？
> 怎么还不结婚？
> 怎么还不买房子/车子？
> 怎么还不找/换工作？

> **临近年关，又到了要被长辈、亲戚“灵魂叩问”的时候了。按惯例，最近网络上会出现各式的过年花式应答攻略。除了这些抖机灵的“攻略”以外，其实早就有先贤针对这些问题给出了最科学的答案。如果你厌烦了那些博人眼球的“攻略”，不妨看看本文如何用科学的方法来回答以上那些让人措手不及的问题吧。**

### 引子：哲学家的捡麦穗问题
两千多年前一个阳光明媚的下午，一个希腊年轻人辗转找到大名鼎鼎的苏格拉底，向他请教如何才能收获最幸福的爱情。苏格拉底听完年轻人的倾诉，带他来到一片麦田，对着年轻人说：现在请你径直穿过这篇麦田，不能回头，在只拿一次的情况下交给我最大的麦穗。

年轻人不懂苏格拉底的用意，但是也照做了。一进入麦田，年轻人看到一颗饱满的麦穗，想摘下，但是又怕后面有更大的麦穗，于是一路纠结犹豫，可是越往后，却越没有大的麦穗。最终两手空空走出了麦田。苏格拉底见状，问：遗憾吗？年轻人回答：遗憾。苏格拉底说：这就是爱情。

苏格拉底又让年轻人再走另一片麦田，还是同样的规则。年轻人这次吸取教训，一进入麦田，看到一株比较大的麦穗，就果断摘下。可是他却发现，后面又遇到了许多更大更饱满的麦穗，可是他已不能再摘。当他把麦穗交给苏格拉底时，苏格拉底问：后悔吗？年轻人回答：后悔。苏格拉底说：这就是婚姻。

这就是历史上著名的“捡麦穗问题”。诚然，苏格拉底的回答多少带着哲学家深邃的寓意在内，并没有给出明确答案。但是，在这个故事被记载在羊皮卷之后一千多年的某天，一个数学家也遇到了同样的问题，而这次他运用知识给出了最优的解法。

### 提出问题：数学家的秘书问题
1949年，一个名叫Merrill M. Flood的美国数学家，遇到了和当时那个年轻人同样的问题，只不过，这次他的问题更加的具象化了一点，简直就是为了现在年轻人量身定做的一般。问题描述如下：

一个扬名海外的名人发布了征秘书启事，一时间应者如云。每个来应征的人，都有各自的优缺点。名人由于时间有限，只能在与每位应征者短暂的接触交流后立刻决定是否与应征者签订雇佣关系。一旦决定，则将婉拒其他未交流的应征者；而一旦犹豫，则眼前的应征者将离开，可能从此再也无法联系上。

这就是数学家的秘书问题，对这个问题的研究，也顺利的得到了最完美的答案。即：**以n/e为界限（n为应征者的数量，e为自然常数，约为2.718），之前所有的应征者全部拒绝，之后只要出现了比前面应征者综合条件最优的应征者相近或者更优的人，就立刻答应。**

也许你会问，这是为什么呢？那么下面就是解答的过程。如果对过程没有兴趣的话，可以直接跳到文末 **究极结论** 小节。

### 解决：数学建模
无论是“麦穗问题”还是“秘书问题”，本质上是一个数学模型问题，用数学符号描述如下：

> 有一个数量为N个的自然数集合[A,B,C, ...,N]，一一对应着N个值的随机数集合[a,b,c, ... n]，即1对应着a，2对应着b，3对应着c，依次类推N对应着n。现设计一种方法，在只取一次的情况下，使取到最大值对应的自然数的概率最大。

为了方便分析，我们令N=3，则自然数集合为[A,B,C]。最多可以尝试取三次，每次取值的可能的顺序情况如下：
![](https://ww1.sinaimg.cn/large/007i4MEmgy1fz9uts2p95j30kz036abw.jpg)

假设，a>b>c，则A对应的就是最大值，那么不管用哪种顺序取，随机取得A的概率都是三分之一。

现在设定一种策略，即先丢弃r个值，从r+1起，将其与r比较，如果大于才取，否则重复刚才的动作。再看取得A的概率，因为N=3，r只能设置为1或2。当r=2的时候，取得A的概率和随机一样都是三分之一；但是当r=1的时候，取得A的概率就扩大到了二分之一。说明在特定的条件下，这种策略有效。现在我们进一步尝试求解在什么条件下，这个概率能够最大。

这时候，作为程序员，还有什么比编写一个程序交给计算机计算来的容易呢。所以，在将相关逻辑写进代码后运行，计算机会计算出在所有可能的排列组合情况下的各种概率，结果如下：

>* 当设定前 5 个作为样本, 选中最优值的概率是15.625 %
>* 当设定前 10 个作为样本, 选中最优值的概率是23.401 %
>* 当设定前 15 个作为样本, 选中最优值的概率是28.82 %
>* 当设定前 20 个作为样本, 选中最优值的概率是32.682 %
>* 当设定前 25 个作为样本, 选中最优值的概率是35.069 %
>* 当设定前 30 个作为样本, 选中最优值的概率是36.452 %
>* 当设定前 35 个作为样本, 选中最优值的概率是36.744 %
>* 当设定前 40 个作为样本, 选中最优值的概率是36.67 %
>* 当设定前 45 个作为样本, 选中最优值的概率是36.09 %
>* 当设定前 50 个作为样本, 选中最优值的概率是34.957 %
>* 当设定前 55 个作为样本, 选中最优值的概率是33.125 %
>* 当设定前 60 个作为样本, 选中最优值的概率是30.937 %
>* 当设定前 65 个作为样本, 选中最优值的概率是28.168 %
>* 当设定前 70 个作为样本, 选中最优值的概率是25.161 %
>* 当设定前 75 个作为样本, 选中最优值的概率是21.472 %
>* 当设定前 80 个作为样本, 选中最优值的概率是18.099 %
>* 当设定前 85 个作为样本, 选中最优值的概率是13.721 %
>* 当设定前 90 个作为样本, 选中最优值的概率是9.476 %
>* 当设定前 95 个作为样本, 选中最优值的概率是4.8 %
>* 当设定前 100 个作为样本, 选中最优值的概率是1.051 %

（为了控制计算时间，在程序中将N设为了100）

从结果中可以看出，当N=100时，在大约将前35个元素设立为样本之后，取得最大值的概率最大约为36.7%。

而在将结果可视化之后，可以更直观的观察，最优方案应该是在将前37个元素设立样本区间，得到最大化概率为36.79%。如图所示。
![](https://ww1.sinaimg.cn/large/007i4MEmgy1fzaxdq6y53j30hd0afjrk.jpg)

当然，通过计算机暴力地计算出所有组合的概率，获得了最优解，但并没有普适性，还是应该利用数学的方法来求解有说服力。

*****（限于篇幅，文末可以获取完整程序代码）*****

### 证明：数学论证
对于任意的截断值 r，最优值选被选中的概率是：

![](https://ww1.sinaimg.cn/large/007i4MEmgy1fz9x9mfbwdj31b20a00vc.jpg)

求和符号内概率的计算是基于：如果应聘者 i 是最佳人选，它被选中当且仅当头 i - 1 个候选中的最佳人选处在头 r - 1 个被拒绝的候选者中。令 n 趋近无穷大，把 x 表示为 r/n 的极限，令 t 为 i/n，dt 为 1/n，总和可以近似为如下积分：

![](https://ww1.sinaimg.cn/large/007i4MEmgy1fz9xhp7da4j30lw04aq32.jpg)

其中，令 P(x) 对 x 的导数为 0，解出 x，x=1/e，最优截断值等于n/e（e为自然常数）。

这种解法，被称之为**最优截止策略**

（***本段资料引用自维基百科***）

### 究极结论
类似于“捡麦穗”和“秘书”问题，应该在样本总数前n/e（n>e时）设立样本区间，之后对比样本区间内的最优样本，取得最优样本的概率等于1/e（36.8%）。

当然过年可不能对长辈亲朋说得这么直接，而是应该这样回答：
**根据数学家的最优截止策略，我目前还不找对象/不结婚/不买房子/不买车子/不找工作/不换工作的原因是，为了找到/买到最好的，我还处于样本收集阶段。**

![](https://ww1.sinaimg.cn/large/007i4MEmgy1fzaxgbwaj2j30b40b4gm0.jpg)


### 题外拓展：什么是自然常数e？
自然常数作为自然界中非常神奇的存在，几乎经常能看到它的身影，例如以e为底形成的等角螺线：

>* 鹦鹉螺的贝壳像等角螺线
>* 菊的种子排列成等角螺线
>* 鹰以等角螺线的方式接近它们的猎物
>* 飞蛾等昆虫以等角螺线的方式接近光源
>* 蜘蛛网的构造与等角螺线相似
>* 旋涡星系的旋臂差不多是等角螺线。银河系的四大旋臂的倾斜度约为 12°
>* 低气压(热带气旋、温带气旋等)的外观像等角螺线

![](https://ww1.sinaimg.cn/large/007i4MEmgy1fzawf1bfl7j30go0gomz3.jpg)

还有细菌和野兔群体也在自然常数e的速率下繁殖能够最快速的扩大群体。所以自然常数e能够在个人问题上发挥作业，也是合情合理的。也许人类作为自然的一部分，也是受控于自然规律的呢。

大家如果有兴趣，后面也可以介绍一下作为自然规律作弊码而存在的**自然常数**。


### 附录：程序代码



```python
from __future__ import division
import numpy as np
import matplotlib.pyplot as plt
```


```python
def choose_candidate(n, reject=np.e):
    '''Choose a candidate from a list of n candidates using
    a specified strategy.

    reject: percentage of candidates to initially reject (optimal strategy by default)
    '''

    candidates = np.arange(1, n+1)
    np.random.shuffle(candidates)

    if reject == np.e:
        stop = int(round(n/reject))
    else:
        stop = int(round(reject*n/100))

    best_from_rejected = np.min(candidates[:stop])
    rest = candidates[stop:]

    try:
        return rest[rest < best_from_rejected][0]
    except IndexError:
        return candidates[-1]
```


```python
# choose from 100 candidates and run simulation 100,000 times
sim = np.array([choose_candidate(n=100) for i in range(100000)])

plt.figure(figsize=(10, 6))
plt.hist(sim, bins=100)
plt.xticks(np.arange(0, 101, 10))
plt.ylim(0, 40000)
plt.xlabel('Chosen candidate')
plt.ylabel('frequency')
plt.show()
```


```python
plt.figure(figsize=(10, 6))
plt.plot(np.cumsum(np.histogram(sim, bins=100)[0])/100000)
plt.ylim(0,1)
plt.xlim(0, 100)
plt.yticks(np.arange(0, 1.1, 0.1))
plt.xticks(np.arange(0, 101, 10))
plt.xlabel('Chosen candidate')
plt.ylabel('Cumulative probability')
plt.show()
```


```python
best_candidate = []
for r in range(5, 101, 5):
    sim = np.array([choose_candidate(n=100, reject=r) for i in range(100000)])
    # np.histogram counts frequency of each candidate
    best_candidate.append(np.histogram(sim, bins=100)[0][0]/100000)

plt.figure(figsize=(10, 6))
plt.scatter(range(5, 101, 1), best_candidate)
plt.xlim(0, 100)
plt.xticks(np.arange(0, 101, 10))
plt.ylim(0, 0.4)
plt.xlabel('% of candidates rejected')
plt.ylabel('Probability of choosing best candidate')
plt.grid(True)
plt.axvline(100/np.e, ls='--', c='black')
plt.show()
```


```python
for i in xrange(len(best_candidate)):
    print(u"当设定前 {} 个作为样本, 选中最优值的概率是{} %".format((i+1)*5, best_candidate[i] * 100))
```


```python
def get_best_candidates(best_n=1):
    '''Return a list of probabilities for different rejection strategies and specify what percentage of the
    best candidates we want to select.'''

    best_candidate = []
    for c in [1] + range(5, 101, 5):
        sim = np.array([choose_candidate(100, reject=c) for i in range(10000)])
        best_candidate.append(len(sim[sim <= best_n])/100)

    return best_candidate
```


```python
plt.figure(figsize=(10, 6))
for i in [1, 2, 5, 10]:
    plt.scatter(range(0, 101, 5), get_best_candidates(i), label=str(i))
plt.xlim(0, 100)
plt.ylim(0, 100)
plt.xticks(np.arange(0, 101, 5))
plt.yticks(np.arange(0, 101, 10))
plt.xlabel('% of candidates rejected')
plt.ylabel('Probability of choosing best candidates')
plt.legend(title='No. of best candidates')
plt.grid(True)
plt.tight_layout()
plt.show()
```



