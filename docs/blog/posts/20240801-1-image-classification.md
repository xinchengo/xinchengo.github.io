---
draft: true
date: 2024-08-01
categories:
    - 笔记
tags:
    - 机器学习
    - 计算机视觉
---

# 计算机视觉之图像分类

这是 *Deep Learning for Coders with fastai and PyTorch AI Applications Without a PhD (Jeremy Howard, Sylvain Gugger)* 第四章 *Under the Hood: Training a Digit Classifier* 和第五章 *Image Classification* 的学习笔记。

第四章中我们需要揭开 `fastai` 库的神秘面纱，用 PyTorch 动手实现一个数字分类器。第五章中，我们需要了解图像识别的更多细节。

<!-- more -->

## 问题

### 检查数据

对于 `DataLoaders` 类，我们可以使用 `show_batch` 方法检查若干个数据：

```py
dls.show_batch(nrows=1, ncols=3)
```

当构建 `DataBlock` 时出现问题，我们还可以用 `summary` 方法检查 `DataBlock` 的构建过程，比如，如果我们在 

```py
dls = pets.dataloaders(path/"images")
```

的执行中遇到了问题，我们可以简单地把 `dataloaders` 替换成 `summary` 检查 `dataloaders` 构建的整个过程：

```py
pets.summary(path/"images")
```


### Softmax 函数和交叉熵误差

