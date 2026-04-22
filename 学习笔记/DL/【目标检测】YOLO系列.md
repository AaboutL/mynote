---
title: 【目标检测】YOLO系列
updated: 2025-07-02 09:45:20Z
created: 2023-03-07 07:27:58Z
latitude: 39.90421100
longitude: 116.40739500
altitude: 0.0000
---

参考链接：https://www.bilibili.com/video/BV1yi4y1g7ro?p=3&vd_source=d5e7a7f2c9d4901c3128ee7eef2f68d5

https://cloud.tencent.com/developer/article/2212232
* * *

# 1\. YOLO v1（2016）

448 * 448 的帧率45fps，63.4mAP

## 1.1 论文思想

<img src="../../_resources/676d6ad5d4a041a7fe0d342a473651ef.png" alt="676d6ad5d4a041a7fe0d342a473651ef.png" width="557" height="354" class="jop-noMdConv">

1.  将一幅图片分成S * S个网格（grid cell），如果某个object的中心落到某个网格中，那么这个网格就负责预测这个object。
2.  每个网格要预测B个bbox（B=2），每个bbox需要预测位置和一个confidence值。
    每个网格还要预测C个类别的分数。
    对于Pascal VOC（20个类别）来说，使用S=7, B=2，C=20。所以最后需要预测的tensor的大小为 7 * 7 * 30（一共有 7 * 7 个grid cell，30 =（4 + 1）* B + C = 5 * 2 + 20）。
    每个bbox由5个值组成，包括（x, y, w, h）+ confidence：
    其中（x，y）表示相对于grid cell的bbox的中心，而（w，h）是相对于全图的。所以 $x, y, w, h \in [0,1]$。
    $confidence = Pr(object) * IoU_{pred}^{truth}$，其中Pr(object)表示网格中是否有目标，如果有目标，则Pr(object)=1, 否则为0.
    测试时，目标的概率为

$$
Pr(class_i | object) * Pr(object) *  IoU_{pred}^{truth} = Pr(class_i) *  IoU_{pred}^{truth}



$$

所以，YOLO输出的目标概率是综合了类别概率和IoU的一个值。

## 1.2 网络结构

<img src="../../_resources/7789769b822a85406599298d0b9dc91f.png" alt="7789769b822a85406599298d0b9dc91f.png" width="735" height="295" class="jop-noMdConv"> <img src="../../_resources/ab006cb91c89f57aea084cf005e632d5.png" alt="ab006cb91c89f57aea084cf005e632d5.png" width="489" height="257" class="jop-noMdConv">

网络的最后一个FC是把前面的4096特征映射成1470，然后把1470的特征转换成 7 * 7 * 30的feature map，其中的每个位置看作是一个长度为30的特征，和上面计算的预测长度是一样的。

## 1.3 损失函数

<img src="../../_resources/d7f614c71bf096f9802876495e16f404.png" alt="d7f614c71bf096f9802876495e16f404.png" width="622" height="321" class="jop-noMdConv">

bounding box损失，宽和高的误差与x、y的误差计算不同，是开根号了。这里主要是考虑到大目标和小目标对于误差大小的敏感程度是不同的。比如，对于一个相同的误差值，其对小目标的影响肯定比对大目标的影响大。如果直接用$|w_{gt} -w_{pred}|、|h_{gt} - h_{pred}|$ 则体现不出这种差别，而使用平方根则可以，如下图，当小目标和大目标的$|w_{gt} -w_{pred}|$相等时，小目标的$\sqrt{w_{gt}} - \sqrt{w_{pred}}$的值更大，说明这个误差对小目标的影响大。
<img src="../../_resources/7a95388fb28bd40571a0f01fc4bf69dd.png" alt="7a95388fb28bd40571a0f01fc4bf69dd.png" width="443" height="285" class="jop-noMdConv">

## 1.4 YOLO的缺点

1.  对于群体性的小目标效果很差。如鸟群，目标小而且密集，由于每个grid cell只预测两个bbox，密集的小目标容易出现在一个grid cell中，此时YOLO无法区分。
2.  当目标出现新的尺寸或配置时，效果较差
3.  主要错误主要来自于定位不准确，因为网络是直接预测位置信息，而不是基于anchor来回归

