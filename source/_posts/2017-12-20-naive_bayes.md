---
title:         朴素贝叶斯的那点事儿
date:        2017-12-20 18:00
author:     "Sylvanas Sun"
catalog:    true
categories: 
    - 机器学习
    - 监督式学习
tags:
    - 机器学习
    - 监督式学习
    - 贝叶斯
    - 2017
---



在机器学习领域中，朴素贝叶斯是一种基于贝叶斯定理的简单概率分类器（分类又被称为监督式学习，所谓监督式学习即从已知样本数据中的特征信息去推测可能出现的输出以完成分类，反之聚类问题被称为非监督式学习），朴素贝叶斯在处理文本数据时可以得到较好的分类结果，所以它被广泛应用于文本分类/垃圾邮件过滤/自然语言处理等场景。

朴素贝叶斯假设了样本的每个特征之间是互相独立、互不影响的，比方说，如果有一个水果是红色的，形状为圆形，并且直径大约为70毫米，那么它就有可能被认为是苹果（具有最高概率的类将会被认为是最有可能的类，这被称为最大后验概率 Maximum A Posteriori），即使上述的这些特征可能会有依赖关系或有其他特征存在，朴素贝叶斯都会认为这些特征都独立地贡献了这个水果是一个苹果的概率，这种假设关系太过于理想，所以这也是朴素贝叶斯的"Naive"之处。

朴素贝叶斯的原名为Naive Bayes Classifier，朴素本身并不是一个正确的翻译，之所以这样翻译是因为朴素贝叶斯虽然Naive，但不代表它的效率会差，相反它的优点正在于实现简单与只需要少量的训练数据，还有另一个原因是它与贝叶斯网络等算法相比，确实是“朴素”了些。

在继续探讨朴素贝叶斯之前，我们先需要理解贝叶斯定理与它的前置理论条件概率与全概率公式。

> 本文作者为[SylvanasSun(sylvanas.sun@gmail.com)][1]，首发于[SylvanasSun’s Blog][2]。
> 原文链接：https://sylvanassun.github.io/2017/12/20/2017-12-20-naive_bayes/
> （转载请务必保留本段声明，并且保留超链接。）

### 条件概率


----------



条件概率（Conditional Probability）是指在事件B发生的情况下，事件A发生的概率，用$P(A|B)$表示，读作在B条件下的A的概率。

