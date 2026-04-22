---
title: 【对比学习】in-batch negatives
updated: 2023-12-09 07:34:26Z
created: 2023-12-09 07:23:00Z
latitude: 39.90421100
longitude: 116.40739500
altitude: 0.0000
tags:
  - embedding
  - 对比学习
---

一般用于文本检索、开放域问答训练
假设batch_size大小为B，每个question有1个positive和 n 个negative，那么一个batch内就有 $B \times (n + 1)$ 个passages 要经过encoding，一般需要比较多的negatives 才能保证较好的效果，所以仅仅encoding的计算开销就很大。
解决方法：只用 B 个questions 和 其对应的 B 个passages，形成两个矩阵 $Q 和 P$， 然后一个question的passage 可以作为其他 question的negative。这样只需要在一个batch内组成negative，然后直接计算$S = QP^T$ 就可以得到所有样本对的相似度，其中 S 中的每一行代表一个question与batch内的所有passages的相似度，包含一个正样本相似度和（B-1）个负样本相似度。这样，一个batch内只有B个positive passages需要经过encoding，比上面的方法减少了n倍。
**补充：负样本的选择原则：**
1. Random：从全体文档中简单随机采样
2. BM25：不包含答案，但与query的BN25匹配度相当高的样本
3. Gold：训练集中，其他query所对应的正样本。