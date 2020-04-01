---
layout: post
title: 'FaceBoxes的思考'
date: 2019-10-01
author: Jekyll
color: rgb(255,210,32)
cover: 'http://on2171g4d.bkt.clouddn.com/jekyll-banner.png'
tags: DeepLearning
---

当只有CPU可以运行网络的时候怎么办？网络FaceBoxes宣传是A CPU Real-time Face Detector with High Accuracy，其实是前期部分借鉴AlexNet后期借鉴GoogleNet和SSD的网络。

我们可以回头看看几年前的状况，那个时候没有完善的GPU训练框架，但是AlexNet依旧告诉世界，深度学习在图像分类中可行。可是AlexNet的参数实在太多，才会走上使用GPU的道路。但是AlexNet的参数主要集中在后半部分，前部分依旧值得借鉴。

# 结构
如果拆分FaceBoxes可以看到AlexNet的前期结构。

![caption](https://github.com/Hummmm/Hummmm.github.io/blob/master/_posts/2019-10-01-Faceboxes/faceboxes.png?raw=true)
> Faceboxes网络结构图

## 前部分RDCL

Rapidly Digested Convolutional Layers
- 缩小输入的空间大小。卷积层与Pooling层交替，从而每隔一个stage，feature map的分辨率就减半。
- 当分辨率（HW）减少时，要增加通道数，保证有效特征数量。
- 选择合适的kernel size，大致层数越深，卷积核尺寸越小。
- 使用了CRelu来代替AlexNet的Relu，但是本质都大大加快了收敛速度，有效解决了数学上看似完美的非线性激活函数sigmoid和tanh的易饱和的问题。

## 后部分MSCL

![caption](https://github.com/Hummmm/Hummmm.github.io/blob/master/_posts/2019-10-01-Faceboxes/nin.jpg?raw=true)

> NIN里的1X1 Conv结构

Multiple Scale Convolutional Layers是GoogleNet和SSD的结合。但是FC层占据绝对多数的模型参数这个特性使得AlexNet网络过于庞大，现在一般尽量避免全连接层，改用全卷积+GlobalAvgPooling (Network in Network(NIN)最早提出了用Global Average Pooling（GAP）层来代替全连接层的方法)。
- 低计算量：而GoogleNet的Inception概念里的Bottleneck结构（借鉴NON，即1X1 Conv结构使得计算量变十分之一，同时在相同尺寸的感受野中叠加更多的卷积，能提取到更丰富的特征）解决了FC结构参数过多的问题。
- 网络自主选择：我们可以用而到底是什么样的Conv好还是Pooling层好，不想选的时候就全上，所以FaceBoxes采用了Inception这个结构，内部使用不同大小的卷积核，可以捕获到更多的尺度信息。
- 多尺度：AlexNet里的data augmentation，Local Response Normalization[<sup>1</sup>](#refer-anchor-1)，Overlapping Pooling[<sup>2</sup>](#refer-anchor-2)，dropout[<sup>3</sup>](#refer-anchor-3)都是为了防止过拟合，但是后续的研究发现这些的作用并没有利用前后层信息来得有效，比如ResNet。首创Inception的GoogleNet原本为了解决过拟合提出分支的概念（Softmax层）,保证浅层网络也可以学习到分类信息。但这个思维其实有点像SSD，FaceBoxes索性直接把分支结构改成了Multi-scale design along the dimension of network depth.

# 结论

网络FaceBoxes其实是前期部分借鉴AlexNet后期借鉴GoogleNet和SSD的网络。

# 思考

个人觉得分支的结构还是可以保留 ，但是功能已经不是用来承担过拟合的作用，而是更像AlexNet里的分组卷积（group=2），相当于直接把模型等分为两个独立模型，在两个GPU上单独训练不同任务，在第三个卷积层和最后的三个全连接层做特征交流。

![caption](https://github.com/Hummmm/Hummmm.github.io/blob/master/_posts/2019-10-01-Faceboxes/alex.jpg?raw=true)
> 上面48个是在GPU1上学习的，下面48个是在GPU2上学习的，GPU1上学习的都是与颜色无关的特征，GOU2上学习的大部分都是与颜色相关的特征。

# 备注

<div id="refer-anchor-1"></div>
- [1] 使用LRN对局部的特征进行归一化，结果作为ReLU激活函数的输入能有效降低错误率
<div id="refer-anchor-2"></div>
- [2] 重叠最大池化（overlapping max pooling），即池化范围z与步长s存在关系$z>s$（如$S_{max}$中核尺度为$3\times3/2$），避免平均池化（average pooling）的平均效应
<div id="refer-anchor-3"></div>
- [3] 使用随机丢弃技术（dropout）选择性地忽略训练中的单个神经元，避免模型的过拟合
