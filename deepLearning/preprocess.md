# 数据预处理

> (数据归一化处理？？)机器学习中运用的数据归一化处理在图像处理中并不常用，因为你不用区分不同的特征。
> 但是0中心化应用很多（PCA算法？？）
> 在图像处理中常见的是均值中心化处理（mean image =[32, 32, 3] array）x -= np.mean(x. axis=0)；另一种方法是减去单通道均值，在红黄蓝三色通道中分别计算，最后得到三个数值。(mean along each channel = 3 numbers) x -= np.mean(x)
## [PCA和白化](../machineLearning/pca.md)
> PCA和白化是另一种预处理的形式，数据先被中心化，x -= np.mean(x, axis=0),然后计算
> 告诉我们数据相关结构的协方差矩阵: cov = np.dot(x.T, x) / x.shape[0]; cov矩阵的(i,j)个元素表示数据的第i个维度和第j个维度之间的协方差，对角线是各个维度的方差；协方差矩阵是对称和半正定的

# 权重初始化
> w = 0.01*np.random.randn(D,H)
> 对于层数较少的神经网络它的效果很好，但是随着层数的增加，对于初始化更加敏感。使用这种初始化方式，在深层网络中，梯度的量级会随着反向传播的进行不断缩小。（训练过程中发现损失值都没有怎么变，说明无法进行反向传播，没有权值得到更新）
> (Xavier初始化???)
> w = np.random.randn(fan_in, fan_out)/np.sqrt(fan_in) 这种初始化方法在tanh和sigmoid方法中使用效果显著，但是在ReLu方法中不好，所以由针对ReLu的合理的分布：
> w = np.random.randn(fan_in, fan_out)/np.sqrt(fan_in/2) ---- 没有这个因数2，你的激活输出的分布会以指数级收缩
> 如何初始化权重是一种数据驱动的技术
$Var(s)=Var(\sum_{i}^{n} \omega_{i} x_{i})\\
=\sum_{i}^{n}Var(\omega_{i}x_{i})\\
=\sum_{i}^{n}[E(\omega_{i})]^{2}Var(x_{i})+E[(x_{i})]^{2}Var(\omega_{i})+Var(x_{i})Var(\omega_{i})\\
=\sum_{i}^{n}Var(x_{i})Var(\omega_{i})\\
=(nVar(\omega))Var(x)$
这里假定$\omega$和$x$的数学期望为0（对ReLu不适用），假定$\omega$和$x$和独立分布，从公式中可以看出，如果希望s和x同分布，在权重的初始化时必须确保权重的方法为1/n,又因为$Var(aX)=a^{2}Var(X)$，所以权重的初始化为
```python
w = np.random.randn(fan_in, fan_out) / np.sqrt(fan_in) 
```

# 批数据的规范化(Batch Normalization)



> $\hat{x}^{\left ( x \right )}=\frac{x^{\left ( k \right )}-E\left [ x^{\left ( k \right )} \right ]}{\sqrt{Var\left [ x^{\left ( k \right )} \right ]}}$单位高斯，批数据规范化保证数据的每一个单位都是单位高斯，这是一个可微分方程
> 现在我们的网络流层是这样的：FC->BN->tanh->FC->BN->tanh， BN层保证神经网络
> 中的每一步的每一个东西都是roughly 单位高斯的，BN层一般添加在全连接层或这卷基层后面，非线性层之前。在实践中，使用批处理规范化的网络对错误的初始化更为健壮。
> $y^{\left ( k \right )}=\gamma ^{(k)}\widehat{x}^{\left ( k \right )}+\beta ^{(k)}$
可以先将$\gamma$和$\beta$的值初始化为1和0，然后在调整$\gamma$和$\beta$的值
网络可以学习：
$\gamma ^{\left ( k \right )}=\sqrt{Var\left [ x^{\left ( k \right )} \right ]}$
$\beta ^{\left ( k \right )}=E\left [ x^{\left ( k \right )} \right ]$ 这时候就抵消了BN的作用，所以这部分通过学习来低消BN的作用，所以BN其实等价于一个恒等函数，或者说可以通过学习达到恒等的作用。当你在FC层和tanh层之间防止BN层的时候，神经网络可以通过BP算法决定是要取消BN层的作用
> BN增强了整个神经网络的梯度流，它支持更高的学习速率，所以你可以更快的训练模型。它减少了算法对合理的初始化的依赖性，BN实际上起到了一些正则化的作用，它减少了dropout的需要。
BN会使减速训练，使得运行时间加长
> 在测试的时候，BN层作用并不相同：
    不需要计算每个batch的均值和标准差，而是使用训练过程中激活的单个固定整体均值
    (e.g. can be estimated during training with running averages)

# Cross-validation strategy and Random Search Strategy
> 首先我们只需要有个大概的参数区间，对这个区间做一个粗略的研究，然后在原有的区间内选出表现比较好的一个小区间。然后我们一遍一遍重复这个步骤，让我们选的区间越来越窄，最后选出一个表现最好的参数在你的代码中。当你优化正则化系数和学习速率时，最好从对数空间中取样，因为这两个变量在你进行反向传播算法的时候是相乘的。 ？？？ 
> 交叉验证没有随机取值验证性能好，在超参数的优化过程中，经常发生一个超参数的重要性会远远高于另一个，这样随机取样

# 稀疏初始化(Sparse initialization)
> 解决未校准方差问题的另一种方法是将所有权重矩阵设置为零，但为了打破对称性，每个神经元都随机连接（从上面的小高斯中抽取权重）到其下方的固定数量的神经元。一个典型的连接神经元数量可能只有10个。