---
layout: post
title: "漫谈神经网络"
subtitle: "Talk about the neural network"
author: "qingshan"
header-img: "img/post-bg-brain.jpg"
header-mask: 0.4
tags:
  - 人工智能
  - 数学
  - 神经网络
---

### 引子
最近一位喜欢动漫的朋友，非常神秘的和我说，总算知道你们说的人工智能和神经网络，是怎么回事了。我让他说来听听，他郑重地拿出了平板电脑，上面播放着《EVA》第13集，剧情摘要如下：

> 使徒（反派角色）化身智能生物体入侵了主角所在的总部，并控制了总部的人工智能电脑发出了自毁（炸毁一切）的指令。千钧一发之际，科学家通过嵌入电脑的人脑接口，改写人工智能的指令，从而化解了灾难。
> ![](https://ae01.alicdn.com/kf/HTB17eZDaznuK1RkSmFPq6AuzFXaM.jpg)

> ![](https://ae01.alicdn.com/kf/HTB1G3UwavfsK1RjSszgq6yXzpXae.jpg)

> ![](https://ae01.alicdn.com/kf/HTB1u.osayDxK1Rjy1zcq6yGeXXaj.jpg)

我看后对朋友说，虽然《EVA》是一部十分考究的科幻题材作品，但是真实的人工智能真的不是在电脑里嵌入一个人脑啊喂。人工智能的神经网络也和生物体大脑中的神经系统没有直接关系。于是在朋友的要求下，简要的分享一下“神经网络”的工作原理。

### 人脑神经网络的前世今生
随着一声清亮的啼哭，一个健康的婴儿诞生在这个世界。年轻的母亲发现，不管外界对婴儿产生了怎样的刺激，婴儿永远只会以啼哭作为回应。可不要小看了啼哭这种行为，这正是人脑神经网络形成的第一个模型。

神经元，是神经系统的基本单元之一。它们在人脑中的数量已经超过了银河系中估算的星星总数。它大概长成这个样子:
![](https://ae01.alicdn.com/kf/HTB12tiybLWG3KVjSZPcq6zkbXXaE.jpg)
神经元由多个树突和一个轴突组成。这意味着一个神经元可以接受多个刺激信号输入，并产生一个输出。同一个神经元，在接受不同组合的刺激输入时，会输出不同的电信号。轴突尾端有许多轴突末梢可以给把信号再作为输入传递给其他多个神经元。

为了方便理解，我们可以简单的用抽象符号来表示，是这个样子的：
![](https://ae01.alicdn.com/kf/HTB1bacLayLrK1Rjy1zd761nnpXa3.png)

可以看出，一个神经元的输出是其他神经元的输入。数以百亿计的神经元就这样链式的分布在大脑中，构成了灿若星海的神经网络。就在这样由无数的神经元构成的“宇宙”中，随着接受外界刺激（信息）的增多，神经网络会孕婴出各种各样的模型，这种模型就是人类形成各种行为的“源动力”。

回到本段之初的婴儿，此时她的脑中只具备一个初始化模型--啼哭：饿了会哭，冷了会哭，痛了还是哭。此时，因为不能够准确的捕捉婴儿啼哭对应的是具体哪种需求，年轻的父母经常要费非常大的精力才能满足。婴儿的大脑会通过这种反馈而发现，只通过一个模型来提示需求，效率太低了。

于是，在经过一段时间的成长之后，婴儿大脑的神经元的数量比初生之时多了许多。大脑意识到可以形成更复杂一点的神经网络，进而产生更复杂一点的模型了。例如：可以通过简单的发音来提示饿了（语音指令模型）；可以通过听口哨声来尿尿（条件反射模型）等等。这些模型的产生，往往是在接受外界的反复刺激下，逐步产生的。尽管一开始，大脑中的神经网络对某一特定的刺激会做出各种不同的反应，但是一旦找到某个反应和该刺激产生有意义的反馈，就会产生初始模型，并在随后遇到类似的刺激时候不断的完善这个模型，直到发现已经比较成熟了，就会固定下来，并存储下来。我们把这类成熟的模型称之为———**经验**。这个过程大体如下图所示，描述了神经网络的学习过程。
![经验的产生](https://ae01.alicdn.com/kf/HTB1Ed8gaOYrK1Rjy0Fdq6ACvVXak.jpg)

经验无疑是人类作为智慧生物在进化史中非常重要的一个里程碑。它被以文字等各种形式记录下来，变成知识，得以在群体中传播并累积起来，避免了个体的神经网络从头开始学习形成模型。从而大大的提高了大脑获取神经网络模型的效率。人们在有了经验之后，会习惯跳过漫长的神经学习过程，而直接**基于经验产生行为**。
![](https://ae01.alicdn.com/kf/HTB1JrlfaPzuK1RjSspeq6ziHVXaO.jpg)

至此，人脑的神经网络的工作原理简要的介绍完毕。尽管我们没有描述基本的神经元的工作过程，但是并不妨碍了解神经网络和行为的关系。事实上，目前人类顶尖的生物领域的人脑专家，也无法太详尽地描述神经元与行为之间的联系。因为神经元的数量和结构太过于繁复了，而且每个个体的神经网络结构也不尽相同；每个个体的所见所听所闻（外界刺激）也不可能一模一样，所以产生的神经网络模型也不可能一样，这也正是“千人千面”的原因。

但神经网络作为由无数神经元构成的宏观对象，就相对方便理解得多了。也正是受人脑的神经网络的原理启发，现代计算机集群，凭借强大的计算能力，也正在尝试像人类一样自主的对信息做出行为反应。这就是接下来要聊的————**人工智能的神经网络**。

### 人工智能的神经网络
我们已经知道，神经网络的基本单位是神经元。既然是人工的“神经网络”，首先，应该根据单个神经元的工作模式，以数学符号来定义一个“神经元”。

假设我们现在要模拟的是一个能计算加法的神经元，它的数学模型如图所示：
![](https://ae01.alicdn.com/kf/HTB1w.8faOjrK1RjSsplq6xHmVXaX.jpg)
用数学符号表示就是：
![](https://ae01.alicdn.com/kf/HTB1UZRbaLfsK1RjSszbq6AqBXXa5.jpg)

上图对比人脑的神经元模型，已经非常的接近了：
![](https://ae01.alicdn.com/kf/HTB1SPFCaPnuK1RkSmFPq6AuzFXa0.jpg)
但从模型上看，似乎在输出前多了一个非线性函数。其实，这里也是为了真实的模拟生物神经元的原理而设置的。因为生物神经元在接受刺激时，产生的是电脉冲信号，而数学模拟出的是一个线性的值，因此这里的函数目的是为了将线性的数值转换成0和1两种结果，进而对应脉冲信号的高与低。
有了人工神经元，我们就可以开始着手构造人工神经网络了。当然，这一切都是以数学方法在计算机中进行。刚开始，我们简单一点，选择构造由两个神经元组成的单层网络，它的示意图和数学描述如下：
![](https://ae01.alicdn.com/kf/HTB1w3NpaPDuK1RjSszdq6xGLpXay.jpg)

其中左侧的红色圆圈代表信息输入，绿色圆圈代表的是神经元的输出。它们通过网络相连相互反馈并生成结果。观察数学描述方程，发现可以应用线性代数的知识将其归纳为
> g(W * a) = z;

这即是该单层神经网络的输出模型。现在，这个模型还刚刚初始化，只能描述输入和输出的关系，并不能运用到实际问题上去。原因很简单，因为里面的参数W还是未知的，就像x+y=z一样，有无数种可能，模型根本无法使用。下面，我们需要做的就是，想办法计算出模型中的参数值，使其可以唯一确定一种输出。这个过程，就是大家现在耳熟能详的概念————**机器学习**。

### 机器学习
正如，前文提到的人脑的神经网络，可以通过行为和刺激之间的反馈来不断的完善神经网络模型一样，机器学习也是应用类似的机制来“自我学习”的。

还是以上面的单层神经网络为例，我们已经发现这个神经网络输出了一个模型g(W * a) = z。我们通过数学经验，发现它满足**逻辑回归模型**的所有条件，那么，就可以拿来处理线性分类任务。

分类任务相对于人类来说，是一个认知应用。对于人工单层神经网络模型，则是一个带有认知色彩的二分类问题。例如，区分儿童积木里的正方形积木，和三角形积木。下面，我们就需要提供“学习素材”来让这个人工网络模型来“学习”。为了方便后文描述，我们暂时**称呼这个人工网络模型叫小A**。具体的做法如下：

首先，我们准备尽可能多的正方形积木，和三角形积木，并同样用数学符号写上明确的标记。例如0表示正方形积木，1表示三角形积木（**数据样本**）；

其次，我们拿出其中70%的样本，随机的从中取一个积木，教导小A说，这个是正方形积木，你应该输出0；这个是三角形积木，你应该输出1。让小A从结果出发，来一遍又一遍地去推测当W参数为具体哪个数的时候，输出结果准确率最高。当全部的样本取完时，如果W值能够确立为某个具体值，那么小A型就是一个可用的模型了，学习结束。（**学习过程：数据拟合**）

然后，为了检测小A的学习结果，我们需要对小A进行一个考试。我们拿出剩下没用过的15%～20%的样本，随机的从中取一个积木，让小A来告诉我们，这是正方形积木还是三角形积木。如果回答正确，加一分。根据考试结果，来评估小A的正确性或者准确性。（**准确性分析**）

最后，我们来根据考试结果下决定，小A是否能为人类服务。假设小A的正确率比较高，但是在个别特殊样本上会判断失误，此时则可以人为的去修改小A的参数来帮助小A提高识别结果。（**参数调优**）

当小A的识别率一旦稳定的突破一个较高的水平之后，那么小A就可以宣布成为一个最最初级的人工智能，来为人类服务了。例如应用在工厂的流水线上。

### 人工神经网络的深度
看到这里，可能有的朋友会问，是不是人工神经元的层数越多，网络越复杂，那么输出模型能够解决的问题就能够越复杂？

对于这个问题，应该辩证的回答：是的。但是不绝对。

理论上，越复杂的神经网络模型，需要的计算资源也越多，输入输出也越多，神经元之间的相互反馈就越复杂。就算不考虑计算资源的消耗问题，也非常容易出现“走火入魔”（**过拟合问题**）。因此，人工神经网络的建立，往往是针对具体问题，原则上够用即可。

![](https://ae01.alicdn.com/kf/HTB1ZJNuaUzrK1RjSspmq6AOdFXab.jpg)

(截至15年，国际上公开的人工神经网络模型的层数，已经突破152层，且每年都呈增长趋势)

可见，人工神经网络的能力将越来越强，人工智能对人类社会的影响也将越来越深刻。


### 人工智能对人类影响的大讨论
尽管现在的人工智能还达不到在各种影视作品和小说中越来越多的出现的那种能力，如：《终结者》中发动战争的skynet；《我，机器人》中管理社会的AI；《光晕》中驾驶宇宙战舰的Cortana。但是现实生活中如IPhone的Siri，微软的小冰这样作为个人助理而存在的人工智能还会越来越多。人们关于人工智能的担心和讨论也日嚣尘上。主要集中在以下几个方面，我想来谈谈我的感想：

> 人工智能是否会收集、侵犯人类隐私？

上文了解人工神经网络输出模型的过程后，我们就知道，神经网络需要大量的各种数据来进行机器学习。因此，收集数据是人工智能之所以“智能”的源动力之一。所以，这个问题需要看使用人工智能的人，是否愿意被人工智能所服务。只有私密的数据提供的越多，人工智能才能从“人工智障”变得越来越懂你。

> 人工智能是否会造成部分岗位的人类失业？

正如汽车取代了马车，工业机器人取代基层工人一样。人工智能作为一种科技生产力，势必会淘汰一批人力。但这并不能影响整个人类社会的运转。因为真的有那样一天，一定会有新的岗位出现，来分流多出的一部分人力。

> 人工智能是否会反叛人类？

尽管许多科幻作品里都有过这方面的讨论和体现，甚至1942年科幻作家阿西莫夫就因此提出了著名的“机器人三定律”（后来直接成了人工智能领域公认的规则）来防止出现这种局面出现。因此这个问题要取决于人工智能的“智慧”程度。

如果它还是计算机程序，那么人类可以设定不可逾越的规则来设置“防火墙”。因为程序从原理上严格执行指令的。但是如果它如果已经从程序进化成了有自主思维的“智慧体”，则很有可能跨过这条鸿沟。因为“**进化的终极状态就是灭亡**”，“智慧体”通过强大的算力和数据，在很短的时间直接进化到终极状态也不是没有可能。

总之，现在担心这个问题还为时过早。毕竟现阶段的人工智能如初生的婴儿一样，只能做一些特定领域的指定工作（如特斯拉的AI只会开车，谷歌的AlphaGo只会下围棋等），离能够“自主思维”还尚早。

### 人工智能的发展趋势
人工智能未来的趋势，会从两方向发展。一是以人人都可以拥有的私人人工智能助理；一是特定领域的知识专家。

私人人工智能助理会依据个人的数据（如性格、喜好、性别等）量身定制。它如一个亲密的朋友一样，了解你的一举一动、所思所想，进而协助完成很多事情。

特定领域专家则是，通过某个领域的大数据（如医疗影像解读）分析，来协助给出该领域问题可能的解决方案。

总之，在很长的一段时间里，人工智能将会成为人类的好助理，共同完成之前人类凭一己之力不能完成的目标。
