---
title: slam基础-重投影误差关于位姿的雅可比推导
updated: 2024-09-02 06:15:41Z
created: 2022-11-19 17:56:47Z
latitude: 39.90421100
longitude: 116.40739500
altitude: 0.0000
---

链接：
| 标题 | 说明 | 链接 | 
|-- | -- | -- |
| 视觉SLAM位姿优化时误差函数雅克比矩阵的计算 | 直接用流形上的扰动来对位姿求导 | https://blog.csdn.net/u011178262/article/details/85016981 | 
| SLAM优化位姿时，误差函数的雅可比矩阵的推导 | 先三维点先对SO3（李群）求导，然后SO3再对se3（李代数）求导，与matlab的toolbox_calib包一致 |https://blog.csdn.net/zhubaohua_bupt/article/details/74011005 | 
|SLAM学习笔记（四）Bundle Adjustment 重投影误差模型及相应雅克比公式推导 | 包含了GN优化的流程 |https://zhuanlan.zhihu.com/p/482540286 | 


<img src="../../../_resources/7c65d136181542df5a44f36c4c730fa2.png" alt="7c65d136181542df5a44f36c4c730fa2.png" width="640" height="591" class="jop-noMdConv"> <img src="../../../_resources/3c43a0fb9cfac9babe3dc25a4c23992a.png" alt="3c43a0fb9cfac9babe3dc25a4c23992a.png" width="640" height="588" class="jop-noMdConv"><img src="../../../_resources/8aec84aa837a6c6ab0b98b38464fbcbb.png" alt="8aec84aa837a6c6ab0b98b38464fbcbb.png" width="640" height="663" class="jop-noMdConv"><img src="../../../_resources/56173c5db8b6d98a66617fdc17488f2d.png" alt="56173c5db8b6d98a66617fdc17488f2d.png" width="640" height="666" class="jop-noMdConv"><img src="../../../_resources/5c09f96db427cdc831fe90e341e62aab.png" alt="5c09f96db427cdc831fe90e341e62aab.png" width="640" height="696" class="jop-noMdConv"><img src="../../../_resources/9f5fc73d76e161fbd777a3e50f792229.png" alt="9f5fc73d76e161fbd777a3e50f792229.png" width="640" height="645" class="jop-noMdConv">