* * *

# 2\. YOLO v2——YOLO9000：Better, Faster, Stronger（2017）

<img src="../../_resources/f39c15b34dd8e25fa7d64cff66eb662b.png" alt="f39c15b34dd8e25fa7d64cff66eb662b.png" width="877" height="402" class="jop-noMdConv"> <img src="../../_resources/a1e440cdec5bffe784eafe1e12541b52.png" alt="a1e440cdec5bffe784eafe1e12541b52.png" width="572" height="542" class="jop-noMdConv">

## 2.1 YOLO v2中的各种尝试

1.  BN
2.  High resolution classifier
3.  增加anchor box
4.  通过聚类得到anchor box的尺寸和长宽比
5.  Direct location prediction
6.  细粒度的特征
7.  多尺度训练

### 2.1.1 增加BN

<span style="color: red;">起到的效果：</span>

1.  帮助收敛
2.  获得2%的mAP提升
3.  对模型起到了正则化
4.  可以去掉dropout

### 2.1.2 更大输入尺寸进行分类器的fine-tune

<img src="../../_resources/1210ea5bab257cd8f037d64779b9c493.png" alt="1210ea5bab257cd8f037d64779b9c493.png" width="500" height="152" class="jop-noMdConv"> <span style="color: red;">起到的效果：</span>

1.  获得4%的mAP提升

### 2.1.3 基于anchor的预测

<span style="color: red;">起到的效果</span>

1.  简化问题，使得网络更容易学习
2.  大幅提高召回率，使得模型有更大的提升空间
    不使用anchor，mAP=69.5，recall=81%
    使用anchor，mAP=69.2，recall=88%。

### 2.1.4 通过聚类确定anchor的尺寸

使用合适的先验可以使得网络更加容易训练
在训练集上使用k-means聚类获得priors（类似anchor）

### 2.1.5 Direct location prediction

直接基于anchor进行训练，容易出现训练不稳定，特别是训练早期。不稳定主要来源于对（x, y）的预测。Faster R-CNN的预测公式如下：

$$
x = (t_x * w_a) + x_a \\
y= (t_y * h_a) + y_a



$$

$x_a, y_a, w_a, h_a$是anchor的信息，$t_x, t_y$是模型输出的bbox关于x和y的信息。上面的公式没有对$t_x, t_y$进行限制，那么预测bbox可能出现在任意位置。
所以在YOLO v2中使用了新的预测方式：
假设grid cell相对图像左上角的位置为$(c_x, c_y)$，anchor（prior）的宽高为$p_w, p_h$，那么预测公式如下：

$$
b_x = \sigma(t_x) + c_x \\
b_y = \sigma(t_y) + c_y \\
b_w = p_w e^{t_w}  \\
b_h = p_h e^{t_h} \\
Pr(object) * IoU(b, object) = \sigma(t_o) \\
其中 \sigma是sigmoid函数，可以把值限制在【0，1】之间。



$$

<span style="color: red;">起到的效果:</span>

1.  更容易学习，更加稳定
2.  和聚类一起，可以提高5%的mAP

### 2.1.6 细粒度特征（浅层特征）

不仅用13 * 13 特征图进行预测，而是把26 * 26的特征图融合到13 * 13特征图上，一起进行预测。
融合的方法：26 * 26 比13 * 13长宽各大2倍，所以把26 * 26 按照2 * 2的格子划分成13 * 13个格子，每个格子的对应位置拿出一个值，可以组成13 * 13的特征图，一共可以组成4个13 * 13 的特征图，假设原始特征图的大小为26 * 26 * 64，那么转换后变成13 * 13 * 256。把这些特征图concat到上一个特征图上。
<span style="color: red;">起到的效果</span>
提升1%

### 2.1.7 多尺度训练

训练时输入网络的尺寸不再固定，每迭代10个batches，模型随机选择一个新的图像尺寸，由于模型的下采样率是32（输入图像和输出特征图的比例），所以随机从$\{320, 352,..., 608\}$中选择。

## 更快网络的设计：Darknet-19

<img src="../../_resources/3eaa5a48fbdbe996e654023e0631950a.png" alt="3eaa5a48fbdbe996e654023e0631950a.png" width="791" height="432" class="jop-noMdConv">

