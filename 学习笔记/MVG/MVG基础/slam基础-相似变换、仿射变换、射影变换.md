---
title: slam基础-相似变换、仿射变换、射影变换
updated: 2022-07-27 06:52:22Z
created: 2022-05-19 11:20:05Z
---

# 2D空间中的变换

1.  等距变换：

<div align="center"><img src="../../../_resources/293c15e64c2f2956a6a9d702d34d7e34.png" alt="293c15e64c2f2956a6a9d702d34d7e34.png" width="272" height="70" class="jop-noMdConv"></div>当ε为1时，是欧式变换；ε为-1时，会出现歧义情况 欧式变换有三个自由度：1个旋转，两个平移

2.  相似变换：

<div align="center"><img src="../../../_resources/b2bed33d9bb1486d173c98a8b6338c57.png" alt="b2bed33d9bb1486d173c98a8b6338c57.png" width="319" height="78" class="jop-noMdConv"></div>

相似变换有4个自由度：1个旋转、1个尺度、2个平移

3.  仿射变换

<div align="center"><img src="../../../_resources/a1561a28931aa0f8185248418882ea95.png" alt="a1561a28931aa0f8185248418882ea95.png" width="295" height="80" class="jop-noMdConv"> <img src="../../../_resources/6f6c21e20b1572b2b27ed2dc12b7befd.png" alt="6f6c21e20b1572b2b27ed2dc12b7befd.png" width="250" height="70" class="jop-noMdConv"></div>

仿射变换有6个自由度，对应了矩阵中的6个元素。
可以把A看做是两个基础变换的组合：旋转和非各向同性的缩放的组合

<div align="center"><img src="../../../_resources/b895b0f51fbdf567e8ff1f8373e3e695.png" alt="b895b0f51fbdf567e8ff1f8373e3e695.png" width="295" height="46" class="jop-noMdConv"> <img src="../../../_resources/9971bdd7283903dbfa39a88d6eb710af.png" alt="9971bdd7283903dbfa39a88d6eb710af.png" width="148" height="63" class="jop-noMdConv"></div>

4.  射影变换
    <div align="center"><img src="../../../_resources/aca0051091c69c271fc4281a9f23bcf7.png" alt="aca0051091c69c271fc4281a9f23bcf7.png" width="218" height="70"></div>

射影变换有8个自由度，两个平面的射影变换可以通过4对点计算，任意三个点不共线。

# 3D空间中的变换

1.  欧式变换：相当于是平移变换（t）和旋转变换（R）的复合，等距变换前后长度，面积，线线之间的角度都不变。自由度为6（3+3）
2.  相似变换：等距变换和均匀缩放（S）的一个复合，类似相似三角形，体积比不变。自由度为7（6+1）
3.  仿射变换：一个平移变换（t）和一个非均匀变换（A）的复合，A是可逆矩阵，并不要求是正交矩阵，仿射变换的不变量是:平行线，平行线的长度的比例，面积的比例。自由度为12（9+3）
4.  射影变换：当图像中的点的齐次坐标的一般非奇异线性变换，射影变换就是把理想点（平行直线在无穷远处相交）变换到图像上，射影变换的不变量是:重合关系、长度的交比。自由度为15（16-1）