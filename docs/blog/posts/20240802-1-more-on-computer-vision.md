---
draft: true
date: 2024-08-02
categories:
    - 笔记
tags:
    - 机器学习
    - 计算机视觉
---

# 计算机视觉的更多内容

这是 *Deep Learning for Coders with fastai and PyTorch AI Applications Without a PhD (Jeremy Howard, Sylvain Gugger)* 第六章 *Other Computer Vision Problems* 和第七章 *Training a State-of-the-Art Model* 的学习笔记。

第六章介绍了多分类鉴别器和图片回归 (regression) 模型，第七章更加深入地介绍了模型的优化和调参。

（非常简略）

<!-- more -->
## 总结

- 多分类鉴别器的损失函数：对每个标签的置信度分别做交叉熵损失 (cross-entropy loss)。
- 数据的标准化 (normalization)：对模型效果影响不大，但是所有预训练的模型都进行了数据的标准化，所以，迁移学习时标准化是必须的。
- 渐进式缩放 (progressive resizing)：逐渐增大用于训练的图像，这种策略有助于提高训练速率，并提升模型的通用性。
- 测试时增强 (test time augmentation, TTA)：在测试时使用数据增强（比如随机裁剪）策略对测试集的每张图像创建多个副本，然后对这些预测值取平均值或者最大值，这种策略有助于增强测试的准确性（比如随机裁剪可能会损伤测试集中的关键信息）
- 混合 (mixup)：对训练集的图片组取加权平均值（预测值也取加权平均值）进行训练的策略，这种策略可以增强训练的准确率（缺点是减慢训练速度），并且防止过拟合，尤其适合训练集较小的情况。
- 标签平滑 (label smoothing)：防止模型过于自信地预测标签，抑制过拟合。

