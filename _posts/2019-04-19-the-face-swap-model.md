---
layout: post
title: "什么？！女神“下海”了？"
subtitle: "The face swap model"
author: "qingshan"
header-img: "img/post-bg-beautiful-woman.jpg"
header-mask: 0.4
tags:
  - 人工智能
  - 神经网络
---


某天，小伙伴给青衫分享了一个羞羞哒小视频，看封面怎么感觉有点眼熟呢？点开后的画风是这样子的：
![](https://ae01.alicdn.com/kf/HTB1oYm0PxjaK1RjSZKz5jXVwXXah.gif)
我去！这不是女神盖尔加朵（神奇女侠扮演者）吗？！什么情况？！

青衫感觉事情肯定没这么简单。于是顺着视频上的水印找到了视频出处网站。这一下可真是发现了新大陆：
![](https://ae01.alicdn.com/kf/HTB15nH8RSzqK1RjSZFj5jblCFXaG.gif)
除了**盖尔·加朵，连泰勒·斯威夫特（歌手霉霉）、艾玛·沃特森（哈利波特里的女主）、斯嘉丽·约翰逊（复联黑寡妇）**的视频都有！连青衫非常喜欢的演员新垣结衣也不能幸免！

![](https://ae01.alicdn.com/kf/HTB1jiG6PxjaK1RjSZKzq6xVwXXaz.jpg)

冷静下来之后，青衫通过查阅资料，得知原来这一切视频都是假的！都是通过**FakeFace**技术**伪造**的！

## FakeFace是什么？
简单来说，FakeFace是通过神经网络模型，通过程序，将一个视频中人物的脸部替换成另外一个人的。它的工作原理就是首先分析被伪造的人物A的视频或者照片，提取其中人物各个角度的面部特写作为素材，然后以素材还原出人物A脸部的所有特征，并具备转换成任意角度的脸部模型。再分析将要被替换的视频中人物B的脸部各个角度的特写，然后从目标A的脸部模型中找出/生成最贴合B当下这个角度的脸部特征，然后逐帧替换。达到“移花接木”的特效。

这种技术，虽说以前的好莱坞工业已经有过实现，但是需要耗费巨大的人力和时间可以实现。但是现在，一台带显卡的普通电脑花费几个小时就可以自动完成这项任务。那么它是怎么做到的呢？

## 技术实现
目前在FakeFace的技术领域，有众多的实现算法。在这里，就只针对效果最好的算法之一的Faceswap-GAN来简单分析一下。其中FaceSwap指的是https://github.com/deepfakes/faceswap 算法。它用监督学习训练一个神经网络模型，将扭曲处理过的脸还原成原始脸，并且期望这个网络具备将任意人脸还原成张三的脸的能力。采用的是自编码模型。

而Faceswap-GAN是在前者的基础之上增加了Adversarial Loss和Perceptual Loss。其中Perceptual Loss    是用训练好的VGGFace网络（该网络不做训练）的参数做一个语义的比对。总体上利用GAN对抗网络来增强替换效果。

它的原理示意图如下：
![](https://ae01.alicdn.com/kf/HTB1dxOKRMTqK1RjSZPhq6xfOFXax.jpg)
![](https://ae01.alicdn.com/kf/HTB1PzLxRSrqK1RjSZK9q6xyypXaH.jpg)

从图中可以看出，想要将一个人的脸“换”到另一个人的身上，只需要三步：
>* 第一步：提取原始素材里的人脸图像数据；
>* 第二步：训练模型；
>* 第三步：根据模型生成目标图像并逐帧覆盖；

经过这一系列操作，就可以成功的进行“换脸”了，就像这样便可以将影帝尼古拉斯凯奇变成美帝总统啦：
![](https://ae01.alicdn.com/kf/HTB1LBzJRQvoK1RjSZPfq6xPKFXaU.jpg)

### 技术实践
如果这一切你觉得有兴趣的话，国外有大神已经将这一切的程序封装成了傻瓜式的软件。只需要提供要替换人物的原始视频，就可以将其替换到其他的视频里面。软件的界面十分简洁，目的就是让没有基础的用户也能够使用。
![](https://ae01.alicdn.com/kf/HTB1p2wzRNjaK1RjSZKzq6xVwXXaP.jpg)
### 应用领域
随着上面介绍到的软件在网上走红之后，许多动手能力强的同学已经着手开始“导演”各种让人脑洞大开的小视频了。例如杨幂"主演"的94版《射雕英雄传》，和各种明星客串的百变女主播。
![](https://ae01.alicdn.com/kf/HTB1BMUbRNnaK1RjSZFt5jbC2VXas.gif)


除了在娱乐领域应用这项技术用来看妹子以外，更让青衫感到惊讶的是有日本有位叫Kizashi Nakano的**“吃货”**小哥，居然将这项技术应用在了吃上面！可以将一碗素拉面瞬间零成本变成豪华海鲜拉面！转变后的画风是这样的：
![](https://ae01.alicdn.com/kf/HTB1.K_QRIbpK1RjSZFy5jX_qFXaI.gif)

原来，这位小哥因为疾病，不能享受到海鲜拉面的美味。因此，在通过学习相关技术原理之后，于是就一头扎进了研究之中。现在，他开发的这套系统已经可以**将素面变成拉面或者炒面，将白米转换成咖喱饭或炒饭，可以在吃一种食物的时候体验到一种仿佛在吃另一种食物的味觉幻觉**。
这个应用在可以遇见的未来，能够对厌食症病人带来巨大的福音。这可比看妹子什么的有意义多了！试想一下，当正在减肥的你忍不住想吃大餐的时候，运行小哥的这套程序，就可以将水果沙拉吃出啤酒炸鸡的味道，还不用担心减肥大计功亏一篑，该是多么美好的事情啊。

鉴于有十分广阔的社会应用前景和意义，小哥的这篇论文也因此被IEEEVR（计算机领域顶尖的学术组织）收录。

可见，当“需求”和“技术”结合到一起的时候，总是能碰撞出让人惊喜的火花。下一个让人振奋的应用，就等你来发现了。

### 参考资料
https://www.shangyexinzhi.com/Article/details/id-90522/

https://github.com/deepfakes/faceswap

https://github.com/shaoanlu/faceswap-GAN

https://zhuanlan.zhihu.com/p/34042498










