# Conditional GAN

!!! warning "本文正在编写中"

!!! info "声明"

    由于该文章为本人大一时为[“科学与社会”研讨课](https://www.teach.ustc.edu.cn/?attachment_id=17309)学习需要而创作。由于作者才疏学浅，在严谨性上可能存在缺陷。

本篇博客是我“科学与社会”研讨课的一部分。本文作为个人的学习笔记可能不是非常详细，具体请参考[链接](https://keras.io/examples/generative/conditional_gan/)。

## 为什么要 Conditional GAN

- GAN (Generative Adversarial Network，生成式对抗网络) 让我们很方便地使用随机信号生成各种数据（图片、视频、音频）等。
- 但是对于**有分类的数据**，比如生成手写数字 (0～9)，一个普通的 DCGAN (Deep Convolutional GAN) 就没有办法处理了。
- 当然，对于 0 到 9 每个数字都训练一个 GAN 也是可以的，但是 DCGAN 还有一个优势：比如随机生成不同年龄的人像照片，如何指定生成人像的年龄、性别等参数？
- 当然，指定生成人像的年龄和性别也可以这样做：由于生成的图像是隐向量 (latent vector) 决定的，将“老年”图片的隐向量减去“婴儿”图片的隐向量，之后就可以把图片按这个方向移动增大生成图片的“年龄”。
- Conditional GAN 的优势在于：可以支持更多的分类，可以建立参数与生成图像间的联系。

## 什么是 Conditional GAN？

### 什么是 GAN？

具体参考[Goodfellow 等人的论文](https://arxiv.org/abs/1406.2661)。

定义：

- $p_{\boldsymbol z}(\boldsymbol z)$：生成器的随机噪声
- $G(\boldsymbol z;\theta_{g})$：一个由参数 $\theta_{g}$ 定义的生成器，是一个随机向量空间到某个空间（生成数据）的映射。
- $D(\boldsymbol x;\theta_{d})$：一个由参数 $\theta_{d}$ 定义的鉴别器，是一个从某个空间（待鉴别数据）到一个 $(0,1)$ 实数的映射，表示“该数据真实的可能性”。

要求解一个最优化问题：

$$
\min_G \max_D V(D, G) = \mathbb{E}_{\boldsymbol{x} \sim p_{\text{data}}(\boldsymbol{x})}[\log D(\boldsymbol{x})] + \mathbb{E}_{\boldsymbol{z} \sim p_{\boldsymbol{z}}(\boldsymbol{z})}[\log (1 - D(G(\boldsymbol{z})))].
$$

直观地理解，就是找到这样一个 $G$，让最厉害的 $D$ 都很难分辨出分辨出数据是否是 $G$ 伪造的。

- 在实践中，往往采用“训练一步 $G$”——“训练 k 步 $D$”这样的迭代方法。
- 在训练的初期，由于生成的数据杂乱无章，$D$ 很容易分辨出假的数据，此时可以采用最大化 $\log(D(G(\boldsymbol z)))$ 来代替最小化 $\log(1-D(G(\boldsymbol z)))$。

论文中作者证明了这个最优化问题取得最小值当且仅当生成数据的分布等于真实数据的分布，而且这个函数关于生成器的参数是凸的，所以优化算法可以得到比较好的结果。

### 正题

具体参考[Mirza 的论文](https://arxiv.org/abs/1411.1784)。

- 我们想让 GAN 生成指定分类（或特征）的数据，也就是说，这个生成器**不仅**要接受一个**随机向量**，还要接受一个**与分类（特征）有关的向量**。
- 比如说，我需要指定生成衬衫、卫衣、长裤的图片，这个生成器不仅要接受随机向量，还要接受是不是衬衫，是不是卫衣，是不是长裤的向量。鉴别器也接受到了是不是衬衫，是不是卫衣，是不是长裤的这个向量。
- 所以，在 GAN 中，判别器是这样的：“这张图片好像是真的”
- 而在 Condition GAN 中：“那人告诉我这张图片是衬衫，但它不像，所以是假的”

形式化地，我们要改变一些定义：

- $G(\boldsymbol z|\boldsymbol y;\theta_{g})$：一个由参数 $\theta_{g}$ 定义的生成器，$\boldsymbol z$ 是随机向量，$\boldsymbol y$ 是附加信息。
- $D(\boldsymbol x|\boldsymbol y;\theta_{d})$：一个由参数 $\theta_{d}$ 定义的鉴别器，$\boldsymbol x$ 是待鉴别数据，$\boldsymbol y$ 是附加信息。

类似的，我们要最优化的是：


$$
\min_G \max_D V(D, G) = \mathbb{E}_{\boldsymbol{x} \sim p_{\text{data}}(\boldsymbol{x})}[\log D(\boldsymbol{x})] + \mathbb{E}_{\boldsymbol{z} \sim p_{\boldsymbol{z}}(\boldsymbol{z})}[\log (1 - D(G(\boldsymbol{z})))].
$$

### 遇到的一些问题

我按照 [Keras 网站上的教程](https://keras.io/examples/generative/conditional_gan/)书写了一个 Conditional GAN 用于模仿 [SPOTS-10](https://github.com/Amotica/SPOTS-10) 的图片，用于“科学与社会”研讨课的课题，然而训练的效果却[不尽如人意](https://github.com/xinchengo/spots/blob/0c745ad9f465861911691c4f846fd2970c0a42d7/conditional_gan.ipynb)——鉴别器的损失函数开始收敛到 0，生成器的损失函数开始发散到正无穷。

我尝试用“暂时锁住鉴别器”的方法训练若干轮的方法，让生成器的值恢复正常，之后在进行了几百轮的训练，但是结果就是生成了一批灰蒙蒙、模糊不清的图像，损失函数是收敛的，但收敛的值并不是一个最优解。

Update：

- 由于 [SPOTS-10](https://github.com/Amotica/SPOTS-10) 的数据是 $32\times 32$，而 [MNIST](https://en.wikipedia.org/wiki/MNIST_database) 的数据是 $28\times 28$ 的，所以在把适用 MNIST 的代码修改成 SPOTS-10 的版本时，我改动了鉴别器和生成器的结构。在该次训练中，损失函数发散了，
- 之后我又尝试了把 $32\times 32$ 的图片裁剪成 $28\times 28$，再在**不改动结构**的情况下直接训练。这次训练中，损失函数收敛了，但是没有收敛到最优解。

## Wasserstein GAN

以下是我读[Arjovsky 等人的原论文](
具体读[这篇论文](https://arxiv.org/abs/1701.07875))的笔记

### 引入

根据 [Google 的教程](https://developers.google.com/machine-learning/gan/problems)，GAN 的训练常常遇到三类问题：

- 梯度消失 (vanishing gradients)：生成器不知道怎么调整——当鉴别器太好，生成器的参数不管往什么方向调整一点点，都不会改变它在鉴别器眼中的真实程度时，会发生这种情况。
- 模式坍缩 (mode collapse)：生成器只知道出几张牌，而不是生成随机图片；
- 无法收敛 (failure to converge)：损失函数发散

Wasserstein GAN 旨在解决的就是**梯度消失**和**模式坍缩**这两个问题。

### 概率密度方法与参数化方法

!!! info "以下部分内容使用 [Deepseek](https://chat.deepseek.com/) 翻译"

    由于本人对一些数学名词的正确翻译并不熟悉，以下部分内容使用 [DeepSeek](https://chat.deepseek.com/) 翻译。可能存在一些名词上的不严谨。

!!! warning "以下内容有待补充"

    现在以下内容基本上从 DeepSeek 翻译中摘录而来，尚未增添适当的注解和补充。这些工作后续将进行。

传统的**无监督学习**方法往往是去学习一个未知的概率密度 (probability density)。具体来说，给定一组真实数据的样本 $\{x^{(i)}\}^m_{i=1}$，传统方法会定义一个参数化的密度族 (parametric family of densities) $(P_{\theta})_{\theta\in\mathbb R^d}$，并寻找能最大化数据似然 (likelihood) 的参数：

$$
\max_{\theta\in\mathbb R^d}\frac{1}{m}\sum_{i=1}^m\log P_{\theta}(x^{(i)})
$$

如果真实数据分布 $\mathbb P_r$ 存在密度，而 $\mathbb P_{\theta}$ 是参数化密度 $\mathbb P_{\theta}$ 对应的分布，那么在大样本的情况下，这等价于最小化 **Kullback-Leibler 散度** (Kullback-Leibler divergence) $KL(\mathbb P_r||\mathbb P_{\theta})$。

### 前置知识



### 一些距离的定义

先介绍一些符号和定义。记 $\mathcal X$ 是一个**紧致度量空间** (compact metric set)，$\Sigma$ 表示 $\mathcal X$ 上所有 Borel 子集的集合，设 $\operatorname{Prob}(\mathcal X)$ 表示定义在 $\mathcal X$ 上的概率测度空间 (space of probability measures)。我们现在可以定义两个分布 $\mathbb P_r,\mathbb P_g\in\operatorname{Prob}(\mathcal X)$ 之间的基本距离和散度：

1. **总变差距离** (**Total Variation**, TV)：
    
    $$
    \delta(\mathbb P_r,\mathbb P_g)=\sup_{A\in\Sigma}|\mathbb P_r(A)-\mathbb P_g(A)|
    $$

2. **Kullback-Leibler 散度** (KL)：
   
    $$
    KL(\mathbb P_r ||\mathbb P_g) = \int\log\left(\frac{P_r(x)}{P_g(x)}\right)\mathrm d\mu(x)
    $$

    其中假设 $\mathbb P_r$ 和 $\mathbb P_g$ 都是关于 $\mathcal X$ 上定义的某个测度 $\mu$ 绝对连续的，因此它们存在密度函数。

    KL 散度是著名的非对称散度，并且当存在某些点使得 $P_g(x)=0$ 且 $P_r(x)>0$ 时，KL 散度可能是无穷大。

3. **Jensen-Shannon 散度** (JS)：

    $$
    \begin{gather}
    JS(\mathbb P_r,\mathbb P_g)=KL(\mathbb P_r ||\mathbb P_m) + KL(\mathbb P_g || \mathbb P_m) \\
    \mathbb P_m = (\mathbb P_r + \mathbb P_g) / 2
    \end{gather}
    $$

    这个散度是对称的，并且总是有定义的，因为我们可以选择 $\mu = \mathbb P_m$。

4. **Earth-Mover 距离** (EM) 或 **Wasserstein-1 距离**：

    $$
    W(\mathbb P_r,\mathbb P_g) = \inf_{\gamma\in\Pi(\mathbb P_r,\mathbb P_g)}\mathbb E_{(x,y)\sim\gamma}\left[||x-y||\right]
    $$

    其中 $\Pi(\mathbb P_r, \mathbb P_g)$ 表示所有联合分布 $\gamma(x, y)$ 的集合，其边缘分布分别为 $\mathbb P_r$ 和 $\mathbb P_g$，必须从 $x$ 到 $y$ 传输的“质量”。EM 距离是最优传输计划的“臣本”。
