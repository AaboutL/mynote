---
title: 【MVG】Optimum Fiducials under Weak Perspective Projection
updated: 2023-06-20 03:59:45Z
created: 2023-04-25 07:10:09Z
latitude: 39.90421100
longitude: 116.40739500
altitude: 0.0000
tags:
  - 弱透视投影
  - 正交投影
---

# 1 公式

已知世界坐标系中的一些点$P_i = [X_i, Y_i, Z_i]^T$。在相机坐标系中观测到这个点的坐标为$P_i^\prime$。世界坐标系到相机坐标系的关系：$R, T$满足：
$P_i^\prime= RP_i + T$，其中：

$$
R = \left[
\begin{matrix}
r_1 \quad r_2 \quad r_3 \\
r_4 \quad r_5 \quad r_6 \\
r_7 \quad r_8 \quad r_9 
\end{matrix}
\right]
\quad and \quad
T = \left[
\begin{matrix}
t_1 \\
t_2 \\
t_3
\end{matrix}
\right]


$$

设$p_i^\prime$为相机图像平面中的对应点(其$z=f$)，相机的焦距为$f$，则透视投影的公式如下：

$$
P_i^\prime = R P_i +T \Leftrightarrow \\
\left[
\begin{matrix}
X_i^\prime \\
Y_i^\prime \\
Z_i^\prime
\end{matrix}
\right] = 
\left[
\begin{matrix}
[r_1 \ r_2 \ r_3]*P_i + t_1\\
[r_4 \ r_5 \ r_6]]*P_i + t_2 \\
[r_7 \ r_8 \ r_9]]*P_i + t_3
\end{matrix}
\right] \Rightarrow \\
x_i^\prime = \frac{f}{[r_7 \ r_8 \ r_9]*P_i + t_3}([r_1 \ r_2 \ r_3] * P_i + t_1) \\
y_i^\prime = \frac{f}{[r_7 \ r_8 \ r_9]*P_i + t_3}([r_4 \ r_5 \ r_6] * P_i + t_2) \\


$$

转换一下：

$$
\frac{([r_7 \ r_8 \ r_9] * P_i + t_3)*x_i^\prime}{t_3} = \frac{f}{t_3}([r_1 \ r_2 \ r_3] * P_i + t_1)


$$

另$\Delta_i = \frac{1}{t_3}[r_7 \ r_8 \ r_9] * P_i$，则上式可写作：

$$
(1+\Delta_i)x_i^\prime = \frac{f}{t_3}X_i^\prime \\
(1+\Delta_i)y_i^\prime = \frac{f}{t_3}Y_i^\prime


$$

如果$t_3$很大，说明这个点距离相机很远，则$\Delta_i$非常小。把$\Delta_i$设置为0，则可以得到弱透视投影方程。
所以，在弱透视投影下，当点距离相机很远时，有：$(x_i^\prime, y_i^\prime)=\alpha(X_i^\prime, Y_i^\prime)$，$\alpha=\frac{f}{t_3}$，可以写成如下形式：

$$
p_i^\prime = AP_i^\prime = 
\left[\begin{matrix}
\alpha & 0 & 0\\
0 & \alpha & 0
\end{matrix}\right] P_i^\prime


$$

合起来写作：

$$
p^\prime = A[R | T][P \ 1]^T


$$

把所有x坐标写作行向量$X=[X_1 ... X_N]$，同样定义$Y, Z, x^\prime，y^\prime，1=[1, 1...1]$

$$
\left[\begin{matrix}
x^\prime \\ y^\prime
\end{matrix}\right] = A[R | T]
\left[\begin{matrix}
X \\ Y \\ Z \\ 1
\end{matrix}\right] \tag{4}


$$

所以可以通过上面的公式恢复出R和T。
求解前，对$X，Y，Z，x^\prime，y^\prime$进行归一化处理，可以简化计算。
做如下定义：

$$
\lambda = [\lambda_1 \ \lambda_2 \ \lambda_3] = [\alpha \ 0 \ 0]R = [\alpha r_1\ \alpha r_2\ \alpha r_3] \\
\gamma = [\gamma_1 \ \gamma_2 \ \gamma_3] = [\alpha \ 0 \ 0]R = [\alpha r_4\ \alpha r_5\ \alpha r_6]


$$

$$
C = \left[\begin{matrix}
X^n \\ Y^n \\ Z^n 
\end{matrix}\right] \ (上标n表示归一化坐标)


$$

然后，可以得出($CC^T$是非奇异的，除非所有点在一个平面)

$$
\lambda C = x^{\prime(n)} \quad \gamma C = y^{\prime(n)}


$$

$$
\lambda^T = (CC^T)^{-1}Cx^{\prime(n)^T} \\
\gamma^T = (CC^T)^{-1}Cy^{\prime(n)^T}


$$

其实对于旋转矩阵性质，可以计算得到$\alpha$：

$$
r_1^2 +r_2^2 + r_3^2 = 1 \rightarrow \alpha^2(r_1^2 +r_2^2 + r_3^2)=\alpha^2 \\ \Rightarrow \lambda_1^2 + \lambda_2^2 + \lambda_3^2=\alpha^2 \\
同理：\gamma_1^2 + \gamma_2^2 + \gamma_3^2=\alpha^2


$$

然后可以计算得到R ：

$$
R_1 = [r_1 \ r_2 \ r_3] = [\lambda_1 \ \lambda_2 \ \lambda_3] / \alpha \\\
R_2 = [r_4 \ r_5 \ r_6] = [\gamma_1 \ \gamma_2 \ \gamma_3] / \alpha \\\
R_3 = [r_7 \ r_8 \ r_9] = R_1 \times R_2


$$

有了R，就可以根据公式（4）计算出T：

$$
T = \frac{1}{\alpha}[x \  y \  f]^T


$$

x、y可以通过计算得到。T的第三个值就是$t_3$，因为从上式中可以计算出缩放$\alpha$，只有知道相机内参的焦距，才能确定$t_3$。