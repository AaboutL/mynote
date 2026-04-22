---
title: 【算子代码】softmax交叉熵损失
updated: 2023-01-10 03:21:39Z
created: 2022-07-21 03:04:09Z
---

## 损失函数和导数推导

输入：$\pmb z = [z_1, z_2, ..., z_n]$，这里的输入指的是上一层fc的输出。
softmax计算结果：$\hat {\pmb y} = [\hat y_1, \hat y_2, ..., \hat y_n]$
label(one-hot)：$\pmb y = [y_1, y_2, ..., y_n]$

### 1\. softmax计算公式

$$
\hat y_i = \frac{e^{z_i}}{\sum_j^n e^{z_j}}

$$

### 2\. 交叉熵计算公式：

$$
L = -\sum_i^n y_ilog(\hat y_i) = -(y_1log(\hat y_1) + y_2log(\hat y_2) + ... + y_nlog(\hat y_n))

$$

### 3\. 偏导数
偏导数要清楚是谁对谁的偏导数，因为我们最终需要优化的是网络参数，所以，从根本上来说，偏导数应该是损失函数对参数的偏导数$\frac{\partial L}{\partial w}$。网络是由一层一层构建的，假设网络有n层+softmaxCE loss，可以表示为：
$$
y_1 = f_{w_1}(x) \\
y_2 = f_{w_2}(y_1) \\ 
... \\
y_n = f_{w_n}(y_{n-1}) \\
l = softmax(y_n) \\
L = CE(l).
$$
反向传播时，参数更新是逐层进行的，需要知道L关于每一层参数w的偏导数。比如求n-1层的偏导数：
$$
\frac{\partial L}{\partial w_{n-1}} = \frac{\partial L}{\partial l} * \frac{\partial l}{\partial y_n} * \frac{\partial y_n}{\partial y_{n-1}} * \frac{\partial y_{n-1}}{\partial w_{n-1}}
$$
所以，每一层只需要知道自己层的偏导数是什么，一般需要记录两个偏导数：
1. 当前层的输出相对于当前层的输入的偏导数：$\frac{\partial y_n}{\partial y_{n-1}}$，目的是作为链式法则的中间层，把梯度传到更浅层。
2. 当前层的输出相对于当前层参数的偏导数：$\frac{\partial y_{n-1}}{\partial w_{n-1}}$，目的就是更新当前层的参数。

**对于损失函数层，因为不存在需要优化的参数，所以只需要记录输出对于输入的偏导数**。

### 3.1 损失函数L对z的偏导数：
**假设label中第k个元素为1，其他为0，即$y_k = 1，y_i=0, i \neq k$**
对$z_1$的偏导数如下：因为$\hat y$中的每一项$\hat y_i$都包含了所有的$\pmb z$，所以每一个$\hat y_i$都需要对$z_1$求偏导

$$
\frac{\partial L}{\partial z_1} = -(\frac{y_1}{\hat y_1}*\frac{\partial \hat y_1}{\partial z_1} + \frac{y_2}{\hat y_2}*\frac{\partial \hat y_2}{\partial z_1} +... + \frac{y_n}{\hat y_n}*\frac{\partial \hat y_n}{\partial z_1}) = -\frac{y_k}{\hat y_k}*\frac{\partial \hat y_k}{\partial z_1}

$$

同理，对$z_i$的偏导数为：

$$
\frac{\partial L}{\partial z_i} = -\frac{y_k}{\hat y_k}*\frac{\partial \hat y_k}{\partial z_i}

$$

**写成矩阵形式为：**

$$
\frac{\partial L}{\partial {\pmb z}} = 
\left[\begin{matrix}
-\frac{y_k}{\hat y_k}*\frac{\partial \hat y_k}{\partial z_1} \\
... \\
-\frac{y_k}{\hat y_k}*\frac{\partial \hat y_k}{\partial z_i} \\
... \\
-\frac{y_k}{\hat y_k}*\frac{\partial \hat y_k}{\partial z_n}
\end{matrix}\right]

$$

### 3.2 softmax函数$\hat {\pmb y}$对z的偏导数$\frac{\partial \hat y_k}{\partial z_i}$

根据上面的矩阵形式可以看出，对$z_i$求导时，要区分$i=k \ or \ i \neq k$两种情况

1.  $i=k$时

$$
\hat y_k = \frac{e^{z_k}}{\sum + e^{z_k}}

$$

$$
\frac{\partial \hat y_k}{\partial z_k} = \frac{e^{z_k}*(\sum +e^{z_k})-e^{z_k}*e^{z_k}}{(\sum +e^{z_k})^2} = \hat y_k*(1-\hat y_k)

$$

2.  $i \neq k$时

$$
\hat y_k = \frac{e^{z_k}}{\sum + e^{z_i}}

$$

$$
\frac{\partial \hat y_k}{\partial z_i} = \frac{-e^{z_k}*e^{z_i}}{(\sum + e^{z_i})^2} = -\hat y_k * \hat y_i

$$

**带入到$\frac{\partial L}{\partial {\pmb z}}$矩阵形式为：**

$$
\frac{\partial L}{\partial {\pmb z}} = 
\left[\begin{matrix}
-\frac{y_k}{\hat y_k}*\frac{\partial \hat y_k}{\partial z_1} \\
... \\
-\frac{y_k}{\hat y_k}*\frac{\partial \hat y_k}{\partial z_i} \\
... \\
-\frac{y_k}{\hat y_k}*\frac{\partial \hat y_k}{\partial z_n}
\end{matrix}\right] = 
\left[\begin{matrix}
-\frac{y_k}{\hat y_k}*(-\hat y_k \hat y_1) \\
... \\
-\frac{y_k}{\hat y_k}* \hat y_k(1- \hat y_k) \\
... \\
-\frac{y_k}{\hat y_k}*(-\hat y_k \hat y_n)
\end{matrix}\right] = 
\left[\begin{matrix}
\hat y_1 \\
... \\
\hat y_k - 1 \\
... \\
\hat y_n
\end{matrix}\right] = 
\left[\begin{matrix}
\hat y_1 - 0 \\
... \\
\hat y_k - y_k\\
... \\
\hat y_n - 0
\end{matrix}\right]=
\hat {\pmb y} - \pmb y

$$
虽然上面分开推导了$i=k和i \neq k$的情况，但是最终偏导数的形式是统一的，不需要区分，所以在代码实现上就比较简单了。
**综上，交叉熵损失L对输入z的偏导数推导完毕**

## python实现：前向和反向传播

```
class SoftmaxWithCrossEntropy(object):
    def __init__(self):
        self.loss = None
        self.logit = None
        self.label = None
    
    def forward(self, input, label):
        self.logit = Softmax(input)
        self.label = label
        self.loss = CrossEntropyError(self.logit, self.label)
        
    def backward(self):
        batch_size = self.label.shape[0]
        if self.label.shape == self.logit.shape ## label是one-hot形式
            dx = (self.logit - self.label) / batch_size
        else: ## label是(0,1,2, ..., n-1)形式
            dx = self.logit
            dx[np.arange(batch_size), self.label] -= 1 ## 每个样本logit的第label个元素减去1
            dx /= batch_size
        return dx
```

```
def CrossEntropyError(logit, label):
    if logit.ndim == 1:
        label = label.reshape(1, label.size)
        logit = logit.reshape(1, logit.size)
    
    if logit.size == label.size ## label是one-hot形式
        label = label.argmax(axis=1) ## 找到one-hot中1出现的位置
    
    batch_size = logit.shape[0]
    return -np.sum(np.log(logit[np.arange(batch_size), label] + 1e-7)) / batch_size
```