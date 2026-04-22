---
title: 【3dmm】3DDFA信息梳理
updated: 2023-06-27 10:07:47Z
created: 2023-06-27 02:36:34Z
latitude: 39.90421100
longitude: 116.40739500
altitude: 0.0000
---

# 欧拉角和旋转矩阵的转换
## 1. matlab版代码
**HPEN**（3dmm参数拟合）：旋转矩阵和欧拉角的转换使用Mike Day的方式，参考文章（Extracting Euler Angles from a Rotation Matrix）
**FaceProfilingRelease_v1.1**（侧脸增强）：同样是Mike Day的方式

## 2. face3d（python代码）
使用的是右手坐标系、Tait-Bryan angles的外旋x-y-z（或内旋z-y-x）

## 3. 3DDFA（guojianzhu）
根据代码中estimate_pose.py文件中的matrix2angle(R)函数的计算方式（参考了Computing Euler angles from a rotation matrix-G.Slabaugh），这个R应该是Tait-Bryan中的$Z_1Y_2X_3$的形式。
但是3DDFA使用的训练集又是300W-LP，300W-LP是matlab的代码拟合出来的，其旋转矩阵转成欧拉角用的是Mike Day的形式。
Tait-Bryan和Mike Day的旋转矩阵的关系：先转置，然后按照斜对角线翻转。
<font color=RED>难道是guojianzhu 对300W-LP重新进行了拟合，使用Tait-Bryan的形式？</font>
作者在issue166中提到，推荐使用face3d的方法拟合参数：https://github.com/YadiraF/face3d/blob/master/face3d/morphable_model/fit.py.

### 验证guojianzhu和zhuxiangyu的参数的pose是否一样
在guojianzhu的3DDFA（尺寸120）数据中选一张图片和其对应的param文件，从param的pose中分离出R
在300W-LP中选同一张图片和其mat文件，从mat中根据pitch、yaw、roll恢复出R
比较两个R，发现是一样的，没有转置关系。


## 4. face_pose_augmentation（python版侧脸增强）
代码使用3DDFA模型推理出来的参数，所以其旋转矩阵与3DDFA一致

# pitch yaw roll 与面部朝向的关系
| 数据 | pitch | yaw | roll |
| -- | -- | -- | -- |
|300W-LP | 负：低头；正：抬头 | 负：朝右；正：朝左 | 负：朝左；正：朝右|
