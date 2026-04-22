---
title: 【vscode】使用sys.path.append添加的路径无法被识别
updated: 2023-08-03 09:32:01Z
created: 2023-08-03 09:25:31Z
latitude: 39.90421100
longitude: 116.40739500
altitude: 0.0000
---

## 问题：

```
sys.path.append("/opt/MVS/Samples/64/Python/MvImport")
from MvCameraControl_class import *
```

上面代码添加了路径“/opt/MVS/Samples/64/Python/MvImport”，但是这个路径下的模块MvCameraControl_class无法被代码提示工具（如pylance、pylint等）识别。

## 解决方法：

把上面要添加的路径加入到settings.json的python.analysis.extraPaths中
<img src="../_resources/b2c00b4742278ab6b0eb35194772a728.png" alt="b2c00b4742278ab6b0eb35194772a728.png" width="910" height="285">