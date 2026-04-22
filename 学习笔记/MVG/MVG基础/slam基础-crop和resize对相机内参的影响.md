---
title: slam基础-crop和resize对相机内参的影响
updated: 2023-04-14 11:43:00Z
created: 2023-04-14 11:32:55Z
latitude: 39.90421100
longitude: 116.40739500
altitude: 0.0000
---

链接： https://blog.csdn.net/u010772377/article/details/118388880

![7e782b9da72cb45767f5406ae123e0ed.png](../../../_resources/7e782b9da72cb45767f5406ae123e0ed.png)
![206bc8e2e411164614f296976be6b4e7.png](../../../_resources/206bc8e2e411164614f296976be6b4e7.png)
![e2398af3bb22b58dcd870b6173f065cb.png](../../../_resources/e2398af3bb22b58dcd870b6173f065cb.png)
![ad6189134f7f4b064ee2bc1c19e2ec4a.png](../../../_resources/ad6189134f7f4b064ee2bc1c19e2ec4a.png)

总结起来：
**crop**只对$c_x, c_y$有影响：$c_{new} = c_{old} - (x_{old} - x_{new})$
**resize**对$c_x, c_y, f_x, f_y$都有影响：都按照图像的缩放比例进行缩放。