---
title: 【Simulation Framework】ManiSkill2
updated: 2023-10-16 08:50:33Z
created: 2023-10-16 07:13:34Z
latitude: 39.90421100
longitude: 116.40739500
altitude: 0.0000
---

MANISKILL2: A UNIFIED BENCHMARK FOR GENERALIZABLE MANIPULATION SKILLS

# soft-body 环境对比
|名称 | 方法| 说明 | 
| -- | -- | -- |
|MuJoCo |  FEM(finite element methods) | 支持rope, cloth, and elastic objects的仿真，但是不能处理大变形、拓扑变化，比如舀面粉或切面团| 
|Bullet | 同上 | 同上 | 
|SoftGym | 基于Nvidia Flex|支持大变形，但不支持弹性塑料材质 |
|ThreeDWorld | 同上 | 同上|
|PlasticineLab | 支持基于连续体力学的材料点方法（MPM）| 不能与刚性机器人配合，而且仿真和渲染性能有待提高|
## ManiSkill2
1. 使用Nvidia Warp JIT框架和原生CUDA从头实现了基于GPU的MPM仿真，高效方便定制；
2. 扩展了Nvidia Warp的功能，支持更高效的主机-设备之间的通信
3. 支持双向动力学耦合接口，使任何刚体模拟框架都能与软体交互，允许ManiSkill2中的机器人和资产与软体模拟无缝交互（第一个支持的框架）