* * *

# 3\. YOLOv3:An Incremental Improvement(2018)

<img src="../../_resources/028931bf99c456dcae2a84348d49a51b.png" alt="028931bf99c456dcae2a84348d49a51b.png" width="417" height="237" class="jop-noMdConv"> <img src="../../_resources/9168c744cfa5c4eb58aef190c852dc89.png" alt="9168c744cfa5c4eb58aef190c852dc89.png" width="377" height="224" class="jop-noMdConv">

## 3.1 Backbone——Darknet-53

<img src="../../_resources/0826c871132383208c2c8dbfb591ce96.png" alt="0826c871132383208c2c8dbfb591ce96.png" width="857" height="374" class="jop-noMdConv"> <img src="../../_resources/66961272d90a79bf8d67a0830ea72937.png" alt="66961272d90a79bf8d67a0830ea72937.png" width="767" height="373" class="jop-noMdConv">

在三个尺度的特征图上进行预测，每个特征图上会预测3种boxes，所以一共对应9个box的配置：（10 * 13）、（16 * 30）、（33 * 23）、（30 * 61）、（62 * 45）、（59 * 119）、（116 * 90）、（156 * 198）、（373 * 326），如下图
<img src="../../_resources/f4209acd814616335a391bacf452a2a3.png" alt="f4209acd814616335a391bacf452a2a3.png" width="635" height="133" class="jop-noMdConv">
每个特征图上会预测$N * N * (3 * (4 + 1 + 80))$个值（N是特征图的尺寸，4个box offset，1个目标概率，80个类别）
<img src="../../_resources/caea1af2e49ed4d01e5c708d07f0f4f1.png" alt="caea1af2e49ed4d01e5c708d07f0f4f1.png" width="658" height="484" class="jop-noMdConv">

## 3.2 目标边界框预测

<img src="../../_resources/8781b5928ca2725f4ff0b1aff8b896f2.png" alt="8781b5928ca2725f4ff0b1aff8b896f2.png" width="688" height="290" class="jop-noMdConv">

## 3.3 正负样本

正样本：每一个gt 的object会分配一个bbox prior（anchor box），这个prior是所有与gt object重叠的IOU最大的那个。对于其他的大于IOU阈值的（比如0.5）直接丢弃。
负样本：对于IOU小于0.5的prior 当做负样本。
如果bbox prior没有匹配的gt object，那么它不会影响位置的loss和class的loss，只会影响是否有object的loss。（也是负样本）

* * *

# 4\. YOLO v3 SPP

<img src="../../_resources/90eb0e242f99ec50d65b234cdbc09bd9.png" alt="90eb0e242f99ec50d65b234cdbc09bd9.png" width="398" height="206" class="jop-noMdConv">

从上图可以看出，加上SPP结构后，效果提升明显，在加上其他的trick后，效果提升10个点。 YOLOv3-SPP-ultralytics中用到的方法：

1.  Mocaic图像增强
2.  SPP模块
3.  CIOU Loss

## 4.1 Mosaic图像增强

将多张图片拼接在一起，进行训练。默认使用四张图像拼接，优点：

1.  增加数据的多样性
2.  增加一张图像中的目标个数
3.  BN一次性统计多张图片的参数？（存疑，毕竟对BN有不同的解释）

## 4.2 SPP模块

并不是SPPNet中的SPP结构
在第一个预测特征图前增加SPP结构（输入是512 * 512），如下：
<img src="../../_resources/6cb77c8b398203284cc1887256156cf4.png" alt="6cb77c8b398203284cc1887256156cf4.png" width="379" height="270" class="jop-noMdConv">
从图中可以看出，包含四个分支：

1.  残差连接
2.  5 * 5 的最大池化，step = 1
3.  9 * 9 的最大池化，step = 1
4.  13 * 13的最大池化，step = 1
    实现了不同尺度的融合

## 4.3 CIoU loss

YOLOv3中使用的是均方损失（L2损失）

IoU系列loss的发展过程：
<img src="../../_resources/a8c12da7726647c2ea65ae91b9f48b8e.png" alt="a8c12da7726647c2ea65ae91b9f48b8e.png" width="586" height="227" class="jop-noMdConv">

