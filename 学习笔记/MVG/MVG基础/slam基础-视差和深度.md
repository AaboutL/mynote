---
title: slam基础-视差和深度
updated: 2022-08-31 03:07:42Z
created: 2022-05-20 02:33:30Z
latitude: 39.91175842
longitude: 116.37922668
altitude: 0.0000
---

# slam面试-视差和深度

<div align="center"><img src="../../../_resources/1d4ef59a5e7efa4747746969321c43d8.png" alt="1d4ef59a5e7efa4747746969321c43d8.png" width="603" height="407" class="jop-noMdConv"></div>

双目相机需要提前标定，并把成像平面对齐
如右图，两相机光心距离为baseline：$b=O_LO_R$，焦距为f，像点$P_L$的x坐标为$u_L$，像点$P_R$的x坐标为$-u_R$，求3D点P的深度z。
根据相似三角形，有：

$$
\frac{P_LP_R}{b} = \frac{z-f}{z} \\
=>zP_LP_R = zb - fb \\
=>fb = z(b - P_LP_R) \\
=>z = \frac{fb}{b-P_LP_R} = \frac{fb}{u_L - u_R}

$$

* * *

<img src="../../../_resources/1cb38479309a8be4be3ee6fb588a242c.png" alt="1cb38479309a8be4be3ee6fb588a242c.png" width="679" height="585">