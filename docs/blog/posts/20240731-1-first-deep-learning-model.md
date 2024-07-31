---
draft: false
date: 2024-07-31
categories:
    - 笔记
tags:
    - 机器学习
---

# 你的第一个机器学习应用

这是 *Deep Learning for Coders with fastai and PyTorch AI Applications Without a PhD (Jeremy Howard, Sylvain Gugger)* 第二章 *From Model to Production* 的学习笔记。

书的本章中我们需要实现一个棕熊，黑熊，泰迪熊的分类器。示例代码可能不完整，可参考原书或[官方代码仓库](https://github.com/fastai/fastbook)。

<!-- more -->

## 问题

### 关于数据集的采集

Bing Image Search 这个 API 需要注册 Azure 账户（需要单独注册，如果使用普通的 Microsoft 账户，进入 Azure 门户会提示有问题），而注册 Azure 账户需要绑定银行卡。由于操作过于麻烦，我选择使用 DuckDuckGo 的图片搜索 API。

DuckDuckGo API 的使用在这本书配套的网络教程第一课 [Is it a bird?](https://course.fast.ai/Lessons/lesson1.html) 中有提及。但是，截止本文创作（2024 年 7 月），Python 库 `duckduckgo_search` 的使用方法已经发生了变化，具体 API 见[该 Python 库的 Github 官方仓库](https://github.com/deedy5/duckduckgo_search)，下面就是一个我采集数据集的代码示例：

```py
bear_types = 'grizzly', 'black', 'teddy'
path = Path('bears')

if not path.exists():
    path.mkdir()
    for o in bear_types:
        dest = (path / o)
        dest.mkdir(exist_ok = True)
        
        responce = DDGS().images(keywords=f"{o} bear", max_results=400)
        results = []
        for item in responce:
            results.append(item['image'])
            
        download_images(dest, urls=results)
```

### 关于数据集的清理

我的 Jupyter Notebook 本来是在 Kaggle 中运行的，但是，Kaggle 运行这段代码时总是出现莫名其妙的问题：

```py
cleaner = ImageClassifierCleaner(learn)
cleaner
```

在 Kaggle 上，这个清理器有时可以正常运行，有时候则不能（两个复选框都正常加载，但图片迟迟没有显示），我怀疑是网络或者是浏览器（Safari）的问题。于是……我只好在本地的 Jupyter Notebook 环境中运行了。本地图片的加载就没有任何卡的时候了，但是，清理数据时仍然有几点需要注意：

- 清理器的界面是上方两排选择框，下方一行待处理的图片。
- 先不要动选择框，把待处理的图片操作勾选完成后，**立马**执行以下的代码：
    ```py
    for idx in cleaner.delete(): cleaner.fns[idx].unlink()
    for idx, cat in cleaner.change(): shutil.move(str(cleaner.fns[idx]), path/cat)
    ```

    这段代码的意义是：处理当前勾选的照片。注意这段代码**只能执行一次**。

- 执行上述代码后，再可以动选择框处理其他的图片组，顺序非常重要，因为你每次动选择框，`cleaner` 的状态就会被重置。

### 关于训练效果

经过反复的的训练和数据清理，我去除了不下十张的奇怪的照片，调整了十几张照片的分类，但是我模型的准确率还是达不到书中示例的水平，我最终的混淆矩阵 (confusion matrix) 如下图所示：

![Confusion matrix](../../images/20240731-confusion-light.png#only-light)
![Confusion matrix](../../images/20240731-confusion-dark.png#only-dark)

图中显示我的模型把 5 只棕熊误认作黑熊，这个错误的来源可能来源于，我对棕熊，黑熊的了解太少，在清理数据的时候把少量棕熊、黑熊的图片移到了错误的分类，模型本身不存在什么问题。

但是，书的作者也不是熊的分类专家，为什么书中的训练效果比我好那么多呢？我猜可能还是搜索引擎的问题，但无论如何，这个示例教会了我们如何在网上搜集数据用开源的机器学习库训练自己的模型，其教育意义是无法掩盖了。一些小小的遗憾，也只能留待以后自己水平更强后再夷平了。

## 总结

- 制作机器学习应用的流程：

    ```mermaid
    flowchart LR
        A(准备数据)
        B(选择模型)
        C(训练模型)
        D(清理数据)
        E(下载模型)
        F(发布应用)
        A --> B
        B --> C
        C --> D
        D --> C
        C --> E
        E --> F
    ```

    其中，**清理数据**和**准备数据**两个步骤是非常重要的。大多数机器学习的问题都可以用几个特定的通用模型解决，训练模型的时候也可以使用很多预训练的模型 (pretrained model) 进行**迁移学习** (transfer learning) 节约时间。但是准备数据没有通用的方法，视具体问题解决方法不尽相同，是机器学习的重点；清理数据是将不符合要求的数据移出数据集，工作量大，对方法的要求很高，是机器学习的难点。