### 4.3.1 IoU loss

<img src="../../_resources/2f1311406015de9beb3ae36eff1281ee.png" alt="2f1311406015de9beb3ae36eff1281ee.png" width="583" height="274" class="jop-noMdConv">

另外一种IoU loss的计算方法如下：

$$
IoULoss = 1 -  IoU



$$

### 4.3.2 GIoU loss

<img src="../../_resources/af95d00c5b14a4dd733ebbd61cf6f7ed.png" alt="af95d00c5b14a4dd733ebbd61cf6f7ed.png" width="597" height="273" class="jop-noMdConv">

其中，$A^c$是两个框的最小外接矩形框的面积，即蓝框的面积 $u$是两个框的并集的面积。 当两个框重合时，$A^c = u, GIoU = 1 - 0 = 1$ 当两个框距离无穷远，$GIoU = 0 - 1 = -1$ 所以，得出：$-1

$$
L_{GIoU} = 1-GIoU, \\
0 <= L_{GIoU} <= 2



$$

当两个框水平或者竖直相交，则GIoU退化成IoU，如下图：
<img src="../../_resources/1047ac2964df7731e2addfef9ec2b763.png" alt="1047ac2964df7731e2addfef9ec2b763.png" width="590" height="186" class="jop-noMdConv">

### 4.3.4 DIoU Loss

<img src="../../_resources/da9f04a453cb1aed2c6cb998ba5c7acc.png" alt="da9f04a453cb1aed2c6cb998ba5c7acc.png" width="675" height="320" class="jop-noMdConv">

针对收敛慢和定位精度差来改进

<img src="../../_resources/76bafa17d0eac581884677bd81d510eb.png" alt="76bafa17d0eac581884677bd81d510eb.png" width="677" height="331" class="jop-noMdConv"> $\\rho^2(b, b^{gt})$指的是预测框和gt框中心点距离的平方。 $c^2$是两个框最小外接矩形框对角线的距离的平方

### 4.3.5 CIoU Loss

<img src="../../_resources/bf2ab59819d78bad8dea344695cc46b8.png" alt="bf2ab59819d78bad8dea344695cc46b8.png" width="676" height="311" class="jop-noMdConv"> $\alpha v$就表示长宽比 示例： <img src="../../_resources/24c979dc335d073651199d94a8f26323.png" alt="24c979dc335d073651199d94a8f26323.png" width="672" height="291" class="jop-noMdConv">

## 4.4 Focal Loss

针对one-stage中正负样本不均衡的问题
two-stage中第二阶段输入只有2000个候选框，相比one-stage，不均衡问题减轻很多
<img src="../../_resources/8c5fe80cd384839c002c071497c093e6.png" alt="8c5fe80cd384839c002c071497c093e6.png" width="694" height="331" class="jop-noMdConv">
<img src="../../_resources/aed5794d3dfb1123abfbbd314c4dbd11.png" alt="aed5794d3dfb1123abfbbd314c4dbd11.png" width="695" height="356" class="jop-noMdConv">
<img src="../../_resources/9764100df80bee77a5aeb84f823b2563.png" alt="9764100df80bee77a5aeb84f823b2563.png" width="696" height="314" class="jop-noMdConv">
<img src="../../_resources/f7b11f1e3e134240bd1df2a9606e5d75.png" alt="f7b11f1e3e134240bd1df2a9606e5d75.png" width="690" height="328" class="jop-noMdConv">
<img src="../../_resources/7bb41d19a8204f42d0310358240f18d1.png" alt="7bb41d19a8204f42d0310358240f18d1.png" width="687" height="323" class="jop-noMdConv">
<img src="../../_resources/ebeeddf00537d97150237a85c9a6de7b.png" alt="ebeeddf00537d97150237a85c9a6de7b.png" width="679" height="319" class="jop-noMdConv">
<img src="../../_resources/06744d9521c918c66635c9743a257e49.png" alt="06744d9521c918c66635c9743a257e49.png" width="675" height="338" class="jop-noMdConv">

Focal loss的问题：易受噪音干扰，如果标注结果不准确，出现错误标注，那么就容易造成网络出问题。