书中讲得有些不知所云，更详细的解读可以参考 [Stanford Course](https://cs231n.github.io/linear-classify/)。

对于一个向量 $z=(z_1, \dots, z_K)\in \mathbb R^K$，我们定义 Softmax 函数 $\sigma(z)$ 为：

$$
\sigma(z)_i = {e^{z_i} \over\sum_{j=1}^{K}e^{z_j}}
$$

交叉熵误差 (cross-entropy loss) $L_i$ 则是 $\sigma(z)_i$ 的负对数（没错，就是 pH 值中的那个 p 的意思），即 $L_i=-\log \sigma (z)_i$

这两个式子到底有什么意义呢，从形式上来看，Softmax 函数以一个向量作为输入，在神经网络中，它们就是最后一层节点的输出值，我们需要做的事情就是将这些输出值转化为的置信度。我们要求所有分类置信度加起来等于 $1$，容易看出，Softmax 函数是符合这个要求的。

因此，从**概率**的角度上出发，我们可以认为神经网络输出的是没有经过规范的概率对数。在 Softmax 函数中，求指数操作把概率对数转化“概率”，然后除以各项的和让结果的和为 $1$，得到每一项的“概率”。

我们还可以从信息论的角度理解这个函数。先介绍信息熵的概念，相信不少信息学竞赛的选手听说过甚至深入学习过：

**信息熵** (entropy)：对于一个离散随机变量 $X\in \mathcal X$，$X$ 的**概率质量函数**为 $p(X)$，则信息熵定义为：

$$
\mathrm H(X) := -\sum_{x \in \mathcal{X}} p(x) \log p(x)
$$

它的实际意义是，这个随机变量期望提供的信息量。比如，一枚硬币，$\mathcal X=\{1,2\},p(1)=p(0)={1\over 2}$，取 $\log$ 的底为 $2$，则它的信息熵为 $1$ bit，一枚骰子，$\mathcal X=\{1,2,3,4,5,6\}$，则它的信息熵为 $\log_2{3}+1$ bit，显然掷一枚骰子的信息量大于抛两枚硬币的信息量。具体的解释见下文的分析。

从直觉上可以判断，一个事件的信息量取决于这个事件是否令人惊奇。一个罕见的事件传递的信息量很大，而一个常见的事件传递的信息量很小。香农根据这一个现象引入了**自信息** (information content)。自信息是一个符合以下公理的量：

1. 一个概率为 $1$ 的事件的自信息为 $0$；
2. 事件的概率越小，自信息越大；
3. 两个独立事件的信息等于它们各自的自信息之和。（**非常关键，决定公式结构**）

根据以上三个公理，可以得出**自信息的公式**：

$$
\mathrm{I}(x) := - \log_b{\left[\Pr{\left(x\right)}\right]} = -\log_b{\left(P\right)}
$$

其中 $b$ 可以是任意大于 $1$ 的常量。我们下面来验证自信息符合上述公理 3，设 $X,Y$ 是两个相互独立的随机变量，它们的**概率质量函数**分别为 $p_X(x)$ 和 $p_Y(y)$，则它们的**联合概率质量函数**为:

$$
p_{X, Y}\!\left(x, y\right) = \Pr(X = x,\, Y = y) 
 = p_X\!(x)\,p_Y\!(y)
$$

因为 $X$ 和 $Y$ 相互独立，结果 $(X,Y)=(x,y)$ 的自信息为：

$$
\begin{aligned}
\operatorname{I}_{X,Y}(x, y) &= -\log_2\left[p_{X,Y}(x, y)\right]
 = -\log_2 \left[p_X\!(x)p_Y\!(y)\right] \\[5pt]
 &= -\log_2 \left[p_X{(x)}\right] -\log_2 \left[p_Y{(y)}\right] \\[5pt]
 &= \operatorname{I}_X(x) + \operatorname{I}_Y(y)
\end{aligned}
$$

观察到自信息比概率降低了一个阶，所以两个概率的相乘就可以转化为两个自信息的相加。这种定义符合公理 $3$，所以它的期望是有**实际意义**的。从两个公式的比较可以推出，信息熵的公式就是对自信息的公式求期望，也就是说，信息熵就是这个随机变量产生自信息的**期望值**。

信息熵的意义是，对于信息熵很低的文件，我们可以构造一种编码（比如**哈夫曼编码**，做过[合并果子](https://www.luogu.com.cn/problem/P1090)的人可能听说过这个概念）让文件大小变小。比如，如果某个长片段在一个文件里经常出现，我们可以用较短的编码代表这个片段，而用较长的编码代表较短但罕见的片段。这时又出现了一个新的问题：如果我们为概率分布 $q$ 设计了一套哈夫曼编码，但是真实的信息是按概率分布 $p$ 出现的，每个事件平均要用多少个比特呢？这时我们就需要引入交叉熵的概念：

**交叉熵** (cross-entropy)：设 $p,q$ 为两个离散概率分布的概率质量函数，我们定义 $q$ 关于 $p$ 的交叉熵为：

$$
H(p, q) = - \sum_{x\in \mathcal X}p(x)\log q(x)
$$

观察上式的形式，它与信息熵公式唯一的区别就是 $\log$ 内的内容是 $q$ 而不是 $p$。现在仍然回到上述的例子，假设我们已经对概率分布 $q$ 设计了一套“量身定制”的哈夫曼编码，如果 $x$ 出现的概率为 $q(x)$，理想的哈夫曼编码中 $x$ 应该用 $-\log q(x)$ bit 表示，也就是说，理想哈夫曼编码的长度应当等同于它的信息量（虽然一般做不到）。这时，因为真实的信息是按照概率分布 $p$ 出现的，我们按照 $p$ 对这个哈夫曼编码的长度求期望就可以了。

容易发现，假设 $p$ 是未知的常量，我们对 $q$ 猜测得越准，平均编码一个事件所需的比特就越少。所以，交叉熵其实也反映了两个概率分布的**接近程度**。回到 Softmax 分类器的问题上，Softmax 的误差函数（交叉熵误差函数）具有以下形式：

$$
\begin{aligned}
L_i &= -\log\left(e^{f_{y_i}}\over\sum_je^{f_j}\right) \\
&=-\sum_k\mathbb{1}(k=y_i)\log\left(e^{f_k}\over\sum_je^{f_j}\right)
\end{aligned}
$$

上式中 $\mathbb{1}$ 的意思就是，若内部条件为真，则值为 $1$，否则为 $0$。根据上式形式可以看出：交叉熵误差函数衡量的是，真实分布的概率质量函数（$p=[0,\dots,1,\dots,0]$，只在正确的分类上为 $1$）和神经网络推断出的概率质量函数的差异。这个函数的值越小，说明预测越准确。

### 检查训练效果——混淆矩阵

混淆矩阵 (confusion matrix)：

```py
interp = ClassificationInterpretation.from_learner(learn)
interp.plot_confusion_matrix(figsize=(12,12), dpi=60)
```

若要输出混淆矩阵中最混淆的项目，可以使用如下代码：

```py
interp.most_confused(min_val=5)
```

### 学习率的调参

学习率的调参——学习率查找器 (Learning Rate Finder)。

学习率查找器可以画出当学习率逐步增加时，单次迭代后误差 (loss) 的图像，并给出这个图像下降最快的点，发生转折的点等特殊点。我们可以借助这个图像找到（可能）的最优学习率。学习率查找器可以用如下代码调用：

```py
learn = cnn_learner(dls, resnet34, metrics=error_rate)
lrs = learn.lr_find(suggest_funcs=(minimum, steep)) # 注意书上的代码较老，截至本文创作时间，API 已经发生变化。
```

![Learning Rate Finder](../../images/20240801-lr-finder-light.png#only-light)
![Learning Rate Finder](../../images/20240801-lr-finder-dark.png#only-dark)

上述代码中输出的 `lrs.minimum` 是误差函数最小时学习率的**十分之一**，`lrs.steep` 是误差函数图像上最陡处的学习率，我们可以参考这两个值设置学习率。

### 迁移学习

迁移学习 (transfer learning) 是指借助一个已有的模型训练新模型的过程。

对于视觉模型来说，较浅层次的节点一般捕捉输入图像中浅层次的特征（比如某些图案，颜色的渐变），而较深层次的节点则表示图片中细节性的特征（比如是否是一只猫，或者是否是一辆车）。由于现实生活中的图像数据都有着相似的特点，我们可以保留预训练模型中 (pretrained model) 浅层次的参数，调节深层次的参数，从而达到用有限的数据集和算力训练视觉模型的目的。

冻结 (freeze) 指在迁移学习的初期，为了防止破坏预训练的模型，把部分（或者全部）参数锁定，只训练最后一层随机添加的参数的过程。这其实等价于将预训练的模型（除去最后一层）当作输入信息的一个“预处理器”，然后以“预处理器”的输出作为输入训练一个线性分类器。

解冻 (unfreezing)：当我们训练好了最后一层的参数，我们需要将先前冻结的参数取消锁定以增强模型的通用性。

FastAI 库中的 `fine_tune` 方法默认会冻结 (freeze) 预训练的模型训练最后一层一个周期，然后解冻，再训练指定个周期。如果要人为调节冻结的周期数，可以使用下列过程：

- `learn.fit_one_cycle(3, 3e-3)`：冻结训练三个周期
- `learn.unfreeze()`：解冻
- `learn.lr_find()`：老的学习率不再管用，需要寻找新的学习率
- `learn.fit_one_cycle(6, lr_max=1e-5)`：解冻后继续训练

上述代码可以用 `learn.fine_tune(6, freeze_epochs=3)` 一句代替

区别性学习率 (discriminative learning rates) 是另一个防止破坏预训练模型的方法。丛直觉上可以推断，浅层的参数需要的学习率要低于深层的参数。这个结果也得到了[实验上的证实](http://arxiv.org/pdf/1411.1792)。

FastAI 中默认使用区别性学习率，但也可以人为调节，代码如下：

```py
learn.fit_one_cycle(12, lr_max=slice(1e-6, 1e-4))
```

## 总结

- 一般神经网络算法的流程：初始化——迭代优化——输出解
- 基线 (baseline)：作为对照的模型，一个复杂的模型准确率只有明显高于基线才有意义。
- 交叉熵误差 (cross-entropy loss)：衡量两个概率分布的区别大小
- 学习率的调参——最陡处或者最低点的十分之一
- 周期数的调参——刚好就行
- 模型的选择——从小的开始试，如果大的模型更好就改用大的