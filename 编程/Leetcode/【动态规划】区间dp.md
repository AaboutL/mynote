---
title: 【动态规划】区间dp
updated: 2023-02-23 07:00:44Z
created: 2023-02-23 06:53:21Z
latitude: 39.91430000
longitude: 116.38610000
altitude: 0.0000
---

区间dp就是在一个序列范围内选择最值。可能选择区间的左边，也可能选择右边，也可能从中间选一个。一次选择结束后，再从剩下的区间中进行选择。
<font color=red>状态定义 dp\[i]\[j]：表示的是在区间$[i,j]$之间的最优选择的结果。
状态更新的过程为把 i 和 j 向中间缩小的过程。</font>

## 问题示例：
### 1.【375. 猜数字大小 II】https://leetcode.cn/problems/guess-number-higher-or-lower-ii/
### 2. 【877. 石子游戏】https://leetcode.cn/problems/stone-game/