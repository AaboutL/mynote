---
title: slam基础-IMU预积分
updated: 2022-12-02 02:02:34Z
created: 2022-11-01 03:32:33Z
latitude: 39.92317400
longitude: 116.44142800
altitude: 0.0000
---

# IMU初始化

根据orbslam3，通过IMU初始化应该获得：

1.  尺度s
2.  重力方向到世界系的变换$R_{WB}$
3.  加速度计和陀螺仪的零偏$\vec b = (\vec b_a, \vec b_w)$
4.  IMU的在每一帧时刻的速度$\vec v_{0:k}$
    惯导状态向量$Y = (s, R_{WB}, \vec b, \vec v_{0:k})$

把第0帧到第k帧之间的IMU测量数据积分起来，得到$I_{0:k} = (I_{01}, I_{12}, ..., I_{k-1k})$
根据上面的表示，构造一个最大后验估计问题：
<img src="../../../_resources/6cdf7d22b819cb7bfb54070ef80f0795.png" alt="6cdf7d22b819cb7bfb54070ef80f0795.png" width="424" height="72" class="jop-noMdConv">
<img src="../../../_resources/95ecbe8cacdd00fd73abff1564f7404a.png" alt="95ecbe8cacdd00fd73abff1564f7404a.png" width="399" height="78" class="jop-noMdConv">

## IMU预积分形式推出

IMU初始化过程中，真实值是通过camera得到的第j帧相对第i帧的变化：包括旋转变化、速度变化、位置变化，通过这三种变化，构造出预积分的真实值（真实值是假设没有噪声的）
IMU的积分形式如下：
<img src="../../../_resources/d9f425834afaae7bf002484d516bcf11.png" alt="d9f425834afaae7bf002484d516bcf11.png" width="454" height="185" class="jop-noMdConv">
上面的形式包含了i时刻的状态。每次经过优化，$i$ 时刻的状态会发生变化，那么就不能再使用旧的i时刻的状态来推导 $j$ 时刻的状态了，需要根据新的i时刻的状态重新积分得到新的j时刻的状态。为了避免重新积分，推出下面的预积分形式：

<img src="../../../_resources/569525a88cc3b141fd08225b205e160d.png" alt="569525a88cc3b141fd08225b205e160d.png" width="473" height="240" class="jop-noMdConv">

上面预积分形式中的$\Delta R_{ij}, \Delta v_{ij}, \Delta p_{ij}$都没有实际物理意义。
**红框中的部分告诉我们在IMU初始化过程中，如何从视觉帧中得到预积分的真实值。**
**蓝色中的部分告诉我们如何通过IMU的测量值计算预积分**
可以看出，每一个等式都是把所有与 $i$ 时刻相关的状态都剔除了。

## 噪声分离

上面的公式与IMU噪声的关系比较复杂，导致使用MAP估计得时候也很复杂。把噪声项从上面的(33)中分离出来，得到下面的式子：
<img src="../../../_resources/189cac6cfc0d6720db16c0c2e84ae4ce.png" alt="189cac6cfc0d6720db16c0c2e84ae4ce.png" width="460" height="159" class="jop-noMdConv">
<img src="../../../_resources/3d97f3ca08f3dec17a5fe162d18a3e6c.png" alt="3d97f3ca08f3dec17a5fe162d18a3e6c.png" width="467" height="155" class="jop-noMdConv">
<img src="../../../_resources/d2c19d02950422b33eabd3d7846ccfc6.png" alt="d2c19d02950422b33eabd3d7846ccfc6.png" width="452" height="209" class="jop-noMdConv">
上面的三个式子说明与噪声相关的项都可以提出来，而且其形式类似”$真实值=测量值-噪声$“，把上面三个式子带入(33)中，得到预积分的测量模型：
<img src="../../../_resources/2410d2ef21476971e7e105b90052472a.png" alt="2410d2ef21476971e7e105b90052472a.png" width="483" height="127" class="jop-noMdConv">
其中$（\delta \phi_{ij}, \delta v_{ij}, \delta p_{ij}）$组成了预积分的噪声向量.
从(35)(36)(37)可以看出，预积分噪声是关于IMU测量噪声的线性变换，所以也是零均值的正态分布，这对准确建模噪声的协方差很重要，用协方差的逆对下面的优化方程中的优化项进行加权：
<img src="../../../_resources/62a11a95ee13e0067a3843c9e6765f79.png" alt="62a11a95ee13e0067a3843c9e6765f79.png" width="470" height="101" class="jop-noMdConv">

## IMU噪声递推

### 1\. 关于旋转预积分的噪声递推

从（35）中得出旋转的噪声表示：
<img src="../../../_resources/d13a370ccd3d9d8cede3bffe3daa7c07.png" alt="d13a370ccd3d9d8cede3bffe3daa7c07.png" width="509" height="53" class="jop-noMdConv">
等式两边取负对数：
<img src="../../../_resources/43cc1fa063b85234acd1500ec50c0fc6.png" alt="43cc1fa063b85234acd1500ec50c0fc6.png" width="513" height="50" class="jop-noMdConv">
重复泰勒一阶近似：
<img src="../../../_resources/cba56aa4eea008c9d136b7185bd5ce4b.png" alt="cba56aa4eea008c9d136b7185bd5ce4b.png" width="423" height="52" class="jop-noMdConv">

<img src="../../../_resources/4f0720016289e51b79f5e1b3f0a3bd8e.png" alt="4f0720016289e51b79f5e1b3f0a3bd8e.png" width="434" height="256" class="jop-noMdConv">

### 2.关于速度/位置预积分的噪声递推

