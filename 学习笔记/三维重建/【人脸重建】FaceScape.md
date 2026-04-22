---
title: 【人脸重建】FaceScape
updated: 2023-09-20 13:18:54Z
created: 2023-09-02 14:07:39Z
latitude: 39.90421100
longitude: 116.40739500
altitude: 0.0000
---

# 论文

## 1\. registration

FaceScape数据集的注册过程分两步：

1.  **基础形状的注册**：对于原始mesh做下采样，得到更少三角面片的粗糙mesh（base shape），然后为简化的mesh创建3dmm。
    - 首先，从正脸图片中提取2D landmark，然后通过逆投影2D landmark得到对应的3D landmark。通过Procrustes transformation方法把所有的landmark注册到标准3D人脸模板上，这样，所有scanned meshes的pose和scale都粗略地对齐到标准人脸模板上。
    - 然后，通过Non-rigid ICP 方法把标准人脸模板注册到中性表情的scanned mesh上。对于19个其他表情，首先使用deformation transfer algorithm 把中性表情上注册的mesh变形到其他表情，来模拟一组对应表情的模板网格的形变；然后使用Non-rigid ICP把这些变形的个体相关的模板注册到scanned meshes，来更准确地拟合非中性表情的scans。
2.  **位移图生成**：在获得拓扑一致的基础形状后，使用UV空间中的位移图表示基础形状中由于降采样而缺失的中和细尺度的细节。
    - 首先，使用Laplacian mesh smoothing方法对原始scans进行平滑。
    - 然后，跟踪与位移图中像素对应的基网格的表面点，通过把这些点沿着法线方向逆投影到原始mesh，找到对应的点。位移图中的像素值设置为这些点从原始mesh到对应的平滑mesh之间的有符号的距离。

# 备注

1.  注册过程（Registration of base shape）  
    issue：https://github.com/zhuhao-nju/facescape/issues/7 说与 lsfm（[https://github.com/menpo/lsfm）流程类似](https://github.com/menpo/lsfm%EF%BC%89%E6%B5%81%E7%A8%8B%E7%B1%BB%E4%BC%BC)
2.  重建：参考了下面两篇论文：
    1.  Accurate, Dense, and Robust Multi-View Stereopsis, PAMI2010 （[https://www.di.ens.fr/pmvs/）](https://www.di.ens.fr/pmvs/%EF%BC%89)
    2.  High-Quality Single-Shot Capture of Facial Geometry, SIGGRAPH2010
