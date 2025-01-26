# Conditional GAN

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