---
layout: post
title: 'Blend Shape of 3D human body'
date: 2018-08-01
author: Jekyll
color: rgb(255,210,32)
cover: 'http://on2171g4d.bkt.clouddn.com/jekyll-banner.png'
tags: Graphics
---


shape是一个广泛的概念，分成 identity-dependent shape 和 pose-dependent shape， 其中用于动作捕捉的skeletal pose 也只是pose中的一种。其中identity-dependent shape由静态参数β决定，而pose-dependent shape由旋转参数 θ和运动序列参数 δ决定。  以上所有参数和body之间的映射关系是：
         M (β , θ , δ ，· ·  ) → R3

此外，shape还有一部分由训练数据得到的常数参数，这些参数可以用来得到一个template的body，这个body包含三部分：继承关系的骨骼结构，mesh，及两者之间的变形关系. Body的 mesh由triangular faces，vertices 和 edges构成.

![caption](https://github.com/Hummmm/Hummmm.github.io/blob/master/_posts/2018-08-01-Blend-Shape/1.png?raw=true)
> Blend Shape网络结构图

# LBS

linear blend skinning (LBS) 是一种利用皮肤的权重和骨骼单位的运动来计算身体变形的一种方式. template vertex displacement scheme是利用多个pose的扫描出来的body进行插值(interpolating)

For a subject, the weight w ib represents the influence of the b th bone on the i th vertex. Each vertex is associated with no more than | B | nearby bones. If v r i is the position of the i th vertex in the rest pose, and each pose-dependent deformation Q b ( θ ) transforms the b th bone in the m th mesh pose M m ,weights W i are the learned parameters.then the deformed i th vertex v m is given by 

![caption](https://github.com/Hummmm/Hummmm.github.io/blob/master/_posts/2018-08-01-Blend-Shape/2.png?raw=true)

# PCA
用主成分分析(principal component analysis )可以在|M|个不同的扫描mesh构建出shape modeling.Using principal component analysis (PCA) on the displacements of the template vertices allows the shape space to be modeled. The vertices of the registered meshes are concatenated into | M | column vectors Sm , m = 1 , .  , | M | of size 3 ×  | V |. Uj is j th component vector for all components U , while βm is corresponding identity-dependent shape parameter vector of shape components βm .

这里常数U是PCA的特征，其中U1是影响最大的变量，Um是影响最小的变量.因此把U后面几个变量仍了也不会影响太多结果，并且利用线性回归可以把身高等特征与U构建映射关系.

![caption](https://github.com/Hummmm/Hummmm.github.io/blob/master/_posts/2018-08-01-Blend-Shape/3.1.png?raw=true)
![caption](https://github.com/Hummmm/Hummmm.github.io/blob/master/_posts/2018-08-01-Blend-Shape/3.2.png?raw=true)

# SCAPE
SCAPE是第一个把pose和shape进行整合的参数模型，主要是解决了很多细节问题，比如从胖的人到瘦的人怎么调整pose才更合适.

![caption](https://github.com/Hummmm/Hummmm.github.io/blob/master/_posts/2018-08-01-Blend-Shape/4.png?raw=true)

# 转换关系

v=template
vshape=v+shapedir*betas
J=vshape*regressor
vpose=vshape+posedir*pose
