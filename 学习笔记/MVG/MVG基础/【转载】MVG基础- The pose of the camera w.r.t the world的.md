---
title: 【转载】MVG基础- The pose of the camera w.r.t the world的含义
updated: 2023-06-04 09:21:36Z
created: 2023-06-01 09:55:58Z
latitude: 39.90421100
longitude: 116.40739500
altitude: 0.0000
---

链接：[https://miaodx.com/blogs/unrealcv\_digest/camera\_pose/](https://miaodx.com/blogs/unrealcv_digest/camera_pose/)
链接中引用了opencv和openMVG的解释，都表达了同一个意思

1.  opencv 的solvePnP中的解释为：
    
    > Output rotation vector (see Rodrigues ) that, together with tvec, brings points from the model coordinate system to the camera coordinate system.
    
    solvePnP的输出rvec/tvec为：<span style="color: red;">把点从model坐标系转换到相机坐标系，即$P_C = T_{CM}P_M$</span>
    
2.  openMVG的解释：
    
    > R: the rotation of the camera to the world frame,
    > t: the translation of the camera. t is not the position of the camera. It is the position of the origin of the world coordinate system expressed in coordinates of the camera-centred coordinate system. The position, C, of the camera expressed in world coordinates is C=−R−1t=−RTt (since R is a rotation matrix).
    
    R: 从camera转到world的旋转矩阵
    t: camera的平移。t不是camera的坐标，它是world系的原点在camera系的坐标
    camera的坐标为：$C=-R^Tt$
    

## <span style="color: red;">The pose of the camera w.r.t the world 指的是</span>:

<span style="color: blue;">$R|t$ bring points from the camera coordinate system to the world coordinate system.点从相机坐标系转换到世界坐标系</span>
字面意思理解：camera关于world的位姿，camera相对world的位姿，也就是camera在world系中的姿态是什么样的，也就是把world坐标系变换到camera坐标系，所以从坐标系变换来说，这个pose就是$^{c}F =\ ^{w}FT_{wc}$。