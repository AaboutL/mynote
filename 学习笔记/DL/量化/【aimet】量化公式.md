## 核心公式
1. **量化**：浮点转量化
	$$
	\bar x = clamp(\lfloor \frac{x}{s} \rceil - z, q_{min}, q_{max})
	$$
2. **反量化**：量化转浮点
	$$
	\hat{x} = (\bar x + z) · s
	$$
	其中：
	- s: scale（缩放因子）
	- z: offset / zero-point （零点偏移）
	- $q_{min}, q_{max}$：量化范围边界，W4对称量化时，$q_{min}=-8, q_{max} = 7, z=0$
	- $\lfloor · \rceil$ ：round to nearest（四舍五入）
	- max, min：浮点参数的最大和最小值
	Scale和Offset计算公式：
$$
s = \frac{max-min}{q_{max} - q_{min}}
$$
$$
z = clamp(\lfloor \frac{min}{s} \rceil - q_{min}, -q_{max}, -q_{min})
$$
3.  **Straight Through Estimate(STE)**
	由于round(四舍五入)操作不可微，所以前向传播用round（不可微），反向传播时假装 round 不存在，梯度直接透传（乘以 1 ）：
	$$
	\frac{\partial{\lfloor x \rceil}}{\partial x} \approx 1
	$$
	**具体原因**： round 的真实导数几乎处处为 0（因为 round 是阶梯函数，平坦处导数为 0）。如果用真实导数，梯度全部消失，无法训练。STE 的核心假设是"round 引入的误差很小，可以近似忽略"。