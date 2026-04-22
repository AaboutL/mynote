---
title: ocr badcase解决率指标
updated: 2022-01-12 09:55:58Z
created: 2022-01-11 08:34:45Z
latitude: 39.92850000
longitude: 116.38500000
altitude: 0.0000
---

# ocr 字段检测badcase解决率指标

假设旧模型badcase有M张图，当前垂类有N个字段

1.  图的解决率：
    新模型完全做对的有K张图，新模型图解决率
    
    $$
    R_{fixed-img} = \frac{K}{M}
    
    $$
    
2.  字段解决率：
    对于第$i$个字段，旧模型错误数量为$w_{old}^i$，正确数量为$r_{old}^i$；
    旧模型正确但是**新模型错误**数量为$w_{new}^i$，
    旧模型错误但是**新模型正确**数量为$r_{new}^i$，则
    引入错误率为：$w_{new}^i / r_{old}^i$
    剔除错误率为：$r_{new}^i / w_{old}^i$
    当前字段错误解决率：$(1 - \frac{w_{new}^i}{r_{old}^i}) \times \frac{r_{new}^i}{w_{old}^i}$
    所有字段的平均错误解决率：

$$
		R_{fixed-ziduan} = \frac{\sum_{i=1}^{N}(1 - \frac{w_{new}^i}{r_{old}^i}) \times \frac{r_{new}^i}{w_{old}^i}}{N}

$$

3.  单图字段解决率
    对于第$j$张图，旧模型错误字段数量为$w_{old}^j$，正确数量为$r_{old}^j$；
    旧模型正确但是**新模型错误**数量为$w_{new}^j$，
    旧模型错误但是**新模型正确**数量为$r_{new}^j$，则
    引入错误率为：$w_{new}^j / r_{old}^j$
    剔除错误率为：$r_{new}^j / w_{old}^j$
    当前图错误解决率：$(1 - \frac{w_{new}^j}{r_{old}^j}) \times \frac{r_{new}^j}{w_{old}^j}$
    所有图的平均错误解决率：

$$
		R_{fixed-img} = \frac{\sum_{j=1}^{M}(1 - \frac{w_{new}^j}{r_{old}^j}) \times \frac{r_{new}^j}{w_{old}^j}}{M}

$$

4.  badcase集合上旧模型endet和端到端指标
5.  badcase集合上新模型endet和端到端指标