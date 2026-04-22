---
title: 【DL基础】BatchNorm
updated: 2023-02-27 04:47:43Z
created: 2023-02-27 03:05:33Z
latitude: 39.90421100
longitude: 116.40739500
altitude: 0.0000
---

BatchNorm可以加速深层神经网络的收敛，避免梯度爆炸。
## 训练深层网络的问题
当一个网络很深时，输入和损失函数相隔的层数很多，梯度回传就会遇到一些问题。在梯度回传时，由于相乘的作用，梯度会逐渐变小。所以，距离损失函数近的层，参数更新的快，更容易收敛；距离损失函数远的层，参数更新的慢，不容易收敛。每当最深的那些层收敛后，如果浅层的参数发生变化，那么深层的参数还得重新学习，可以认为深层需要学习很多次。这就是训练很深的网络时，收敛慢的原因。
一般浅层用于抽取局部的边缘、纹理等信息，当这些信息的参数发生变化时，深层的参数肯定要跟着发生变化。
**如何在训练浅层时，避免深层发生变化？**

## BatchNorm
导致变化的原因被认为是不同层的特征的分布在不受限制的变化。
BatchNorm的解决方法就是把不同层的特征分布固定住，在高斯分布的假设下，让均值和方差限制在一个小范围内变化。当每一层的大致分布固定时，那么就更容易学习了，只需要调整一些细微的变化。
BatchNorm的公式：
$$
均值： \mu_B = \frac{1}{|B|}\sum_{i \in B}x_i \\
方差： \sigma_B^2=\frac{1}{|B|}\sum_{i \in B}(x_i-\mu_B)^2 + \epsilon \\
B表示一个批量
$$
再加上两个可学习的参数$\gamma, \beta$：
$$
x_{i+1} = \gamma \frac{x_i - \mu_B}{\sigma_B}+\beta
$$
这是说明，把特征转换为均值为0，方差为1的正态分布不太合适，那么你可以学习一个均值$\beta$和方差$\gamma$，使得特征对网络的学习更加友好，但是会限制$\gamma, \beta$的值变化不能太剧烈。
### 对全连接层
BN作用在特征维，即对每个特征做一个BN。
比如一个批量有 |B| 个样本$B=\{s_1, s_2...S_B\}$，每个样本有 |F|个特征$F=\{f_1, f_2...f_{|F|}\}$。 每一行表示一个样本，每一列表示一个特征，那么BN会分别对每一列计算一次，就是在特征的 |B| 个样本之间计算均值和方差，然后做一个线性变换。那么最后会有 |F| 个均值和方差。

### 对卷积层
BN作用在通道维。假设卷积层的特征维度为：NCHW
对通道维做BN，最后得到 |C|个均值和方差。

## BN到底在做什么
<font color=red>论文中说BN的作用是为了减少内部协变量的转移</font>，但是作者也没有实际做对比实验去看协变量是否真的发生变化。
后面有人专门做实验看过是否真的减少了协变量转移，发现并没有。
<font color=blue>所以后续论文认为他是在每个小批量中加入噪声来控制模型的复杂度。</font>认为$\mu_B, \sigma_B$是一个噪音，它们是在一个随机的小批量上得到，它们的噪音很大，相当于一个随机的偏移和缩放，然后再用学习的方式得到一个稳定的均值和方差$\beta, \gamma$。
<font color=blue>如果认为BN就是用来控制模型复杂度的方法，那么就不需要跟dropout方法混合使用了。</font>

## 总结
可以加速收敛，但是不改变模型精度。 

## 代码
batch_norm 算子：
~~~python
def batch_norm(X, gamma, beta, moving_mean, moving_var, eps, momentum):
	if not torch.is_grad_enabled(): ## 推理时
		X_hat=(X-moving_mean)/torch.sqrt(moving_var + eps)
	else:
		assert len(X.shape) in (2, 4) #只考虑FC或者2维卷积
		if len(X.shape)==2:
			mean = X.mean(dim=0)
			var = ((X-mean)**2).mean(dim=0)
		else:
			mean = X.mean(dim=(0,2,3), keepdim=True)
			var = ((X-mean)**2).mean(dim=(0,2,3), keepdim=True)
		X_hat = (X-mean) / torch.sqrt(var + eps)
		moving_mean = momentum * moving_mean + (1-momentum) * mean
		moving_var = momentum * moving_var + (1-momentum) * var
	Y = gamma * X_hat + bata
	return Y, moving_mean.data, moving_var.data
~~~

BatchNorm层
~~~python
class BatchNorm(nn.Module):
	# num_features: 全连接层的输出数量或者卷积层的通道数
	# num_dims: 2表示全连接层，4表示卷积层
	def __init__(self, num_features, num_dims):
		super.__init__()
		if num_dims==2:
			shape=(1, num_dims)
		elif num_dims=4:
			shape=(1, num_dims, 1, 1)
		self.gamma = nn.Parameter(torch.ones(shape))
		self.beta = nn.Parameter(torch.zeros(shape))
		
		self.moving_var = torch.ones(shape)
		self.moving_mean = torch.zeros(shape)
	
	def forward(self, X):
		if self.moving_mean.device != X.device:
    	self.moving_mean = self.moving_mean.to(X.device)
    	self.moving_var = self.moving_var.to(X.device)
		Y, self.moving_mean, self.moving_var = batch_norm(
            X, self.gamma, self.beta, self.moving_mean,
            self.moving_var, eps=1e-5, momentum=0.9)
		return Y
~~~