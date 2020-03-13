---
title: 双目自动调参算法，暨DARPA机器人比赛项目
tags: SLAM
edit: 2015-06-01
categories: 技术类
status: Completed
mathjax: true
highlight: true
mermaid: true
description: 立体匹配是依赖大量手工调参的过程才可以得到较好的深度图，我们希望可以有一种自动调参算法来代替人工。
---

#文章动机
立体匹配是依赖大量手工调参的过程才可以得到较好的深度图，我们希望可以有一种自动调参算法来代替人工。本篇并不打算用公式来阐述原因，更想表达的是为什么这么做立体匹配算法背后的动机。

![caption](https://github.com/Hummmm/Hummmm.github.io/blob/master/_posts/2015-06-01-Depth-Tuning/depth.png?raw=true)
> The stereo parameter update in the dynamic scene. From left to right, each column shows the data when the door is closed, partially open, and fully open, respectively. The first row shows the camera observations while the door is in different situations. The second row shows the depth map before the stereo parameter tuning. The third rows shows the depth map after the phase I tuning of our approach. The last row shows the depth map after the phase II tuning of our approach.

# 目标标准
首先制定一个好的深度匹配图目标
- 不同深度的物体图像边缘光滑清晰
- 相同深度色块内没有噪音点，稠密
- 深度图尽可能包含整个场景近远物体

# 参数 
立体图像匹配前：
- 亮度：图像增强预处理，图像不可以过亮过暗，因此可以引入一个亮度阈值限制均值化后图像的强度。
- 光滑：做完极线矫正以后的双目图像对应点应该在同一条水平线上，对应的patch之间亮度变化（SAD）应该类似。SAD 窗越大，对应3D点依赖邻间像素信息变化更多，匹配出来的深度图越光滑。
- 景深：不同的场景深度范围不同，可以设置一个视差范围来限制深度搜索范围（horopter）。一来过近过远的像素计算得到的距离误差大，而来剔除不靠谱的视差可以减少计算。

立体图像匹配后：
- 准确度：除了周边patch，对应patch之间的相似度（SAD response）高于平均水平。
- 边缘：当patch在物体边缘的时候会同时包含小视差和大视差（speckle)。当speckle内视差范围过大时往往在物体边缘上，可以删除。


# 参数调整
在所有参数中，景深是最重要的一个参数。当景深范围错误，会有大块图像空或者细节信息丢失。寻找视差偏小和偏大的像素, 根据它们立刻调整景深范围。
- 如果图像包含地面，物体景深往往是连续的；不然反之。
- 景深过于极端的零星点基本是噪音

# 目标函数
- 一个高质量的深度图应当充满对应点
- 深度图投射到双目图像后应当同位置亮度类似
- 当speckles少时，每个深度图小区域内景深变化不大
- 立体匹配运行时间少

# 优化：
CMA-ES优化算法

# 实验
使用Lidar作为ground truth，双目立体匹配得到的数据作为预测值，测试分为动态和静态环境。

# 结果

环境变化后隔几帧算法得到的深度图可以和一个有经验的调参技术人员得到的目标函数值类似，并且明显优于调参新手的结果。























