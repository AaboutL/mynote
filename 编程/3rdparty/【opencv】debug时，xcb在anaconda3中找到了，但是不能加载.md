---
title: 【opencv】debug时，xcb在anaconda3中找到了，但是不能加载
updated: 2023-06-07 09:16:06Z
created: 2023-06-07 09:05:07Z
latitude: 39.91014100
longitude: 116.35732000
altitude: 0.0000
---

问题：
> qt.qpa.plugin: Could not load the Qt platform plugin "xcb" in "/home/ts/anaconda3/lib/python3.10/site-packages/cv2/qt/plugins" even though it was found.
This application failed to start because no Qt platform plugin could be initialized. Reinstalling the application may fix this problem.

解决方法：
当您将 opencv 与 pyqt5 --- 一起使用时，会出现此问题。看起来 opencv 内部使用的 qt 插件与 pyqt5 不兼容。只需取消设置环境变量 QT_QPA_PLATFORM_PLUGIN_PATH 在 import cv2 语句之后。
~~~python
import os
import cv2
os.environ.pop("QT_QPA_PLATFORM_PLUGIN_PATH")
~~~