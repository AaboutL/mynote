# Flow Matching 原理详解及其在机器人 VLA 中的应用

> 适合对 Flow Matching 完全不了解的读者。

---

## 目录

1. [为什么要有 Flow Matching](#1-为什么要有-flow-matching)
2. [连续变形：从分布到动力系统](#2-连续变形从分布到动力系统)
3. [为什么 ODE 能推动"分布"变化](#3-为什么-ode-能推动分布变化)
4. [Flow Matching 到底学什么](#4-flow-matching-到底学什么)
5. [最简单版本：直线路径](#5-最简单版本直线路径)
6. [为什么只看中间点就够了](#6-为什么只看中间点-xt-就够了)
7. [更一般的 Flow Matching 公式](#7-更一般的-flow-matching-公式)
8. [它和 Diffusion 是什么关系](#8-它和-diffusion-是什么关系)
9. [为什么 FM 适合机器人 / VLA](#9-为什么-fm-适合机器人--vla)
10. [Flow Matching 在机器人 VLA 中的标准建模方式](#10-flow-matching-在机器人-vla-中的标准建模方式)
11. [数据处理：VLA 里怎么准备训练数据](#11-数据处理vla-里怎么准备训练数据)
12. [训练：VLA 里怎么用 Flow Matching](#12-训练vla-里怎么用-flow-matching)
13. [推理：机器人执行时怎么生成动作](#13-推理机器人执行时怎么生成动作)
14. [完整的 VLA + FM 训练 / 推理流程](#14-完整的-vla--fm-训练--推理流程)
15. [它和"普通回归动作"相比强在哪](#15-它和普通回归动作相比强在哪)
16. [实际落地时最关键的工程点](#16-实际落地时最关键的工程点)
17. [你可以怎样理解 FM 在 VLA 中的角色](#17-你可以怎样理解-fm-在-vla-中的角色)
18. [一句话总结整套数学逻辑](#18-一句话总结整套数学逻辑)
19. [最简公式总表](#19-最简公式总表)

---

## 先给结论

Flow Matching（FM）本质上是在学一个**连续时间的速度场**。这个速度场告诉你：如果一个点当前在位置 $x$、时间在 $t$，它下一瞬间应该往哪里走、走多快。训练好后，从一个简单分布（通常是高斯噪声）出发，沿着这个速度场积分一个常微分方程（ODE），就能把噪声"流"成目标样本。

在机器人 **VLA（Vision-Language-Action）** 里，它通常不是用来生成图像，而是用来生成**连续动作序列**：给定视觉观测、语言指令和机器人状态，FM 学习如何把随机噪声逐步变成一段合理的动作轨迹。

---

## 1. 为什么要有 Flow Matching

设真实数据分布为 $p_1(x)$，这里的 $x$ 可以是图像、视频片段、机器人未来动作序列等。  
我们希望从这个分布中采样，但 $p_1(x)$ 通常非常复杂，直接采样很难。

于是引入一个容易采样的简单分布 $p_0(x)$，通常取：

$$
p_0(x) = \mathcal{N}(0, I)
$$

目标变成：

> 学一个机制，把来自 $p_0$ 的样本逐渐变成来自 $p_1$ 的样本。

Flow Matching 的思路是：不直接学"从噪声一步跳到数据"，而是学一个**连续变形过程**。

---

## 2. 连续变形：从分布到动力系统

假设样本 $x_t$ 随时间 $t \in [0, 1]$ 连续变化，满足 ODE：

$$
\frac{d x_t}{dt} = v_t(x_t)
$$

这里：

- $x_t$：时间 $t$ 时刻的样本位置
- $v_t(x)$：速度场（vector field）

如果你知道了这个速度场，那么：

1. 先从初始分布 $p_0$ 采样一个点 $x_0$
2. 解上面的 ODE
3. 得到终点 $x_1$

如果这个速度场学得对，那么 $x_1$ 的分布就会是目标分布 $p_1$。

---

## 3. 为什么 ODE 能推动"分布"变化

单个点 obey ODE，这很好理解。但我们真正关心的是：**一群点的分布**怎么变化。

设 $p_t(x)$ 表示时间 $t$ 时样本的概率密度，它满足一个守恒方程：

$$
\partial_t p_t(x) + \nabla \cdot \bigl(p_t(x) v_t(x)\bigr) = 0
$$

这叫**连续性方程**（continuity equation）。

### 3.1 连续性方程的推导

考虑空间中的任意一个区域 $B$，这个区域里的概率质量是：

$$
\int_B p_t(x)\, dx
$$

随着时间变化，这个概率质量只会因为"流"穿过边界而改变：

$$
\frac{d}{dt}\int_B p_t(x)\, dx
= -\int_{\partial B} p_t(x)\, v_t(x) \cdot n(x)\, dS
$$

其中 $\partial B$ 是区域边界，$n(x)$ 是外法向量，$p_t v_t \cdot n$ 是流出边界的概率流量。

用散度定理：

$$
\int_{\partial B} p_t v_t \cdot n\, dS
= \int_B \nabla \cdot (p_t v_t)\, dx
$$

所以：

$$
\frac{d}{dt}\int_B p_t(x)\, dx
= -\int_B \nabla \cdot (p_t(x) v_t(x))\, dx
$$

又因为：

$$
\frac{d}{dt}\int_B p_t(x)\, dx
= \int_B \partial_t p_t(x)\, dx
$$

于是得到：

$$
\int_B \left[\partial_t p_t(x) + \nabla \cdot (p_t(x) v_t(x))\right] dx = 0
$$

这对任意区域 $B$ 都成立，因此被积函数处处为 0：

$$
\boxed{
\partial_t p_t(x) + \nabla \cdot (p_t(x) v_t(x)) = 0
}
$$

这说明：**只要你定义了速度场，分布的演化就被完全决定了。**

---

## 4. Flow Matching 到底学什么

核心问题是：怎么训练出这个速度场 $v_t(x)$？

最直接的想法是：**人为构造一条"从噪声到数据"的路径，然后让模型去拟合这条路径上的真实速度。**

---

## 5. 最简单版本：直线路径

假设可以采样到：

- $x_0 \sim p_0$（噪声）
- $x_1 \sim p_1$（真实数据）

定义它们之间的一条直线：

$$
x_t = (1 - t) x_0 + t x_1, \quad t \in [0, 1]
$$

### 5.1 这条路径的速度是什么

对 $t$ 求导：

$$
\frac{d x_t}{dt}
= \frac{d}{dt}\bigl((1-t) x_0 + t x_1\bigr)
= -x_0 + x_1
$$

即真实速度为：

$$
u_t = x_1 - x_0
$$

注意这里速度和 $t$ 无关，因为直线匀速。

### 5.2 训练目标

让模型 $v_\theta(x, t)$ 在点 $x_t$ 处预测这个真实速度：

$$
\mathcal{L}(\theta)
= \mathbb{E}_{x_0, x_1, t}
\left[
\|v_\theta(x_t, t) - (x_1 - x_0)\|^2
\right]
$$

其中 $t \sim \mathrm{Uniform}(0, 1)$。

**最朴素的 Flow Matching 训练步骤：**

1. 采噪声 $x_0$
2. 采真实样本 $x_1$
3. 采时间 $t$
4. 算中间点 $x_t$
5. 算目标速度 $x_1 - x_0$
6. 用 MSE 回归

---

## 6. 为什么只看中间点 $x_t$ 就够了

训练时目标速度写成了 $x_1 - x_0$，看起来依赖整个配对 $(x_0, x_1)$。但推理时模型只知道 $x_t, t$，并不知道 $x_0, x_1$。为什么还能学？

关键在于**平方误差回归的最优解是条件期望**。

### 6.1 一步一步推导

记输入 $Z = (x_t, t)$，监督目标 $Y = u_t$，损失：

$$
\mathcal{L}(v) = \mathbb{E}\bigl[\|v(Z) - Y\|^2\bigr]
$$

按条件期望展开：

$$
\mathcal{L}(v)
= \mathbb{E}_Z
\left[
\mathbb{E}\bigl[\|v(Z) - Y\|^2 \mid Z\bigr]
\right]
$$

所以只要对每个固定的 $Z = z$ 最小化：

$$
\mathbb{E}\bigl[\|a - Y\|^2 \mid Z = z\bigr]
$$

其中 $a = v(z)$ 是待优化常数向量。展开平方：

$$
\|a - Y\|^2 = \|a\|^2 - 2a^\top Y + \|Y\|^2
$$

取条件期望：

$$
\mathbb{E}[\|a - Y\|^2 \mid Z = z]
= \|a\|^2 - 2a^\top \mathbb{E}[Y \mid Z = z] + \mathbb{E}[\|Y\|^2 \mid Z = z]
$$

对 $a$ 求导令其为 0：

$$
2a - 2\mathbb{E}[Y \mid Z = z] = 0
\implies a^* = \mathbb{E}[Y \mid Z = z]
$$

所以：

$$
\boxed{
v^*(x, t) = \mathbb{E}[u_t \mid x_t = x, t]
}
$$

这说明模型最终学到的是：**在所有可能经过该点 $x$ 的路径中，平均起来最合理的瞬时速度。** 这正是我们想要的"全局速度场"。

---

## 7. 更一般的 Flow Matching 公式

实际里常用更一般的路径：

$$
x_t = \alpha_t x_1 + \sigma_t \epsilon, \quad \epsilon \sim \mathcal{N}(0, I)
$$

这里：

- $x_1$：真实数据
- $\epsilon$：高斯噪声
- $\alpha_t, \sigma_t$：时间调度函数

于是条件分布为：

$$
p_t(x \mid x_1) = \mathcal{N}(\alpha_t x_1,\ \sigma_t^2 I)
$$

### 7.1 目标速度推导

从 $x_t = \alpha_t x_1 + \sigma_t \epsilon$ 对 $t$ 求导：

$$
\frac{d x_t}{dt} = \dot{\alpha}_t x_1 + \dot{\sigma}_t \epsilon
$$

由原式变形消去 $\epsilon$：

$$
\epsilon = \frac{x_t - \alpha_t x_1}{\sigma_t}
$$

代回：

$$
\frac{d x_t}{dt}
= \dot{\alpha}_t x_1 + \dot{\sigma}_t \frac{x_t - \alpha_t x_1}{\sigma_t}
$$

所以目标速度为：

$$
\boxed{
u_t(x_t \mid x_1)
= \dot{\alpha}_t x_1
+ \frac{\dot{\sigma}_t}{\sigma_t}(x_t - \alpha_t x_1)
}
$$

---

## 8. 它和 Diffusion 是什么关系

### 方法对比

| 方法 | 学什么 | 生成过程 |
|:---:|:---:|:---:|
| Diffusion | 去噪方向 / score $\nabla_x \log p_t$ | 逐步反向去噪，常带随机性 |
| Flow Matching | 连续速度场 $v_t(x)$ | 解一个 ODE，把样本流到目标分布 |

### 直观区别

- **Diffusion**：更像"逐步反复擦掉噪声"
- **Flow Matching**：更像"学一张全局导航地图，每个位置都有箭头指向正确方向"

### 为什么 FM 常被认为更直接

因为它把生成问题变成了一个标准监督学习问题：

$$
\text{输入：}(x_t, t) \quad \to \quad \text{输出：目标速度}
$$

而不是间接学习 score 再恢复动力学。

---

## 9. 为什么 FM 适合机器人 / VLA

机器人动作是连续量，常见形式如：

- 末端执行器位移 $(\Delta x, \Delta y, \Delta z)$
- 旋转
- gripper 开合
- 多步动作序列 $a_{t:t+H-1}$

这些量天然适合当作连续向量。Flow Matching 直接对**连续动作轨迹分布**建模，优势在于：

1. **能表达多模态动作**：同一任务可能有多种合法动作轨迹，FM 不是只回归均值
2. **输出是连续的**：不必把动作离散成 token
3. **采样步数可较少**：相比很多 diffusion policy，ODE 采样可以更快

---

## 10. Flow Matching 在机器人 VLA 中的标准建模方式

VLA 的目标是学一个条件分布：

$$
p(a_{t:t+H-1} \mid o_{\le t},\ s_{\le t},\ l)
$$

其中：

- $o_{\le t}$：视觉观测（图像/视频）
- $s_{\le t}$：机器人状态（关节角、速度、夹爪状态等）
- $l$：语言指令
- $a_{t:t+H-1}$：未来 $H$ 步动作块（action chunk）

令条件上下文为：

$$
c = \mathrm{Enc}(o_{\le t},\ s_{\le t},\ l)
$$

那么 FM 模型学的是条件速度场：

$$
v_\theta(x, \tau, c)
$$

其中 $\tau \in [0,1]$ 是 flow 时间，不是机器人真实时间。

---

## 11. 数据处理：VLA 里怎么准备训练数据

### 11.1 原始数据形式

机器人示教数据通常是轨迹：

$$
\mathcal{D} = \{(o_{1:T},\ s_{1:T},\ a_{1:T},\ l)\}
$$

一条轨迹包括：多相机图像序列、本体状态、动作、任务语言描述。

### 11.2 对齐与清洗

**常见预处理步骤：**

1. **时间对齐**：图像、状态、动作采样频率不同，需对齐到统一控制频率
2. **动作表示统一**：例如把绝对位姿改为相对位姿

   $$
   a_t = (\Delta p_t,\ \Delta r_t,\ g_t)
   $$

   其中 $g_t$ 是 gripper 状态

3. **动作归一化**：按维度做均值方差归一化

   $$
   \tilde{a} = \frac{a - \mu}{\sigma}
   $$

4. **构造 action chunk**：不直接预测一步，而是预测未来 $H$ 步

   $$
   A_t = [a_t,\ a_{t+1},\ \dots,\ a_{t+H-1}]
   $$

5. **视觉处理**：resize、crop、多相机拼接或分别编码
6. **语言处理**：用 tokenizer 或语言编码器转为 embedding

### 11.3 训练样本

每个训练样本最终变成：$(c,\ A^*)$

- $c$：由视觉、语言、状态编码后的条件
- $A^*$：目标动作块，形状 $H \times d_a$

FM 的任务是学 $p(A^* \mid c)$。

---

## 12. 训练：VLA 里怎么用 Flow Matching

### 12.1 选一条路径

设目标动作块为 $A^*$，采样噪声 $\epsilon \sim \mathcal{N}(0, I)$，定义中间状态：

$$
X_\tau = \alpha_\tau A^* + \sigma_\tau \epsilon
$$

最常见取法：

$$
\alpha_\tau = \tau, \qquad \sigma_\tau = 1 - \tau
$$

则：

$$
X_\tau = \tau A^* + (1 - \tau) \epsilon
$$

表示从噪声平滑走向真实动作。

### 12.2 推出监督目标

对 $\tau$ 求导：

$$
\frac{dX_\tau}{d\tau}
= \dot{\alpha}_\tau A^* + \dot{\sigma}_\tau \epsilon
= 1 \cdot A^* + (-1) \cdot \epsilon
= A^* - \epsilon
$$

所以监督目标是：

$$
U_\tau = A^* - \epsilon
$$

### 12.3 条件 Flow Matching 损失

模型输入：中间动作 $X_\tau$，flow 时间 $\tau$，条件上下文 $c$

模型输出：预测速度 $v_\theta(X_\tau, \tau, c)$

损失：

$$
\boxed{
\mathcal{L}(\theta) =
\mathbb{E}_{(c, A^*),\, \epsilon,\, \tau}
\left[
\|v_\theta(X_\tau, \tau, c) - U_\tau\|^2
\right]
}
$$

### 12.4 模型结构

VLA 中常见结构：

```
视觉观测  ──► 视觉编码器（ViT / CNN / Video Transformer）
语言指令  ──► 语言编码器（text transformer / LLM encoder）  ──► 融合主干（Transformer）──► c
机器人状态──► 状态编码器（MLP）

(X_τ, τ, c) ──► Flow Head ──► 预测速度（与动作同维度）
```

FM 通常不是整个 VLA 的全部，而是**动作生成头**。

---

## 13. 推理：机器人执行时怎么生成动作

### 13.1 从噪声开始

采样初值：

$$
X_0 \sim \mathcal{N}(0, I)
$$

### 13.2 解 ODE

求解：

$$
\frac{dX_\tau}{d\tau} = v_\theta(X_\tau, \tau, c)
$$

从 $\tau = 0$ 积分到 $\tau = 1$。

**Euler 离散化**：设步长 $\Delta\tau = 1/K$，则：

$$
X_{\tau_{k+1}}
= X_{\tau_k}
+ \Delta\tau \cdot v_\theta(X_{\tau_k}, \tau_k, c)
$$

迭代 $K$ 步后得到 $X_1$，视为生成动作块 $\hat{A}$。

### 13.3 反归一化

如果训练前做了标准化，执行前恢复：

$$
\hat{A}_{\text{real}} = \hat{A} \odot \sigma + \mu
$$

### 13.4 闭环执行

机器人通常不会一次把整个 chunk 全执行完，而是：

1. 生成未来 $H$ 步动作
2. 执行前 $k$ 步（如 1 步或几步）
3. 获取新观测
4. 重新规划

这叫 **receding horizon / MPC-style closed-loop control**。

---

## 14. 完整的 VLA + FM 训练 / 推理流程

### 14.1 训练流程

```
从示教轨迹切出时刻 t
    │
    ▼
取条件 c = (图像历史, 状态历史, 语言)
取目标动作块 A*
    │
    ▼
采样 ε ~ N(0, I)
采样 τ ~ U[0, 1]
    │
    ▼
构造中间状态  X_τ = α_τ A* + σ_τ ε
计算目标速度  U_τ = α̇_τ A* + σ̇_τ ε
    │
    ▼
最小化  ‖v_θ(X_τ, τ, c) - U_τ‖²
```

### 14.2 推理流程

```
编码当前观测 → 得到条件 c
    │
    ▼
采样初始噪声 X_0 ~ N(0, I)
    │
    ▼
ODE 积分（τ: 0 → 1）
    X_{τ+Δτ} = X_τ + Δτ · v_θ(X_τ, τ, c)
    │
    ▼
得到动作块 Â = X_1
    │
    ▼
反归一化 → 执行前 k 步
    │
    ▼
用新观测重复以上流程（闭环控制）
```

---

## 15. 它和"普通回归动作"相比强在哪

如果直接回归：

$$
\hat{A} = f_\theta(c)
$$

模型往往学到的是多种可行动作的"平均"，容易产生犹豫或不自然的动作。

而 FM 学的是条件分布 $p(A \mid c)$，所以能表示：

- 同一任务的多种抓取方式
- 不同绕障路径
- 不同起手姿态

这对真实机器人非常重要。

---

## 16. 实际落地时最关键的工程点

### 16.1 动作空间设计

FM 对动作表示很敏感。常见更稳定的表示：

- 相对位姿增量，而不是绝对位姿
- 旋转用 6D rotation / axis-angle，而不是直接欧拉角
- gripper 单独一维

### 16.2 归一化很重要

因为 FM 训练时把动作和高斯噪声混合，若动作尺度差异很大，训练会不稳定。

### 16.3 用 action chunk 而不是单步

单步动作噪声大、局部性太强；chunk 让模型更容易学到任务结构。

### 16.4 推理步数与实时性

ODE 步数太多会影响控制频率。机器人里通常选较少步数 + 高频重规划。

---

## 17. 你可以怎样理解 FM 在 VLA 中的角色

可以把整个系统理解成两部分：

### 第一部分：条件理解
VLA 编码器负责理解：

- 看到什么（视觉观测）
- 现在机器人处于什么状态
- 用户想做什么（语言指令）

得到条件表示 $c$。

### 第二部分：动作生成
Flow Matching 负责回答：

> 在这个条件下，未来动作分布是什么？  
> 从一团随机噪声出发，怎样连续地"流"成一段合理动作轨迹？

所以，**FM 在 VLA 中本质上是一个条件连续生成策略头**。

---

## 18. 一句话总结整套数学逻辑

Flow Matching 的数学链条：

1. 先定义从简单分布到目标分布的连续路径 $x_t$
2. 由路径得到真实速度 $u_t = \frac{dx_t}{dt}$
3. 用回归训练 $v_\theta(x_t, t) \approx u_t$
4. 最优解是条件期望速度场
5. 用这个速度场解 ODE，就能把噪声推到目标分布
6. 在 VLA 里，把"目标分布"换成"条件动作轨迹分布"即可

---

## 19. 最简公式总表

| 含义 | 公式 |
|:---|:---|
| 生成动力学 | $\dfrac{dx_t}{dt} = v_t(x_t)$ |
| 分布演化（连续性方程） | $\partial_t p_t + \nabla \cdot (p_t v_t) = 0$ |
| 直线路径 | $x_t = (1-t)x_0 + t x_1$ |
| 直线路径速度 | $\dfrac{dx_t}{dt} = x_1 - x_0$ |
| 一般高斯路径 | $x_t = \alpha_t x_1 + \sigma_t \epsilon$ |
| 一般路径速度（原始形式） | $u_t = \dot{\alpha}_t x_1 + \dot{\sigma}_t \epsilon$ |
| 一般路径速度（用 $x_t$ 表示） | $u_t(x_t \mid x_1) = \dot{\alpha}_t x_1 + \dfrac{\dot{\sigma}_t}{\sigma_t}(x_t - \alpha_t x_1)$ |
| FM 损失（无条件） | $\mathcal{L} = \mathbb{E}\|v_\theta(x_t, t) - u_t\|^2$ |
| FM 损失（条件，用于 VLA） | $\mathcal{L} = \mathbb{E}\|v_\theta(X_\tau, \tau, c) - U_\tau\|^2$ |
| 推理 ODE 积分（Euler） | $X_{\tau_{k+1}} = X_{\tau_k} + \Delta\tau\, v_\theta(X_{\tau_k}, \tau_k, c)$ |
| 最优速度场 | $v^*(x, t) = \mathbb{E}[u_t \mid x_t = x]$ |
