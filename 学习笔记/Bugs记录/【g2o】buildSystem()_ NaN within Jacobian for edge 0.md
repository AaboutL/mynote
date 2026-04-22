---
title: '【g2o】buildSystem(): NaN within Jacobian for edge 0xaaab1d321db0 for vertex 0'
updated: 2022-10-09 07:23:47Z
created: 2022-08-26 03:03:00Z
latitude: 39.91014100
longitude: 116.35732000
altitude: 0.0000
---

### 原因
1. 顶点没有给初值，就添加到优化器中了。
2. 二元边的两个顶点编号设置错误，下面是正确的设置方式
	~~~
	e->setVertex(0, dynamic_cast<g2o::OptimizableGraph::Vertex *>(optimizer.vertex(vertexIdx)));
	e->setVertex(1, dynamic_cast<g2o::OptimizableGraph::Vertex *>(optimizer.vertex(i)));
	~~~
	上面是正确的设置方法，setVertex()函数的第一个参数：0表示设置空间点，1表示设置pose。