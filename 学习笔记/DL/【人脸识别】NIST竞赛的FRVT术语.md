---
title: 【人脸识别】NIST竞赛的FRVT术语
updated: 2023-09-28 03:29:18Z
created: 2023-09-28 03:18:40Z
latitude: 39.90421100
longitude: 116.40739500
altitude: 0.0000
---

gallery：底库  
probe/query：搜索数据  
**FNIR**(False negative identification rate)：

| 术语  | 全称  | 说明  |
| --- | --- | --- |
| gallery | \-  | 底库数据 |
| probe/query | \-  | 用来检索的数据 |
| FNIR | False negative identification rate |检索图在底库中有对应的id，但是在阈值为T的topK中，没有这个id。这些数据的比例。|
| FPIR | False positive identification rate |检索图在底库中没有对应的id，但是在阈值为T时，却找到了一个或多个匹配。这种数据的比例|
| Selectivity | 

