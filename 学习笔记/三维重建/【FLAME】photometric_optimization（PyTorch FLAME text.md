---
title: 【FLAME】photometric_optimization（PyTorch FLAME texture fitting）
updated: 2023-07-13 09:52:18Z
created: 2023-07-13 07:47:56Z
latitude: 39.90421100
longitude: 116.40739500
altitude: 0.0000
---

链接：https://github.com/HavenFeng/photometric_optimization
# 1. 用途
这个仓库提供了一个合成分析（analysis-by-synthesis）框架用来把带纹理的FLAME模型拟合到一张图像上。
FLAME提供了head 模型，但是没有提供appearance space
1. 描述了如何用in-the-wild图像为FLAME创建Texture Space
2. 提供了拟合Texture space的代码，优化了FLAME的参数、外观、光照
3. 用于优化 FLAME 纹理以匹配 in-the-wild 图像的代码

# 2. 流程
从FFHQ dataset中随机筛选了1500张图片
## 2.1 初始化
从in-the-wild图像中生成texture space 是一个鸡生蛋蛋生鸡的问题。给定texture space，它可以被用于analysis-by-synthesis
为了获得初始的texture space，把FLAME拟合到BFM的模板上，然后把BFM顶点的颜色投影到FLAME的mesh上，得到初始的texture basis。
## 2.2 Model fitting
把FLAME拟合到FFHQ dataset的图像上，优化FLAME的：
1. 形状（shape）、姿态（pose）、表情（expression）参数；
2. 初始 texture space 的参数
3. 球谐波光照（Spherical Harmonics (SH) lighting）参数，(we optimize for 9 SH coefficient only, shared across all three color channels)；
4. 偏离初始 texture space 的偏移，（能够反映texture的细节信息）

拟合的loss包括：
1. 关键点距离loss
2. 光度损失（photometric loss）：photometric loss is optimized for the skin region only to gain robustness to partial occlusions
3. shape, pose, expression, appearance, and the texture offset的正则项

## 2.3 Texture completion
拟合完成后，计算出来的 texture offset 只能表示没有被遮挡的区域。
对于遮挡的地方，通过训练 GMCNN 模型，来修复。

## 2.4 Texture space computation
当1500个texture maps 计算完成后，通过PCA的方式计算 texture space。

