---
title: slam基础-求导
updated: 2022-11-19 17:50:38Z
created: 2022-05-24 09:15:34Z
latitude: 39.91175842
longitude: 116.37922668
altitude: 0.0000
---

# slam面试-求导
## 相对于旋转的导数
### 1. 旋转求导 $\frac{\partial R_1R_2}{\partial R_1}$
$$
\frac{\partial R_1R_2}{\partial R_1} = \underset{\phi \to 0}{lim}\frac{Ln(R_1Exp(\phi)R_2) - Ln(R_1R_2)}{\phi} \\ 
= \underset{\phi \to 0}{lim}\frac{Ln(R_1R_2Exp(R_2^T\phi)) - Ln(R_1R_2)}{\phi} \\ 
= \underset{\phi \to 0}{lim}\frac{Ln(R_1R_2) + J_r^{-1}\{Ln(R_1R_2)\}R_2^T\phi - Ln(R_1R_2)}{\phi} \\
=J_r^{-1}\{Ln(R_1R_2)\}R_2^T
$$
### 2. 旋转求导：$\frac{\partial R_1R_2}{\partial R_2}$
$$
\frac{\partial R_1R_2}{\partial R_1} = \underset{\phi \to 0}{lim}\frac{Ln(R_1R_2Exp(\phi)) - Ln(R_1R_2)}{\phi} \\
= \underset{\phi \to 0}{lim}\frac{Ln(Exp(Ln(R_1R_2) + J_r^{-1}\{Ln(R_1R_2)\}\phi)) - Ln(R_1R_2)}{\phi} \\
=\underset{\phi \to 0}{lim}\frac{Ln(R_1R_2) +J_r^{-1}\{Ln(R_1R_2)\}\phi - Ln(R_1R_2)}{\phi} \\
= J_r^{-1}\{Ln(R_1R_2)\}
$$
**在用极限的方式进行导数推导时，需要用到减法，但是在SO(3)中并没有定义加减法操作，所以需要转换到李群so(3)中进行。**
### 3. 假设对一个空间点p进行了旋转，得到了Rp。现在要计算旋转之后点的坐标相对于旋转的导数，非正式的定义为：$\frac{\partial(Rp)}{\partial R}$。
1. **用李代数求导**
设R对应的李代数为$\vec\phi$，所以对旋转矩阵的求导转化成对旋转向量的求导。
	**左雅可比矩阵**：
$$
\frac{\partial(Rp)}{\partial R} \Leftrightarrow \frac{\partial(Exp(\vec\phi)p)}{\partial \vec\phi} \\
= \underset{\delta\phi\to 0}{lim}\frac{Exp(\phi + \delta\phi)p - Exp(\phi)p}{\delta\phi} \\ 
=\underset{\delta\phi\to 0}{lim}\frac{Exp(J_l(\phi)\delta\phi)Exp(\phi)p - Exp(\phi)p}{\delta\phi} \\
=\underset{\delta\phi\to 0}{lim}\frac{(I + (J_l(\phi)\delta\phi)^\wedge)Exp(\phi)p - Exp(\phi)p}{\delta\phi} \\ 
=\underset{\delta\phi\to 0}{lim}\frac{(J_l(\phi)\delta\phi)^\wedge Exp(\phi)p}{\delta\phi} \\ 
=\underset{\delta\phi\to 0}{lim}\frac{-(Exp(\phi)p)^\wedge J_l(\phi)\delta\phi}{\delta\phi} \\
=-(Rp)^\wedge J_l(\phi)
$$
**右雅可比矩阵**：
$$
\frac{\partial(Rp)}{\partial R} \Leftrightarrow \frac{\partial(Exp(\vec\phi)p)}{\partial \vec\phi} \\
= \underset{\delta\phi\to 0}{lim}\frac{Exp(\phi + \delta\phi)p - Exp(\phi)p}{\delta\phi} \\ 
=\underset{\delta\phi\to 0}{lim}\frac{Exp(\phi)Exp(J_r(\phi)\delta\phi)p - Exp(\phi)p}{\delta\phi} \\
=\underset{\delta\phi\to 0}{lim}\frac{Exp(\phi)(I + (J_r(\phi)\delta\phi)^\wedge)p - Exp(\phi)p}{\delta\phi} \\ 
=\underset{\delta\phi\to 0}{lim}\frac{Exp(\phi)(J_r(\phi)\delta\phi)^\wedge p}{\delta\phi} \\ 
=\underset{\delta\phi\to 0}{lim}\frac{-Exp(\phi)p^\wedge J_r(\phi)\delta\phi}{\delta\phi} \\
=-Rp^\wedge J_r(\phi)
$$
李代数推导结果中有一项$J_l(\phi)$，实际计算比较复杂。
	
