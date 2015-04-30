---
layout: post
title: "CaffeNet 的分析"
date: 2014-12-25 14:00
---

* CaffeNet 训练图
![](/images/caffenet_train_val.svg)

卷积 (Convolution)
-----------------

自然图像有其固有特性，也就是说，图像的一部分的统计特性与其他部分是一样的。这也意味着我们在这一部分学习的特征也能用在另一部分上，所以对于这个图像上的所有位置，我们都能使用同样的学习特征。

![](/images/Convolution_schematic.gif)

池化 (Pooling)
-------------

![](/images/Pooling_schematic.gif)

ReLU Nonlinearity (Rectified Linear Units)
------------------------------------------
非线性激活函数 RELU

局部反应正常化 (Local Response Normalization)
------------------------------------------

Dropout
-------

让网络层不参与反向传导，不利正向传导

Softmax回归
----------

DATA 数据输入
------------

训练使用 [ImageNet](http://image-net.org) 的标注数据集。ImageNet is a dataset of over 15 million labeled high-resolution images belonging to roughly 22,000 categories.

CaffeNet 只选取其中的 1000 个分类

图片经过预处理成 256x256 的小图，存入 leveldb or lmdb

    path/to/images/1.xxx  label
    path/to/images/2.xxx  label
