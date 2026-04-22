---
title: 【3dmm】参数拟合——弱透视投影到透视投影
updated: 2024-09-24 02:53:17Z
created: 2023-06-23 02:55:24Z
latitude: 39.90421100
longitude: 116.40739500
altitude: 0.0000
---

## 1\. 为什么要从弱透视投影转换到透视投影

弱透视投影不考虑相机参数，假设相机距离足够远，而实际中，镜头不会距离太远，所以应用会带来精度损失。透视投影会考虑相机参数。

## 2\. 拟合步骤

拟合步骤还是按照zhuxiangyu的流程，只是pose拟合、shape拟合、expression拟合需要重新推导公式。

### 2.1 pose拟合

Pose拟合可以直接使用PnP进行解算

### 2.2 expression拟合

设相机矩阵为$K$，  
旋转矩阵和平移向量分别为$R、t$，  
2d landmark表示为$\pmb x=(u, v, 1)$。  
拟合公式如下：

$$
S_{2d} = K(R(\bar S + A_{id}\alpha_{id} + A_{exp}\alpha_{exp}) + t)
$$

为了表示方便，假设$R(\bar S + A_{id}\alpha_{id} + A_{exp}\alpha_{exp}) + t$ 已经在归一化平面。  
拟合表情参数时，需要固定形状参数$\alpha_{id}$和位姿$R/t$。

$$
\begin{align}
& R(\bar S + A_{id}\alpha_{id} + A_{exp}\alpha_{exp}) + t \\
=& R(\bar S + A_{id}\alpha_{id}) + t + RA_{exp}\alpha_{exp} \\
=& t_{3d} + E\alpha_{exp} \\
\stackrel{展开} \Longrightarrow& 
\begin{bmatrix}
x_s \\ y_s \\ z_s
\end{bmatrix} + 
\begin{bmatrix}
x_e \\ y_e \\ z_e
\end{bmatrix} * \alpha_{exp} = 
\begin{bmatrix}
x_s + x_e\alpha \\ y_s + y_e\alpha \\ z_s + z_e\alpha
\end{bmatrix} 
\stackrel{归一化平面} \Longrightarrow &
\begin{bmatrix}
\frac{x_s + x_e\alpha}{z_s + z_e} \\ \frac{y_s + y_e\alpha}{z_s + z_e} \\ 1
\end{bmatrix} \\
\stackrel{则} \Longrightarrow &
\begin{bmatrix}
u \\ v \\ v
\end{bmatrix} = K
\begin{bmatrix}
\frac{x_s + x_e\alpha}{z_s + z_e} \\ \frac{y_s + y_e\alpha}{z_s + z_e} \\ 1
\end{bmatrix} \\ 
\stackrel{两边左乘K^{-1},并定义} \Longrightarrow &
\begin{bmatrix}
x_n \\ y_n \\ 1
\end{bmatrix} = 
K^{-1} \begin{bmatrix}
u \\ v \\ 1
\end{bmatrix} = 
\begin{bmatrix}
\frac{x_s + x_e\alpha}{z_s + z_e} \\ \frac{y_s + y_e\alpha}{z_s + z_e} \\ 1
\end{bmatrix} \\ 
\stackrel{两边乘以z_s + z_e，并整理} \Longrightarrow &
z_s \begin{bmatrix}
x_n \\ y_n \\ 1 
\end{bmatrix} + 
z_e\alpha \begin{bmatrix}
x_n \\ y_n \\ 1 
\end{bmatrix} = 
\begin{bmatrix}
x_s \\ y_s \\ z_s
\end{bmatrix} + 
\begin{bmatrix}
x_e\alpha \\ y_e \alpha \\ z_e \alpha
\end{bmatrix} \\
\stackrel{交换位置} \Longrightarrow &
\begin{bmatrix}
x_e - z_e x_n \\ 
y_e - z_e y_n \\
z_e - z_e
\end{bmatrix} *\alpha = 
\begin{bmatrix}
z_s x_n - x_s\\ 
 z_s y_n - y_s\\
z_s - z_s
\end{bmatrix} \\
\stackrel{令} \Longrightarrow &
A = \begin{bmatrix}
x_e - z_e x_n \\ 
y_e - z_e y_n \\
0
\end{bmatrix} ,
b = \begin{bmatrix}
z_s x_n - x_s\\ 
 z_s y_n - y_s\\
0
\end{bmatrix} \\
\Longrightarrow & A \alpha = b \\ 
\Longrightarrow & A^TA\alpha = A^T b \\
\stackrel{系数加上正则项}\Longrightarrow & (A^TA + \lambda*M_{reg})\alpha = A^T b \\
\Longrightarrow & \alpha = \frac{A^T b}{A^TA + \lambda*M_{reg}}

\end{align} 
$$

## 使用同样的方式拟合shape参数