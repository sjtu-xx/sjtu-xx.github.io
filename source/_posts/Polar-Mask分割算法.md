---
title: Polar Mask分割算法【转载】
toc: true
date: 2020-04-08 22:36:16
tags:
categories:
- 深度学习
- 实例分割
---

PolarMask的精度并不是很高，而且速度上也没有优势，但是它的思路是非常巧妙的，对后面的研究有着很大的启发意义。
<!--more-->
转载自：https://zhuanlan.zhihu.com/p/84890413

PolarMask, 一种single shot的实例分割框架，文章在[https://arxiv.org/abs/1909.13226](https://link.zhihu.com/?target=https%3A//arxiv.org/abs/1909.13226), 代码已经开源，在[https://github.com/xieenze/PolarMask](https://link.zhihu.com/?target=https%3A//github.com/xieenze/PolarMask)。欢迎大家多多指教。

PolarMask基于FCOS，把实例分割统一到了FCN的框架下。在4月份FCOS挂出来之后，我和沈老师就一直在讨论如何进把instance segmentation融合进FCN里。FCOS本质上是一种FCN的dense prediction的检测框架，可以在性能上不输anchor based的目标检测方法，让行业看到了anchor free方法的潜力。接下来要解决的问题是实例分割。

这个工作我个人觉得最大的贡献在于，把更复杂的实例分割问题，转化成在网络设计和计算量复杂度上和物体检测一样复杂的任务，把对实例分割的建模变得简单和高效。

Mask R-CNN
首先回顾一下之前最经典的实例分割方法，‘先检测再分割’，在这方面做到极致的算法是Mask RCNN。

![img](image/v2-ba7114dd24d45324477916feda18ba92_1440w.jpg)



Mask R-CNN属于基于两阶段的检测算法，在检测框的基础上进行像素级的语义分割，简化了实例分割的难度，同时取得了stoa的性能，在'先检测再分割’ 这一范式上做到了极致。

------

**PolarMask

**我们的PolarMask提出了一种新的instance segmentation建模方式，通过寻找物体的contour建模，提供了一种新的方法供大家选择。

**两种实例分割的建模方式：

**1 像素级建模 类似于图b，在检测框中对每个pixel分类
2 轮廓建模 类似于图c和图d，其中，图c是基于直角坐标系建模轮廓，图d是基于极坐标系建模轮廓

![img](image/v2-e1631235db01c2ca6e65c9601bb2b7f2_1440w.jpg)

可以看到Mask R-CNN属于第一种建模方式，而我们提出的PolarMask属于图d建模方式。图c也会work，但是相比图d缺乏固定角度先验。换句话说，基于极坐标系的方式已经将固定角度设为先验，网络只需回归固定角度的长度即可，简化了问题的难度。


PolarMask 基于极坐标系建模轮廓，把实例分割问题转化为实例中心点分类(instance center classification)问题和密集距离回归(dense distance regression)问题。同时，我们还提出了两个有效的方法，用来优化high-quality正样本采样和dense distance regression的损失函数优化，分别是Polar CenterNess和 Polar IoU Loss。没有使用任何trick(多尺度训练，延长训练时间等)，PolarMask 在ResNext 101的配置下 在coco test-dev上取得了32.9的mAP。 这是首次，我们证明了更复杂的实例分割问题，可以在网络设计和计算复杂度上，和anchor free物体检测一样简单。我们希望PolarMask可以成为一个简单且强有效的single shot instance segmentation 的baseline。

PolarMask最重要的特点是:
(1) anchor free and bbox free，不需要出检测框
(2) fully convolutional network, 相比FCOS把4根射线散发到36根射线，将instance segmentation和object detection用同一种建模方式来表达。

我们选取FCOS嵌入我们的方法，主要是为了simple。FCOS是目前state-of-the-art的anchor-free检测器，并且十分simple。我们在FCOS的基础上，几乎不加任何计算量，就可以建模实例分割问题，并取得competitive的性能，证明了实例分割可以简化成和目标检测相同复杂的问题。
此外，FCOS可以看成PolarMask的特殊形式，而PolarMask可以看作FCOS的通用形式，因为bbox本质上是最简单的Mask，只有0,90,180,270四个角度回归长度。 我们首次将instance segmentation和object detection用同一种建模方式来表达。

------



**网络结构**

![img](image/v2-d078fdf98ec216ace64b3f58e29a7b4b_1440w.jpg)

整个网络和FCOS一样简单，首先是标准的backbone + fpn模型，其次是head部分，我们把fcos的bbox分支替换为mask分支，仅仅是把channel=4替换为channel=n, 这里n=36，相当于36根射线的长度。同时我们提出了一种新的Polar Centerness 用来替换FCOS的bbox centerness。
可以看到，在网络复杂度上，PolarMask和FCOS并无明显差别。

**Polar Segmentation建模**

![img](image/v2-9f57162d6c22a2f8cedcc551fdc768db_1440w.jpg)

首先，输入一张原图，经过网络可以得到中心点的位置和n(n=36 is best in our setting)根射线的距离，其次，根据角度和长度计算出轮廓上的这些点的坐标，从0°开始连接这些点，最后把联通区域内的区域当做实例分割的结果。
在实验中，我们以重心为基准，assign到feature map上，会在重心周围采样，作为正样本，别的地方当做负样本，训练方式和FCOS保持一致，采用Focal Loss, 在此，我们提出Polar CenterNess，用来选择出高质量的正样本，给低质量的正样本降低权重。

------

**Polar CenterNess**

如何在Polar Coordinate下定义高质量的正样本？我们通过如下公式定义

![img](image/v2-1935d8988a501ab1532035a32dc8ab6b_1440w.jpg)

其中 d1 d2....dn指的是36根射线的长度，最好的正样本必须具备d*min ——>* d*max.*

用一张图举例

![img](image/v2-21693594e81a6e2cb03811997f378325_1440w.jpg)

以看到中间的图，会出现长度回归差别很大的问题，而右边的图中心点位置就较为合适，到所有轮廓的长度回归就较为接近，36根射线的距离会比较均衡。 Polar Centerness 可以给右边图的点较高的centerness分数，给中间图的点降低centerness分数，这样在infernece的时候右边图的点分数较高。

根据消融实验，Polar Centerness可以有效提高1.4的性能，同时不增加网络复杂度。结果如下图所展示

![img](image/v2-c245decddce93a7088bdd848f7d7099d_1440w.jpg)

------

**Polar IoU Loss**

在PolarMask中，需要回归k(k=36)根射线的距离，这相比目标检测更为复杂，如何监督regression branch是一个问题。我们提出Polar IoU Loss近似计算出predict mask和gt mask的iou，通过Iou Loss 更好的优化mask的回归。通过实验证明，Polar IoU Loss相比Smooth L1loss可以明显提高2.6个点，同时Smooth L1loss还面临和其他loss不均衡的问题，需要精心调整权重，这是十分低效的，Polar IoU loss不需要调整权重就可以使mask分支快速且稳定收敛。
那么，Polar IoU Loss如何计算呢？如下图所展示

![img](image/v2-2e74b81da53e7d1794d6d60d4088cdb2_1440w.jpg)

可以看到 两个mask的Iou可以简化为在dθ下的三角形面积iou问题并对无数个三角形求和，最终可以推倒到如下形式：

![img](image/v2-afc9b59d8829ed113e9f53999f8955da_1440w.jpg)

其实最终的表达形式十分简单，但是相比smooth l1的确可以不用调参并裸涨2.6个点。说明loss func的设计对于深度神经网络呢意义重大。结果如下所展示

![img](image/v2-000f2903d0c2098ad7465391ba4cb444_1440w.jpg)

我们在论文中还做了如下消融实验：射线数量的选择，加不加bbox branch, backbone以及尺寸和速度的trade off. 细节在论文中都有，不一一展开。

------

**上限分析**

看到这里，很多人心里都会有一个疑问，射线这种建模方式，对于凹的物体会有性能损失，上限达不到100mAP，PolarMask怎么处理这个问题？
答案是这样，PolarMask相比Mask R-CNN这种pixel建模的方法，对于形状特别奇怪的mask的确建模会失败，但是这并不代表polarmask毫无意义。原因有两个，(1)Mask R-CNN的上限也到不了100 mAP 因为有下采样这类操作使得信息损失。(2)不管Mask R-CNN还是PolarMask，他们的实际性能距离100mAP的上限都特别远。
所以我们目前应该关注如何让实际网络性能去更好地趋近于上限。

定量分析分析射线建模的上限：

![img](image/v2-9b1ec4ad61022d706976f7100a43e3df_1440w.jpg)

如图所示，当采用mass center做instance中心时，当射线数量不断提高，射线的gt和真实的gt的平均iou高达90%以上，这证明了对于射线建模的性能上限的忧虑还远远不需要担心。现阶段需要操心的问题是如何不断提高基于射线建模的网络性能。



------

**实验**

最终，配上一图一表展示一下相比sota的结果

![img](image/v2-59ed0e0bac433b0d37a2e8b553610d43_1440w.jpg)

可以看到, 没用采用任何trick的情况下，PolarMask在resnext101-fpn的情况下，取得了32.9的配置，虽然不是stoa，但是也比较有竞争力。我们目前并没有采用很多常用的能涨点的trick，比如 ms train和longer training epochs。相比之下，别的one stage方法都不约而同的采用了mstrain和longer training epoches。 我们会进一步改进，争取再提高性能。



------

**后记**：

我们会尽快完善并放出带多尺度训练和增长训练时间的代码和模型，以提供给大家并和上述模型公平比较，做instance segmentation非常费卡和时间，希望大家理解。

------

**一些调参的碎碎念：**

这个工作在今年4月FCOS出来的同时，我和沈春华老师就在讨论如何进行FCOS进行single shot anchor free实例分割，以及在CVPR开会的时候，和文海，彦伟, 宋林等小伙伴就论证过contour regression的可行性，因为大家可以看出，anchor free 物体检测已经是大势所趋，下一个领域必然是anchor free实例分割，预测一下，再下一个领域是全景分割。在和沈老师这么多月的讨论中以及一次又一次实验的失败中，polarmask逐渐成型。最开始mask regression无数次无法收敛，写mask iou loss又复杂效果还不怎么好，直到某一天想出了polar iou loss。发现性能很好，不用调参，一把出结果。感觉平时做研究还是需要多思考，而不是盲目的做实验。同时，polar centerness也是我在旅游的过程中想到的，然后立马找了个咖啡厅写代码调试，裸涨1.4个点。这两个方法在不增加计算量的情况下，充分发挥了polarmask的性能，看起来其实很简单，但是实际上是背后很多次思考和化简的结果。

总得来说，我最喜欢这篇文章就两点，

1 足够简单，不加任何trick, 也没有任何复杂的操作，比如deformable conv和roi align操作，有希望在工业界大规模应用

2 找到了一种表达方式，把bbox detection和mask segmentation统一了起来，和FCOS是一种传承的工作，FCOS理论上可以看成PolarMask的特殊版，而PolarMask是FCOS的泛化版，因为bbox本质上是最简单的Mask。 PolarMask本质上可以看成一个目标检测和实例分割统一的框架。只需要简单修改就可以退化到FCOS。