![](http://wx2.sinaimg.cn/large/63503acbly1fmnd8qww3lj20d308fdfw.jpg)

在上方的文氏图中，描述了两个事件A和B，与它们的交集`A ∩ B`，代入条件概率公式，可推出事件A发生的概率为$P(A|B) = \frac{P({A}\bigcap{B})}{P(B)}$。

对该公式稍作变换可推得${P({A}\bigcap{B})} = {P(A|B)}{P(B)}$与${P({A}\bigcap{B})} = {P(B|A)}{P(A)}$（`P(B|A)`为在A条件下的B的概率）。

然后根据这个关系可推得${P(A|B)}{P(B)} = {P(B|A)}{P(A)}$。

让我们举个栗子，假设有两个人在扔两个个六面的骰子`D1`与`D2`，我们来预测`D1`与`D2`的向上面的结果的概率。

![](http://wx1.sinaimg.cn/large/63503acbly1fmne20wdcpj20a8078q2w.jpg)

在`Table1`中描述了一个含有36个结果的样本空间，标红处为`D1`的向上面为2的6个结果，概率为$P(D1=2) = \frac{6}{36} = \frac{1}{6}$。

![](http://wx3.sinaimg.cn/large/63503acbly1fmne218l47j209l0723yg.jpg)

`Table2`描述了`D1 + D2 <= 5`的概率，一共10个结果，用条件概率公式表示为${P(D1+D2\leq5)} = \frac{10}{36}$。

![](http://wx2.sinaimg.cn/large/63503acbly1fmne21knj0j20a3071q2w.jpg)

`Table3`描述了满足`Table2`的条件同时也满足`D1 = 2`的结果，它选中了`Table2`中的3个结果，用条件概率公式表示为${P(D1=2 | D1+D2\leq5)} = \frac{3}{10} = 0.3$。

### 全概率公式


----------



全概率公式是将边缘概率与条件概率关联起来的基本规则，它表示了一个结果的总概率，可以通过几个不同的事件来实现。

全概率公式将对一复杂事件的概率求解问题转化为了在不同情况下发生的简单事件的概率的求和问题，公式为$P(B) = {\sum_{i=1}^n}P(A_i)P(B|A_i)$。

假定一个样本空间S，它是两个事件A与C之和，同时事件B与它们两个都有交集，如下图所示：

![](http://wx4.sinaimg.cn/large/63503acbly1fmnd8rfet4j20ka0imdgo.jpg)

那么事件B的概率可以表示为$P(B) = P({B}\bigcap{A}) + P({B}\bigcap{C})$

通过条件概率，可以推断出$P({B}\bigcap{A}) = P(B|A)P(A)$，所以$P(B) = P(B|A)P(A) + P(B|C)P(C)$

这就是全概率公式，即事件B的概率等于事件A与事件C的概率分别乘以B对这两个事件的条件概率之和。

同样举个栗子来应用这个公式，假设有两家工厂生产并对外提供电灯泡，工厂X生产的电灯泡在99%的情况下能够工作超过5000小时，工厂Y生产的电灯泡在95%的情况下能够工作超过5000小时。工厂X在市场的占有率为60%，工厂Y为40%，如何推测出购买的灯泡的工作时间超过5000小时的概率是多少呢？

运用全概率公式，可以得出：
$$
\begin{equation}\begin{split}
Pr(A) &=Pr(A | B_x) \cdot Pr(B_x) + Pr(A|B_y) \cdot Pr(B_y)\\
&= \frac{99}{100} \cdot \frac{6}{10} + \frac{95}{100} \cdot \frac{4}{10}\\
&= \frac{594 + 380}{1000}\\
&= \frac{974}{1000}
\end{split}\end{equation}
$$

 - $Pr(B_x) = \frac{6}{10}$：购买到工厂X制造的电灯泡的概率。

 - $Pr(B_y) = \frac{4}{10}$：购买到工厂y制造的电灯泡的概率。

 - $Pr(A|B_x) = \frac{99}{100}$：工厂x制造的电灯泡工作时间超过5000小时的概率。

 - $Pr(A|B_y) = \frac{95}{100}$：工厂y制造的电灯泡工作时间超过5000小时的概率。

因此，可以得知购买一个工作时间超过5000小时的电灯泡的概率为97.4%。


### 贝叶斯定理


----------



贝叶斯定理最早由英国数学家（同时也是神学家和哲学家）Thomas Bayes（1701-1761）提出，有趣的是他生前并没有发表过什么有关数学的学术文章，就连他最著名的成就贝叶斯定理也是由他的朋友Richard Price从他死后的遗物（笔记）中找到并发表的。

Thomas Bayes在晚年对概率学产生了兴趣，所谓的贝叶斯定理只是他生前为了解决一个逆概率问题（为了证明上帝是否存在，似乎哲学家们都很喜欢这个问题啊）所写的一篇文章。在那个时期，人们已经能够计算出正向概率问题，比方说，有一个袋子中有X个白球，Y个黑球，你伸手进去摸到黑球的概率是多少？这就是一个正向概率问题，而逆概率问题正好反过来，我们事先并不知道袋子中球的比例，而是不断伸手去摸好几个球，然后根据它们的颜色来推测黑球与白球的比例。

贝叶斯定理是关于随机事件A和B的条件概率的一则定理。通常，事件A在事件B（发生）的条件下的概率，与事件B在事件A（发生）的条件下的概率是不一样的，但它们两者之间是有确定的关系的，贝叶斯定理陈述了这个关系。

贝叶斯定理的一个主要应用为贝叶斯推理，它是一种建立在主观判断基础之上的推理方法，也就是说，你只需要先预估一个值，然后再去根据实际结果去不断修正，不需要任何客观因素。这种推理方式需要大量的计算，因此一直遭到其他人的诟病，无法得到广泛的应用，直到计算机的高速发展，并且人们发现很多事情都是无法事先进行客观判断的，因此贝叶斯推理才得以东山再起。

说了这么多理论知识（很多数学理论都像是在说绕口令），让我们来看一看公式吧，其实只需要把我们在上面推导出的条件概率公式继续进行推理，就可以得出贝叶斯公式。

$$P(A|B) = \frac{P(B|A)P(A)}{P(B)}$$

 - $P(A|B)$：在B条件下的事件A的概率，在贝叶斯定理中，条件概率也被称为后验概率，即在事件B发生之后，我们对事件A概率的重新评估。

 - $P(B|A)$：在A条件下的事件B的概率，与上一条同理。

 - $P(A)$与$P(B)$被称为先验概率（也被称为边缘概率），即在事件B发生之前，我们对事件A概率的一个推断（不考虑任何事件B方面的因素），后面同理。

 - $\frac{P(B|A)}{P(B)}$被称为标准相似度，它是一个调整因子，主要是为了保证预测概率更接近真实概率。

 - 根据这些术语，贝叶斯定理表述为： 后验概率 = 标准相似度 * 先验概率。

让我们以著名的假阳性问题为例，假设某种疾病的发病率为0.001（1000个人中会有一个人得病），现有一种试剂在患者确实得病的情况下，有99%的几率呈现为阳性，而在患者没有得病的情况下，它有5%的几率呈现为阳性（也就是假阳性），如有一位病人的检验成果为阳性，那么他的得病概率是多少呢？

代入贝叶斯定理，假定事件A表示为得病的概率（`P(A) = 0.001`），这是我们的先验概率，它是在病人在实际注射试剂（缺乏实验的结果）之前预计的发病率，再假定事件B为试剂结果为阳性的概率，我们需要计算的是条件概率`P(A|B)`，即在事件B条件下的A概率，这就是后验概率，也就是病人在注射试剂之后（得到实验结果）得出的发病率。

由于还有未得病的概率，所以还需要假设事件C为未得病的先验概率（`P(C) = 1 - 0.001 = 0.999`），那么`P(B|C)`后验概率表示的是未得病条件下的试剂结果为阳性的概率，之后再代入全概率公式就可得出最终结果。

$$ 
\begin{equation}\begin{split}
P(A|B)&=\frac{P(B|A)P(A)}{P(B)}\\
&= \frac{P(B|A)P(A)}{P(B|A)P(A) + P(B|C)P(C)}\\
&= \frac{0.99 \times 0.001}{0.99 \times 0.001 + 0.05 \times 0.999}\approx 0.019
\end{split}\end{equation}
$$

最终结果约等于2%，即使一个病人的试剂结果为阳性，他的患病几率也只有2%而已。

### 朴素贝叶斯的概率模型


----------



我们设一个待分类项$X = {f_1,f_2,\cdots,f_n}$，其中每个`f`为`X`的一个特征属性，然后设一个类别集合$C_1,C_2,\cdots,C_m$。

然后需要计算$P(C_1|X),P(C_2|X),\cdots,P(C_m|X)$，我们可以根据一个训练样本集合（已知分类的待分类项集合），然后统计得到在各类别下各个特征属性的条件概率：

$P(f_1|C_1),P(f_2|C_1),\cdots,P(f_n|C_1),\cdots,P(f_1|C_2),P(f_2|C_2),\cdots,P(f_n|C_2),\cdots,P(f_1|C_m),P(f_2|C_m),\cdots,P(f_n|C_m)$

如果$P(C_k|X) = MAX(P(C_1|X),P(C_2|X),\cdots,P(C_m|X))$，则${X}\in{C_k}$（贝叶斯分类其实就是取概率最大的那一个）。

朴素贝叶斯会假设每个特征都是独立的，根据贝叶斯定理可推得：$P(C_i|X) = \frac{P(X|C_i)P(C_i)}{P(X)}$，由于分母对于所有类别为常数，因此只需要将分子最大化即可，又因为各特征是互相独立的，所以最终推得：

![](http://wx4.sinaimg.cn/large/63503acbly1fmrz54fusig20ge01hdfm.gif)

根据上述的公式推导，朴素贝叶斯的流程可如下图所示：

![](http://wx3.sinaimg.cn/large/63503acbly1fmpnscvyl1j20np0sfaeh.jpg)



接下来我们通过一个案例来过一遍上图的流程。

现有一网站想要通过程序自动识别出账号的真实性（将账号分类为真实账号与不真实账号，所谓不真实账号即带有虚假信息或恶意注册的小号）。

-  首先需要确定特征属性和类别，然后获取训练样本。假设一个账号具有三个特征：日志数量/注册天数（`F1`）、好友数量/注册天数（`F2`）、是否使用了真实的头像（True为1，False为0）。

-  该网站使用曾经人工检测过的10000个账号作为训练样本，那么计算每个类别的概率为$P(C_0) = 8900 \div 10000 = 0.89, P(C_1) = 1100 \div 10000 = 0.11$，`C0`为真实账号的类别概率也就是89%，`C1`为虚假账号的类别概率也就是11%。

-  之后需要计算每个类别下的各个特征的条件概率，代入朴素贝叶斯分类器，可得$P(F_1|C)P(F_2|C)P(F_3|C)P(C)$，不过有一个问题是，`F1`与`F2`是连续变量，不适宜按照某个特定值计算概率。解决方法为将连续值转化为离散值，然后计算区间的概率，比如将`F1`分解为`[0,0.05]、[0.05,0.2]、[0.2,+∞]`三个区间，然后计算每个区间的概率即可。

- 已知某一账号的数据如下：$F_1 = 0.1,F_2 = 0.2,F_3 = 0$，推测该账号是真实账号还是虚假账号。在此例中，`F1`为0.1，落在第二个区间内，所以在计算的时候，就使用第二个区间的发生概率。根据训练样本可得出结果为：

$$
\begin{equation}\begin{split}
P(F_1|C_0) = 0.5, P(F_1|C_1) = 0.1\\
P(F_2|C_0) = 0.7, P(F_2|C_1) = 0.2\\
P(F_3|C_0) = 0.2, P(F_3|C_1) = 0.9
\end{split}\end{equation}
$$
	
 - 接下来使用训练后的分类器可得出该账号的真实账号概率与虚假账号概率，然后取最大概率作为它的类别：

$$
\begin{equation}\begin{split}
P(F_1|C_0)P(F_2|C_0)P(F_3|C_0)P(C_0) &= 0.5 \times 0.7 \times 0.2 \times 0.89\\
&= 0.0623
\end{split}\end{equation}
$$
$$
\begin{equation}\begin{split}
P(F_1|C_1)P(F_2|C_1)P(F_3|C_1)P(C_1) &= 0.1 \times 0.2 \times 0.9 \times 0.11\\
&= 0.00198
\end{split}\end{equation}
$$
   
   最终结果为该账号是一个真实账号。


### 朴素贝叶斯的算法模型


----------



在朴素贝叶斯中含有以下三种算法模型：

 - Gaussian Naive Bayes：适合在特征变量具有连续性的时候使用，同时它还假设特征遵从于高斯分布（正态分布）。举个栗子，假设我们有一组人体特征的统计资料，该数据中的特征：身高、体重和脚掌长度等都为连续变量，很明显我们不能采用离散变量的方法来计算概率，由于样本太少，也无法分成区间计算，那么要怎么办呢？解决方法是假设特征项都是正态分布，然后通过样本计算出均值与标准差，这样就得到了正态分布的密度函数，有了密度函数，就可以代入值，进而算出某一点的密度函数的值。

 - MultiNomial Naive Bayes：与Gaussian Naive Bayes相反，多项式模型更适合处理特征是离散变量的情况，该模型会在计算先验概率$P(C_m)$和条件概率$P(F_n|C_m)$时会做一些平滑处理。具体公式为![](http://wx1.sinaimg.cn/large/63503acbly1fmrz54ux0tg20480130nd.gif)，其中`T`为总的样本数，`m`为总类别数，$T_{cm}$即类别为$C_m$的样本个数，`a`是一个平滑值。条件概率的公式为![](http://wx1.sinaimg.cn/large/63503acbly1fmrz55ag8ng205j0130sh.gif)，`n`为特征的个数，`T_cmfn`为类别为`C_m`特征为`F_n`的样本个数。当平滑值`a = 1`与`0 < a < 1`时，被称作为`Laplace`平滑，当`a = 0`时不做平滑。它的思想其实就是对每类别下所有划分的计数加1，这样如果训练样本数量足够大时，就不会对结果产生影响，并且解决了$P(F|C)$的频率为0的现象（某个类别下的某个特征划分没有出现，这会严重影响分类器的质量）。

 - Bernoulli Naive Bayes：Bernoulli适用于在特征属性为二进制的场景下，它对每个特征的取值是基于布尔值的，一个典型例子就是判断单词有没有在文本中出现。


### 朴素贝叶斯的实现


----------



了解了足够多的理论，接下来我们要动手使用python来实现一个Gaussian Naive Bayes，目的是解决皮马人（一个印第安人部落）的糖尿病问题，[样本数据（请从该超链接中获取）][7]是一个csv格式的文件，每个值都是一个数字，该文件描述了从患者的年龄、怀孕次数和验血结果等方面的即时测量数据。每个记录都有一个类别值（一个布尔值，以0或1表示），该值表述了患者是否在五年内得过糖尿病。这是一个在机器学习文献中被大量研究过的数据集，一个比较好的预测精度应该在70%~76%。样本数据的每列含义如下：

```
列1：怀孕次数
列2：在口服葡萄糖耐量试验中，血浆葡萄糖的浓度（2小时）
列3：心脏的舒张压（(mm Hg)）
列4：肱三头肌皮肤褶皱厚度（mm）
列5：二小时内的血清胰岛素（mu U/ml）
列6：体质指数 （(weight in kg/(height in m)^2)）
列7：糖尿病家族作用
列8：年龄
列9：类别布尔值，0为5年没得过糖尿病，1为5年内得过糖尿病
------------------------------------
6,148,72,35,0,33.6,0.627,50,1
1,85,66,29,0,26.6,0.351,31,0
8,183,64,0,0,23.3,0.672,32,1
1,89,66,23,94,28.1,0.167,21,0
0,137,40,35,168,43.1,2.288,33,1
.........
```

首先要做的是读取这个csv文件，并解析成我们可以直接使用的数据结构。由于样本数据文件中没有任何的空行和标记符号，每行都是对应的一行数据，只需要简单地把每一行封装到一个list中即可（返回结果为一个list，它的每一项元素都是包含一行数据的list），注意该文件中的数据都为数字，需要先做类型转换。

```python
import csv

def load_csv_file(filename):
    with open(filename) as f:
        lines = csv.reader(f)
        data_set = list(lines)
    for i in range(len(data_set)):
        data_set[i] = [float(x) for x in data_set[i]]
    return data_set
```

获得了样本数据后，为了评估模型的准确性还需要将它切分为训练数据集（朴素贝叶斯需要使用它来进行预测）与测试数据集。数据在切分过程中是随机选取的，但我们会选择一个比率来控制训练数据集与测试数据集的大小，一般为67%：33%，这是一个比较常见的比率。

```python
import random

def split_data_set(data_set, split_ratio):
    train_size = int(len(data_set) * split_ratio)
    train_set = []
    data_set_copy = list(data_set)
    while len(train_set) < train_size:
        index = random.randrange(len(data_set_copy))
        train_set.append(data_set_copy.pop(index))
    return [train_set, data_set_copy]
```

切分了样本数据后，还要对训练数据集进行更细致的处理，由于Gaussian Naive Bayes假设了每个特征都遵循正态分布，所以需要从训练数据集中抽取出摘要，它包含了均值与标准差，摘要的数量由类别和特征属性的组合数决定，例如，如果有3个类别与7个特征属性，那么就需要对每个特征属性和类别计算出均值和标准差，这就是21个摘要。

在计算训练数据集的摘要之前，我们的第一个任务是要将训练数据集中的特征与类别进行分离，也就是说，构造出一个`key`为类别，值为所属该类别的数据行的散列表。

```python
def separate_by_class(data_set, class_index):
    result = {}
    for i in range(len(data_set)):
        vector = data_set[i]
        class_val = vector[class_index]
        if (class_val not in result):
            result[class_val] = []
        result[class_val].append(vector)
    return result
```

由于已经知道了类别只有一个，而且在每行数据的最后一个，所以只需要将-1传入到class_index参数即可。然后就是计算训练数据集的摘要（每个类别中的每个特征属性的均值与标准差），均值会被作为正态分布的中间值，而标准差则描述了数据的离散程度，在计算概率时，它会被作为正态分布中每个特征属性的期望分布。

标准差就是方差的平方根，只要先求出方差（每个特征值与平均值的差的平方之和的平均值）就可以得出标准差。

```python
import math

def mean(numbers):
    return sum(numbers) / float(len(numbers))

def stdev(numbers):
    avg = mean(numbers)
    variance = sum([pow(x - avg, 2) for x in numbers]) / float(len(numbers))
    return math.sqrt(variance)    
```

有了这些辅助函数，计算摘要就很简单了，具体步骤就是先从训练数据集中构造出`key`为类别的散列表，然后根据类别与每个特征进行计算求出均值与标准差即可。

```python
def summarize(data_set):
    # 使用zip函数将每个元素中的第n个属性封装为一个元组
	# 简单地说，就是把每列（特征）都打包到一个元组中
    summaries = [(mean(feature), stdev(feature)) for feature in zip(*data_set)]
    del summaries[-1] # 最后一行是类别与类别的摘要 所以删除
    return summaries

def summarize_by_class(data_set):
    class_map = separate_by_class(data_set, -1)
    summaries = {}
    for class_val, data in class_map.items():
        summaries[class_val] = summarize(data)
    return summaries
```

数据的处理阶段已经完成了，下面的任务是要去根据训练数据集来进行预测，该阶段需要计算类概率与每个特征与类别的条件概率，然后选出概率最大的类别作为分类结果。关键在于计算条件概率，需要用到正态分布的密度函数，而它所依赖的参数（特征，均值，标准差）我们已经准备好了。

```python
def calculate_probability(x, mean, stdev):
    exponent = math.exp(-(math.pow(x - mean, 2) / (2 * math.pow(stdev, 2))))
    return (1 / (math.sqrt(2 * math.pi) * stdev)) * exponent

def calculate_conditional_probabilities(summaries, input_vector):
    probabilities = {}
    for class_val, class_summaries in summaries.items():
        probabilities[class_val] = 1
        for i in range(len(class_summaries)):
            mean, stdev = class_summaries[i]
			# input_vector是test_set的一行数据，x为该行中的某一特征属性
            x = input_vector[i]
			# 将概率相乘
            probabilities[class_val] *= calculate_probability(x, mean, stdev)
    return probabilities
```

函数`calculate_conditional_probabilities()`返回了一个`key`为类别，值为其概率的散列表，这个散列表记录了每个特征类别的条件概率，之后只需要选出其中最大概率的类别即可。

```python
def predict(summaries, input_vector):
    probabilities = calculate_conditional_probabilities(summaries, input_vector)
    best_label, best_prob = None, -1
    for class_val, probability in probabilities.items():
        if best_label is None or probability > best_prob:
            best_label = class_val
            best_prob = probability
    return best_label
```

最后我们定义一个函数来对测试数据集中的每个数据实例进行预测以预估模型的准确性，该函数返回了一个预测值列表，包含了每个数据实例的预测值。根据这个返回值，就可以对预测结果进行准确性的评估了。

```python
def get_predictions(summaries, test_set):
    predictions = []
    for i in range(len(test_set)):
        result = predict(summaries, test_set[i])
        predictions.append(result)
    return predictions

def get_accuracy(predictions, test_set):
    correct = 0
    for x in range(len(test_set)):
		# 分类结果与测试数据集一致，调整值自增
        if test_set[x][-1] == predictions[x]:
            correct += 1
    return (correct / float(len(test_set))) * 100.0
```

完整代码如下：

```python
import csv, random, math

"""
A simple classifier base on the gaussian naive bayes and
problem of the pima indians diabetes.
(https://archive.ics.uci.edu/ml/datasets/Pima+Indians+Diabetes)
"""

def load_csv_file(filename):
    with open(filename) as f:
        lines = csv.reader(f)
        data_set = list(lines)
    for i in range(len(data_set)):
        data_set[i] = [float(x) for x in data_set[i]]
    return data_set

def split_data_set(data_set, split_ratio):
    train_size = int(len(data_set) * split_ratio)
    train_set = []
    data_set_copy = list(data_set)
    while len(train_set) < train_size:
        index = random.randrange(len(data_set_copy))
        train_set.append(data_set_copy.pop(index))
    return [train_set, data_set_copy]

def separate_by_class(data_set, class_index):
    result = {}
    for i in range(len(data_set)):
        vector = data_set[i]
        class_val = vector[class_index]
        if (class_val not in result):
            result[class_val] = []
        result[class_val].append(vector)
    return result

def mean(numbers):
    return sum(numbers) / float(len(numbers))

def stdev(numbers):
    avg = mean(numbers)
    variance = sum([pow(x - avg, 2) for x in numbers]) / float(len(numbers))
    return math.sqrt(variance)

def summarize(data_set):
    summaries = [(mean(feature), stdev(feature)) for feature in zip(*data_set)]
    del summaries[-1]
    return summaries

def summarize_by_class(data_set):
    class_map = separate_by_class(data_set, -1)
    summaries = {}
    for class_val, data in class_map.items():
        summaries[class_val] = summarize(data)
    return summaries

def calculate_probability(x, mean, stdev):
    exponent = math.exp(-(math.pow(x - mean, 2) / (2 * math.pow(stdev, 2))))
    return (1 / (math.sqrt(2 * math.pi) * stdev)) * exponent

def calculate_conditional_probabilities(summaries, input_vector):
    probabilities = {}
    for class_val, class_summaries in summaries.items():
        probabilities[class_val] = 1
        for i in range(len(class_summaries)):
            mean, stdev = class_summaries[i]
            x = input_vector[i]
            probabilities[class_val] *= calculate_probability(x, mean, stdev)
    return probabilities

def predict(summaries, input_vector):
    probabilities = calculate_conditional_probabilities(summaries, input_vector)
    best_label, best_prob = None, -1
    for class_val, probability in probabilities.items():
        if best_label is None or probability > best_prob:
            best_label = class_val
            best_prob = probability
    return best_label

def get_predictions(summaries, test_set):
    predictions = []
    for i in range(len(test_set)):
        result = predict(summaries, test_set[i])
        predictions.append(result)
    return predictions

def get_accuracy(predictions, test_set):
    correct = 0
    for x in range(len(test_set)):
        if test_set[x][-1] == predictions[x]:
            correct += 1
    return (correct / float(len(test_set))) * 100.0

def main():
    filename = 'pima-indians-diabetes.data.csv'
    split_ratio = 0.67
    data_set = load_csv_file(filename)
    train_set, test_set = split_data_set(data_set, split_ratio)
    print('Split %s rows into train set = %s and test set = %s rows'
                %(len(data_set), len(train_set), len(test_set)))
    # prepare model
    summaries = summarize_by_class(train_set)
    # predict and test
    predictions = get_predictions(summaries, test_set)
    accuracy = get_accuracy(predictions, test_set)
    print('Accuracy: %s' % accuracy)

main()

```


### 参考文献


----------



 - [Bayes' theorem - Wikipedia][3]

 - [Conditional probability - Wikipedia][4]

 - [Law of total probability - Wikipedia][5]

 - [Naive Bayes classifier - Wikipedia][6]

 - [ 6 Easy Steps to Learn Naive Bayes Algorithm (with code in Python)][8]

 - [How the Naive Bayes Classifier works in Machine Learning][9]

 - [Naive Bayes Classifier From Scratch in Python][10]

 - [数学之美番外篇：平凡而又神奇的贝叶斯方法 – 刘未鹏 | Mind Hacks][11]

 - [朴素贝叶斯分类器的应用 - 阮一峰的网络日志][12]

 - [算法杂货铺——分类算法之朴素贝叶斯分类(Naive Bayesian classification) - T2噬菌体 - 博客园][13]


[1]: https://github.com/SylvanasSun
[2]: https://sylvanassun.github.io/
[3]: https://en.wikipedia.org/wiki/Bayes%27_theorem
[4]: https://en.wikipedia.org/wiki/Conditional_probability
[5]: https://en.wikipedia.org/wiki/Law_of_total_probability
[6]: https://en.wikipedia.org/wiki/Naive_Bayes_classifier
[7]: https://archive.ics.uci.edu/ml/machine-learning-databases/pima-indians-diabetes/pima-indians-diabetes.data
[8]: https://www.analyticsvidhya.com/blog/2017/09/naive-bayes-explained/
[9]: https://dataaspirant.com/2017/02/06/naive-bayes-classifier-machine-learning/
[10]: https://machinelearningmastery.com/naive-bayes-classifier-scratch-python/
[11]: http://mindhacks.cn/2008/09/21/the-magical-bayesian-method/
[12]: http://www.ruanyifeng.com/blog/2013/12/naive_bayes_classifier.html
[13]: http://www.cnblogs.com/leoo2sk/archive/2010/09/17/1829190.html