2. **扰动模型**
对R进行一次扰动$\Delta R$，看结果相对于扰动的变化率。设扰动$\Delta R$对应的李代数为$\delta\phi$，对$\delta\phi$求导
**左扰动**：
$$
\frac{\partial(Rp)}{\partial R} \Leftrightarrow \frac{\partial(Rp)}{\partial \delta\phi} \\ 
=\underset{\delta\phi\to 0}{lim}\frac{Exp(\delta\phi)Exp(\phi)p - Exp(\phi)p}{\delta\phi} \\
=\underset{\delta\phi\to 0}{lim}\frac{(I + (\delta\phi)^\wedge)Exp(\phi)p - Exp(\phi)p}{\delta\phi} \\ 
=\underset{\delta\phi\to 0}{lim}\frac{ (\delta\phi)^\wedge Exp(\phi)p}{\delta\phi} \\
=\underset{\delta\phi\to 0}{lim}\frac{-(Exp(\phi)p)^\wedge \delta\phi}{\delta\phi} \\
=-(Rp)^\wedge
$$

**右扰动**：
$$
\frac{\partial(Rp)}{\partial R} \Leftrightarrow \frac{\partial(Rp)}{\partial \delta\phi} \\ 
=\underset{\delta\phi\to 0}{lim}\frac{Exp(\phi)Exp(\delta\phi)p - Exp(\phi)p}{\delta\phi} \\
=\underset{\delta\phi\to 0}{lim}\frac{Exp(\phi)(I + (\delta\phi)^\wedge)p - Exp(\phi)p}{\delta\phi} \\ 
=\underset{\delta\phi\to 0}{lim}\frac{ Exp(\phi)(\delta\phi)^\wedge p}{\delta\phi} \\
=\underset{\delta\phi\to 0}{lim}\frac{-Exp(\phi)p^\wedge \delta\phi}{\delta\phi} \\
=-Rp^\wedge
$$

对比李代数求导与扰动模型可以看出，扰动模型的结果相比李代数求导结果，少一个雅可比矩阵。

### 4. SE(3)上的扰动模型
对空间中的点p进行了旋转和平移变换$T=(R|t)$，则对T的偏导为：
$\delta \xi = [\delta \phi \quad \delta t]^T$
$(\delta \xi)^\wedge = 
\left[
\begin{matrix}
(\delta \phi)^\wedge \quad \delta t \\
0^T \quad 0
\end{matrix}
\right]
$
**左扰动**
$$
\frac{\partial (Tp)}{\partial T} \Leftrightarrow \frac{\partial (Tp)}{\partial \delta \xi} \\
=\underset{\delta \xi \to 0}{lim}\frac{Exp(\xi + \delta \xi)p - Exp(\xi)p}{\delta \xi} \\
=\underset{\delta \xi \to 0}{lim}\frac{Exp(\delta \xi)Exp(\xi)p - Exp(\xi)p}{\delta \xi} \\
=\underset{\delta \xi \to 0}{lim}\frac{(I + (\delta \xi)^\wedge)Exp(\xi)p - Exp(\xi)p}{\delta \xi} \\ 
=\underset{\delta \xi \to 0}{lim}\frac{ (\delta \xi)^\wedge Exp(\xi)p}{\delta \xi} \\
=\underset{\delta \xi \to 0}{lim}\frac{
\left[
\begin{matrix}
(\delta \phi)^\wedge \quad \delta t \\
0^T \quad 0
\end{matrix}
\right] 
\left[
\begin{matrix}
Tp \\
1
\end{matrix}
\right]}{
\left[
\begin{matrix}
\delta \phi \\
\delta t
\end{matrix}
\right]}
=\underset{\delta \xi \to 0}{lim}\frac{
\left[
\begin{matrix}
(\delta \phi)^\wedge Tp + \delta t \\
0
\end{matrix}
\right]}
{\left[
\begin{matrix}
\delta \phi \\
\delta t
\end{matrix}
\right]} \\ 
=\underset{\delta \xi \to 0}{lim}
\left[
\begin{matrix}
\frac{(\delta \phi)^\wedge Tp + \delta t}{\delta \phi} \quad \frac{(\delta \phi)^\wedge Tp + \delta t}{\delta t}\\
0^T \quad 0^T
\end{matrix}
\right] \\ 
= \left[
\begin{matrix}
-(Tp)^\wedge \quad I\\
0^T \quad 0^T
\end{matrix}
\right]
$$