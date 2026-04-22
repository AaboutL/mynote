---
title: ORB-SLAM2模块解析
updated: 2022-06-24 01:25:04Z
created: 2022-06-06 09:13:04Z
---

# ORB-SLAM2模块解析
<img src="../../../_resources/orbslam2.png" alt="orbslam2.png" width="683" height="312">

## 1. 初始化模块
### 1.1 单目初始化
初始化的目的是恢复最开始两帧之间的相对位姿，以及对两帧之间匹配的关键点三角化得到初始地图点。**恢复相对位姿、三角化得到初始地图点**
初始化过程：
```mermaid
graph TB
1[基于匹配点选择mMaxIteration组用于RANSAC的数据,每组数据8对随机匹配点]-->
2[用RANSAC数据分别计算基础矩阵和单应矩阵]-->
3[然后分别计算基础矩阵和单应矩阵的重投影误差的卡方分布来计算两个矩阵的得分]-->
4[根据得分选择效果更好的矩阵]
```
在计算单应矩阵和基础矩阵时，需要先分别对每幅图像的像点进行归一化。
**归一化的好处**：
1. 提高结果的精度
2. 归一化后的图像坐标对任何尺度缩放和坐标原点的选择不变，归一化步骤通过为测量数据选择有效的标准坐标系，预先消除了坐标系变换的影响。

具体步骤：
1. 对点进行平移使其形心位于原点：
$$
初始像点为：x_1, x_2, ..., x_N \\
像点求均值为：\overline u = \frac{\sum x_i}{N} \\
像点减去均值：u_i ^ \prime = x_i - \overline u
$$
2. 对点进行缩放使它们到原点的平均距离等于$\sqrt{2}$
$$
把减去均值的点求均值，得到偏离均值的程度：\overline u^\prime = \frac{\sum u_i^\prime}{N} \\
其倒数作为缩放因子：s = \frac{1}{\overline u^\prime} \\
缩放后的点为：x_i^\prime = s * u_i^\prime \\
注意这里x, u, s都是二维的，对应像素坐标的两个分量。
$$
3. 对两幅图像独立进行上述变换
$$
把上述平移、缩放写成矩阵形式：\\
T = \left[
\begin{matrix}
s_x \quad 0 \quad -x_{mean}*s_x \\
0 \quad s_y \quad -y_{mean}*s_y \\
0 \quad\quad\quad 0 \quad\quad\quad 1
\end{matrix}
\right]
$$

用归一化点计算的变换矩阵与实际的变换矩阵相差两个归一化变换:
$$
设两幅图像的初始点分别为：x、x^\prime,有x^\prime = Hx \\
归一化之后的对应点为：\overline x = Tx、\overline x^\prime=T^\prime x^\prime \\
则：T^{\prime-1}\overline x^\prime = HT^{-1}\overline x \Rightarrow \overline H = T^\prime H T^{-1} \\
\Rightarrow H = T^{\prime-1} \overline H T
$$

>关于归一化的详细说明，参考《多视几何》第二版的P78。

**单应矩阵和基础矩阵得分的计算**：
* 单应矩阵使用卡方分布的显著性值与**重投影点与原始像点的误差**的差作为得分
* 基础矩阵使用卡方分布的显著性值与**像点到对极线的距离**的差作为得分

### 1.2 双目初始化
```mermaid
graph TB
1[原始左图和右图]-->
2[根据标定参数对左右图进行立体匹配,得到去畸变且光轴平行,行坐标对齐的左图和右图]-->
3[分别提取左图和右图特征点]
```

## 特征提取 ORBextractor
```mermaid
classDiagram
direction RL
class ORBextractor{
# vector<cv::Point> pattern //计算描述子的模板
# int nfeatures //整个图像金字塔中，要提取的特征点数量
# double scaleFactor //图像金字塔中，层与层之间的缩放因子
# int nlevels //图像金字塔的层数
# iniThFAST //初始的FAST响应阈值
# minThFAST //最小的FAST响应阈值
# vector<int> mnFeaturesPerLevel //分配到每层图像中，要提取的特征点数量
# vector<int> umax //计算特征点方向时，有个圆形的图像区域，此vector存储每行u轴的边界
# vector<float> mvScaleFactor //每一层相对最底层的缩放因子
# vector<float> mvInvScaleFactor
# vector<float> mvLevelSigma2 //每一层缩放因子的平方
# vector<float> mvInvLevelSigma2
+ vector<cv::Mat> mvImagePyramid //图像金字塔
+ORBextractor(int nfeatures, float scaleFactor, int nlevels, int iniThFAST, int minThFAST)
+operator()(cv::InputArray image, cv::InputArray mask, std::vector<cv::KeyPoint>& keypoints,cv::OutputArray descriptors) void
#ComputePyramid(cv::Mat image) void
#ComputeKeyPointsOctTree(vector<vector<cv::KeyPoint>>& allKeypoints) void
#DistributeOctTree(const std::vector<cv::KeyPoint>& vToDistributeKeys, const int &minX, const int &maxX, const int &minY, const int &maxY, const int &nFeatures, const int &level) vector<cv::KeyPoint>
}

class ExtractorNode{
+ vector<cv::KeyPoint> vKeys //当前节点的特征点
+ cv::Point2i UL, UR, BL, BR //当前节点对应的图像块边界
+ list<ExtractorNode>::iterator lit
+ bool bBoMore //如果此节点只有一个特征点，设为true
+ExtractorNode()
+DidideNode(ExtractorNode &n1, ExtractorNode &n2, ExtractorNode &n3, ExtractorNode &n4) void
}
```
特征提取的入口函数为重载的operator()，这是一个仿函数。
```mermaid
graph TB
1(operator入口开始) -->
2[ComputePyramid:每一层都需要进行补边操作,目的是提取图像边缘的特征点]-->
3[ComputeKeyPointsOctTree:提取特征点,并进行均匀操作]-->
4[对图像进行高斯模糊]-->
5[在高斯模糊后的图像上计算描述子]-->
6[对非第0层图像中的特征点坐标恢复到第0层坐标系下]-->
7(结束)
```

## MapPoint
```mermaid
classDiagram
class MapPoint{
+long unsigned int mnId //MapPoint的全局ID
+static long unsigned int nNextId
+const long int mnFirstKFid //创建该MapPoint的关键帧ID
+const long int mnFirstFrame //创建该MapPoint的帧ID
+int nObs //被观测到的相机数目，单目+1，双目或RGB-D则+2
+float mTractProjX //用于tracking,投影到某帧上的坐标
+float mTrackProjY //同上
+float mTrackProjXR //右目
+int mnTrackScaleLevel //所处的尺度
+float mTrackViewCos //被追踪到时，那帧相机看到当前地图点的视角
+bool mbTrackInView //SearchByProjection中决定是否对改点进行投影的变量
+long unsigned int mnTrackReferenceForFrame //UpdateLocalPoints中防止将MapPoints重复添加至mvpLocalMapPoints的标记
+long unsigned int mnLastFrameSeen //SearchLocalPoints中决定是否进行isInFrustum判断的变量
+mnBALocalForKF
+mnFuseCandidateForKF
+mnLoopPointForKF
+mnCorrectedByKF
+mnCorrectedReference
+cv::Mat mPosGBA
+mnBAGlobalForKF
#cv::Mat mWorldPos
#map<KeyFrame*, size_t> mObservations
#cv::Mat mNormalVector
#cv::Mat mDescriptor
#KeyFrame* mpRefKF
#int mnVisible
#int mnFound
#bool mbBad
#MapPoint* mpReplaced
#float mfMinDistance
#float mfMaxDistance
#Map* mpMap
}
```