从(36)(37)中得出，噪声$\delta v_{ij}, \delta p_{ij}$是关于加速度计噪声$\eta_k^{ad}$和关于旋转预积分噪声的线性组合，所以其递推形式如下：
<img src="../../../_resources/f82fd04f8dba74fb12d533c662abe27e.png" alt="f82fd04f8dba74fb12d533c662abe27e.png" width="552" height="136" class="jop-noMdConv">

<img src="../../../_resources/03788e5017e8aa2f5d396829b60188f8.png" alt="03788e5017e8aa2f5d396829b60188f8.png" width="552" height="166" class="jop-noMdConv"> <img src="../../../_resources/f796ba96a64f1b763002f139afe0c6f6.png" alt="f796ba96a64f1b763002f139afe0c6f6.png" width="573" height="118" class="jop-noMdConv">

从上面的递推公式可以看出，预积分的噪声是关于IMU测量噪声的线性变换，而IMU测量噪声在IMU的规格书中有说明。所以可以通过递推的方式，从IMU测量噪声递推的计算出ij的预积分噪声。

### 写成矩阵形式

定义：预积分观测噪声：（把预积分的值当做观测）
$$
\eta_{ij}^{\Delta} = [\delta \phi_{ij}^T, \delta v_{ij}^T, \delta p_{ij}^T]^T \in R^9
$$

令IMU测量噪声
$$
\eta_k^d = [(\eta_k^{gd})^T, (\eta_k^{ad})^T]^T
$$
有：
<img src="../../../_resources/7900df4cb58517f87ebde5fd0c5803db.png" alt="7900df4cb58517f87ebde5fd0c5803db.png" width="721" height="131" class="jop-noMdConv">
即：<img src="../../../_resources/1a05a0f873653409901ffa3c52591bb8.png" alt="1a05a0f873653409901ffa3c52591bb8.png" width="237" height="39" class="jop-noMdConv">
其中，A表示预积分观测噪声递推的雅可比，B是预积分观测噪声关于IMU测量噪声的雅可比
**协方差矩阵的表示：（此协方差指的是三个预积分之间的协方差）**
<img src="../../../_resources/b5470ef5824c9f3524d4f3862cad00cf.png" alt="b5470ef5824c9f3524d4f3862cad00cf.png" width="358" height="53" class="jop-noMdConv">

* * *

在此之前的推导都假设IMU的零偏在相邻两个关键帧之间是固定的，给出零偏更新后，预积分的更新方法

* * *

## bias更新

当给定一个零偏的更新$b \leftarrow \bar b + \delta b$，可以通过泰勒一阶展开来更新预积分：
<img src="../../../_resources/65c050f97ce93e4d4f06604ffec977f8.png" alt="65c050f97ce93e4d4f06604ffec977f8.png" width="564" height="131" class="jop-noMdConv">
其中的Jacobian矩阵说明了预积分如何随零偏的变化而变化。在预积分过程中，Jacobian矩阵是固定的，而且可以提前计算。Jacobian矩阵的表示如下：
<img src="../../../_resources/3d1889de94a57e865528491d5f94233a.png" alt="3d1889de94a57e865528491d5f94233a.png" width="596" height="175" class="jop-noMdConv">
上面的Jacobian可以进一步推出其递推形式，这样雅可比就不需要每次从头计算了。
### 1. 旋转预积分关于$b^g$的雅可比递推：( https://github.com/UZ-SLAMLab/ORB_SLAM3/issues/212 )
<img src="../../../_resources/e12eb6dbbe2740890071ec6f1ab4bed7.png" alt="e12eb6dbbe2740890071ec6f1ab4bed7.png" width="510" height="180" class="jop-noMdConv">

### 2. 速度预积分关于$b^a$的雅可比递推：
$$
\frac{\partial \Delta \bar v_{i,j}}{\partial b^a} = \frac{\partial \Delta \bar v_{i,j-1}}{\partial b^a} - \Delta \bar R_{i, j-1}\Delta t
$$
### 3. 速度预积分关于$b^g$的雅可比递推：
$$
\frac{\partial \Delta \bar v_{i,j}}{\partial b^g} = \frac{\partial \Delta \bar v_{i,j-1}}{\partial b^a} - \Delta \bar R_{i,j-1}(\tilde a_{j-1} - \bar b_i^a)^{\wedge} \frac{\partial \bar R_{i,j-1}}{\partial b^g}\Delta t
$$
### 4. 位置预积分关于$b^a$的雅可比递推：
$$
\frac{\partial \Delta \bar p_{i,j}}{\partial b^a} = \frac{\partial \Delta \bar p_{i,j-1}}{\partial b^a} + \frac{\partial \Delta \bar v_{i, j-1}}{\partial b^a}\Delta t - \frac{1}{2}\Delta \bar R_{i,j-1}\Delta t^2 
$$
### 5. 位置预积分关于$b^g$的雅可比递推：
$$
\frac{\partial \Delta \bar p_{i,j}}{\partial b^g} = \frac{\partial \Delta \bar p_{i,j-1}}{\partial b^g} + \frac{\partial \Delta \bar v_{i,j-1}}{\partial b^g}\Delta t - \frac{1}{2}\Delta \bar R_{i,j-1}(\tilde a_{j-1}- \bar b_i^a)^{\wedge}\frac{\partial \Delta \bar R_{i,j-1}}{\partial b^g}\Delta t^2
$$

## 预积分的残差

根据预积分的测量模型(38)以及零均值正态分布的噪声，残差可以表示为：$r_{I_{ij}} = [r_{\Delta R_{ij}}^T, r_{\Delta v_{ij}}^T, r_{\Delta p_{ij}}^T]^T \in R^9$:
<img src="../../../_resources/187b9bb1091eaf67b76d1eb4a06ed1fc.png" alt="187b9bb1091eaf67b76d1eb4a06ed1fc.png" width="597" height="244" class="jop-noMdConv">