---
title: 【opencv】cv2.imshow显示图像出现QObject::moveToThread
updated: 2023-06-02 12:23:53Z
created: 2023-06-02 12:19:36Z
latitude: 39.90421100
longitude: 116.40739500
altitude: 0.0000
---

使用python版opencv显示图像时，出现如下提示：
> QObject::moveToThread: Current thread (0x2bfa3f0) is not the object's thread (0x301bfb0).
Cannot move to target thread (0x2bfa3f0)

原因：使用了anaconda3来管理python包，而且，opencv和pyqt5安装的时候用的不是同一个管理工具，如opencv使用pip安装，pyqt5使用conda安装，或者相反。
解决方法：删除已经安装的包，然后使用同一个管理工具安装两个包。