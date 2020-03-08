---
title: 目标检测
tags: DeepLearning
edit: 2019-06-02
categories: 技术类
status: Completed
mathjax: true
highlight: true
mermaid: true
description: 目标检测目前有 one-stage 和 two-stage 两种，two-stage 指的是检测算法需要分两步完成，首先需要获取候选区域，然后进行分类，比如R-CNN系列；与之相对的是 one-stage 检测，可以理解为一步到位，不需要单独寻找候选区域，典型的有SSD/YOLO。
---
![caption](https://github.com/Hummmm/Hummmm.github.io/blob/master/_posts/2019-06-10-Object-Detection/YOLOv1-01.png?raw=true)
> 各网络结构图
 
我们根据时间顺序来讲目标检测的发展过程。
##Two stage流程
物体检测方法首先需要产生大量可能包含待检测物体的先验框, 然后用分类器判断每个先验框对应的边界框里是否包含待检测物体，以及物体所属类别的概率或者置信度，同时需要后处理修正边界框，最后基于一些准则过滤掉置信度不高和重叠度较高的边界框，进而得到检测结果。这种基于先产生候选区再检测的方法虽然有相对较高的检测准确率，但运行速度较慢。

###滑动窗口检测器
 一种用于目标测的检测暴力方法是从左到右、从上到下滑动窗口，利用分类识别目标。一来这样导致需要大量不同尺寸的窗口，而来计算量巨大。要提升性能，一个显而易见的办法就是减少窗口数量。
```
for window in windows:
    patchs = get_patch(image, window)
    results = detector(patchs)
```
###RCNN
**本质**：RCNN 并没有使用过多深度学习，只是从经验驱动特征（SIFT、HOG）到数据驱动特征（CNN feature map). 
```
ROIs = region_proposal(image) #用传统selective search来选候选区域（Region Proposals）并且调整固定大小
for ROI in ROIs 
   patch = get_patch(image, ROI) #用CNN获取感兴趣的区域（ROI）
   results = detector(patch)#利用 feature map 训练SVM来对目标和背景进行分类（每个类一个二进制SVM），最后用边界框回归（Bounding boxes Regression）输出一些校正因子的线性回归分类器。
```
###Fast RCNN
**动机**：RCNN很多ROI是重叠的，并且是multiple-stage pipeline，需要额外的特征存储空间，因此速度并没有很快。
**特性**：Fast R-CNN 最重要的一点就是包含特征提取器、分类器和边界框回归器在内的整个网络能通过多任务损失函数进行端到端的训练。
```
feature_maps = process(image) #使用CNN网络先提取整个图像的特征
ROIs = region_proposal(feature_maps) #用Selective Search选定的候选区域映射到卷积层生成ROIs，而不是对每个图像块提取多次。
for ROI in ROIs 
   patch = roi_pooling(feature_maps, ROI) #使用全连接层，所以应用RoI Pooling将不同大小的ROI转换为固定大小
   results = detector2(patch)#这种多任务损失即结合了分类损失和定位损失的方法，大大提升了模型准确度。
```
###Faster R-CNN 
**动机**：《Fast R-CNN依赖于外部候选区域方法，如选择性搜索。但这些算法在CPU上运行且速度很慢。
```
feature_maps = process(image) 
ROIs = rpn(feature_maps) #流程图与 Fast R-CNN 基本相同，但用RPN代替了原来的外部候选区域。
for ROI in ROIs 
   patch = roi_pooling(feature_maps, ROI) 
   class_scores, box = detector(patch) 
   class_probabilities = softmax(class_scores)
```
- 区域生成网络（RPN）
**本质**： 基于滑窗的无类别obejct检测器。
Faster R-CNN 不会创建随机边界框。相反，它会预测一些与锚点的参考框相关的偏移量（如𝛿x、𝛿y）。在 Faster R-CNN 中，我们使用卷积核得到结果馈送到两个独立的全连接层，来做 6个参数的预测：4 个参数对应某个锚点的预测边框，2个参数对应 objectness 置信度得分。two stage型的检测算法在RPN 之后 还会进行 再一次 的 分类任务 和 边框回归任务，以进一步提升检测精度。

##one-stage检测器
将物体检测任务直接当作回归问题（regression problem）来处理，将候选区和检测两个阶段合二为一。只需一眼就能知道每张图像中有哪些物体以及物体的位置。单次检测器通常需要在准确率和实时处理速度之间进行权衡。它们在检测太近距离或太小的目标时容易出现问题.
识别与定位合二为一，结构简便，检测速度快。整个流程中都能看到整张图像的信息，因此它在检测物体时能很好的利用上下文信息，从而不容易在背景上预测出错误的物体信息。同时可以学习到高度泛化的特征，能将一个域上学到的特征迁移到不同但相关的域上，如在自然图像上做训练的网络，在艺术图片上可以得到较好的测试结果。
```
feature_maps = process(image) 
results = detector3(feature_maps) # No more separate step for ROIs
```
###SSD
介于RCNN和YOLO之间，速度和精度比较平衡。SSD采用和fast rcnn一样的vgg来做特征提取，之后在多尺度特征上做卷积，最后用卷积核预测。SSD 使用卷积网络中较深的层来检测目标。如果我们按接近真实的比例重绘上图，我们会发现图像的空间分辨率已经被显著降低，且可能已无法定位在低分辨率中难以检测的小目标。如果出现了这样的问题，我们需要增加输入图像的分辨率

