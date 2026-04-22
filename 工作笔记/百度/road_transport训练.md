---
title: road_transport训练
updated: 2021-12-22 03:32:52Z
created: 2021-12-20 03:21:42Z
latitude: 39.92850000
longitude: 116.38500000
altitude: 0.0000
---

|日期| 训练任务 | 机器 | 训练代码                   | 配置 | 进度 |
|--| --     | --   | --                       | --   | --  | 
|20211220|字段检测 |v100  | endet_s2                 | input=512,bs=8,gpu=2  | 工具有问题    |
|20211220|四方向   | v100 | direction_classification | input=256,bs=96,gpu=1 | 利用率低kill |
|20211220|主体检测 |v100  | Endet_s2                 | input=256,bs=8,gpu=1  |利用率低kill| 
|20211222|字段检测 |p40  | train_multiclass_detector | input=256,bs=8,gpu=1  |data_aug中| 
|20211222|四方向  | p40| direction_classification | input=256,bs=96,gpu=1 | 训练中 |
|20211220|主体检测 |p40| Endet_s1               | input=256,bs=8,gpu=1  |data_aug中| 