SSD 通过多个特征图完成检测。但是，最底层不会被选择执行目标检测。它们的分辨率高但是语义值不够，导致速度显著下降而不能被使用。SSD 只使用较上层执行目标检测，因此对于小的物体的检测性能较差,虽然该重建层的语义较强，但在经过所有的上采样和下采样之后，目标的位置不精确。在重建层和相应的特征图之间添加横向连接可以使位置侦测更加准确。

- Focal Loss
类别不平衡会损害性能。SSD 在训练期间重新采样目标类和背景类的比率，这样它就不会被图像背景淹没。Focal loss（FL）采用另一种方法来减少训练良好的类的损失。因此，只要该模型能够很好地检测背景，就可以减少其损失并重新增强对目标类的训练。我们从交叉熵损失 （Cross Entroy Loss）开始，并添加一个权重来降低高可信度类的交叉熵。
例如，令 γ = 0.5, 经良好分类的样本的 Focal Loss 趋近于 0。

###YOLO v1
one stage detection的开山之作，事实上，YOLO也并没有真正的去掉候选区，而是直接将输入图片划分成7x7=49个网格（因此YOLO要求输入图像有固定大小），每个网格预测两个边界框，一共预测49x2=98个边界框。YOLO 在卷积层之后使用了 DarkNet 来做特征检测。它并没有使用多尺度特征图来做独立的检测。相反，它将特征图部分平滑化，并将其和另一个较低分辨率的特征图拼接。输入一张图片到YOLO网络将输出一个7730的张量表示图片中每个网格对应的可能的两个边界框以及每个边界框的置信度和包含的对象属于各个类别的概率。
###YOLO v2
**动机**：由于YOLO网格设置比较稀疏，其总体预测精度不高( 低召回率)，其对小物体的检测效果较差，尤其是对密集的小物体表现比较差。
YOLO v2在v1的基础上改进非常多，每一个改进都值得学习。
- BN 层:起到一定的正则化效果，能提升模型收敛速度，防止模型过拟合
- 高分辨率图像：避免低分辨率会影响模型检测能力
- Anchor：v1利用全连接层直接对边界框进行预测，导致丢失较多空间信息，定位不准。v2去掉了全连接层，使用Anchor Boxes预测边界框，同时为了得到更高分辨率的特征图，v2还去掉了一个池化层，模型的召回率提升。
- 先验框聚类：用聚类分析得到的先验框比手动选择的先验框有更高的平均IOU值。
- Darknet-19：对输入图片的大小没有限制，mAP值没有显著提升，但计算量减少了。
- Direct location prediction：约束先验框偏移量，网络中将得到的网络预测结果再输入sigmoid函数中，让输出结果介于0到1之间。
- pass through层：将高分辨率的特征图与低分辨率的特征图联系在一起，类似resnet。

###YOLO v3
YOLO v3是均衡精度和速度的目标网络，在小物体检测问题上得到改善。主要的改进有：更好的主干网络（类ResNet）；利用多尺度特征进行对象检测（类FPN，3尺度*9个锚）对于小目标的检测效果提升还是比较明显的。

- 多尺度融合：虽然在YOLO v3中每个grid cell预测3个bounding box，看起来比YOLO v2中每个grid cell预测5个bounding box要少，其实不是！因为YOLO v3采用了多个scale的特征融合，所以boundign box的数量要比之前多很多。

- 多标签分类：类别预测方面主要是将原来的单标签分类改进为多标签分类，对象分类用多个logistic取代了softmax，因为softmax不适合多标签。
YOLO v3由c和cuda实现，GPU利用效率高，只依赖OPENCV第三方库，可移植性高。快速简单，背景误检率低，通用强，但是位置精准性差，召回率低（每个网格的候选框个数少导致，虽然v23采用了锚，但是没SSD复杂）。

- Darknet
YOLO作者自己写的一个深度学习框架叫darknet. 后来在YOLOv2中又提了一个基于ResNet魔改的19层卷积网络，称为Darknet-19，在YOLOv3中又提了一个更深的Darknet-53。这两个都是用于提取特征的主干网络。可以用这个框架实现resnet mobilenet等其他backbone，也可以实现darknet这个backbone。

- FPN
上面使用不同尺寸特征图进行预测的网络称为特征金字塔网络（FPN），是一种旨在提高准确率和速度的特征提取器。FPN 由自下而上和自上而下路径组成。其中自下而上的路径是用于特征提取的常用卷积网络。空间分辨率自下而上地下降。当检测到更高层的结构，每层的语义值增加



 

