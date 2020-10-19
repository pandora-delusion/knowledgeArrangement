# PCA原理总结
主成分分析（Principal components analysis）是最重要的降维方法之一。在数据压缩消除冗余和数据噪音消除等领域都有广泛的应用。
### PCA的思想
> PCA：找出数据里最主要的方面，用数据里最主要的方面来代替原始数据。
具体的，假如我们的数据集是n维的，共有m个数据$\left ( x^{1},x^{2},\cdots,x^{m}\right )$。我们希望将这m个数据的维度从n维降到n’维，希望这m个n‘维数据集能尽可能表示原始数据集。我们知道数据从n维降到n’维肯定会有损失，但是希望损失尽可能的小。
首先看最简单的情况，也就是n=2， n'=1，也就是将数据从二维降到一维。如下图所示，我们希望找到某一个维度方向，他可以代表这两个维度的数据。图中列了两个方向，u1和u2，那么哪个向量可以更好的代表原始数据集呢？u2比u1好
![avatar](pca1.png)
有两种解释：1.样本到这个直线的距离足够近，2.样本点在这个直线上的投影能尽可能的分开
当把问题推广到任意维，希望的降维标准为：样本点到这个超平面的距离足够近，或者说样本点在这个超平面上的投影尽可能的分开。
基于上面有种标准，我们可以得到PCA的两种等价推导
### PCA的推导：基于最小投影距离
> 假设m个n维数据$\left ( x^{1},x^{2},\cdots,x^{m}\right )$都已经进行了中心化，即$\sum_{i=1}^{m}x^{i}=0$。经过投影变换后得到的新坐标系为$\left ( \omega_{1},\omega_{2}, \cdots, \omega_{n} \right )$，其中$\omega$是标准正交基，即$\left \| \omega  \right \|_{2}=1$，$\omega_{i}^{T}\omega_{j}=0$。
如果我们将数据从n维降到n’维，即丢弃新坐标系中的部分坐标，则新的坐标系为$\left ( \omega_{1},\omega_{2}, \cdots, \omega_{n'} \right )$，样本点$x^{\left ( i \right )}$在n‘维坐标系中的投影为：$z^{i}=\left (z_{1}^{(i)},z_{2}^{(i)},\cdots,z_{n'}^{(i)} \right )$，其中$z_{j}^{(i)}=\omega_{j}^{T}x^{(i)}$是$x^{(i)}$在低维坐标系里第j维的坐标。
新的坐标向量$\omega_{n'}$应该是一个n维向量
如果我们用$z^{(i)}$来回复原始数据$x^{(i)}$，则得到的恢复数据$\overline{x}^{(i)}=\omega z^{(i)}$，$\omega$为标准正交基组成的矩阵。
现在我们考虑真个样本集，希望所有的样本到这个超平面的距离足够近，即最小化下式：
$$\sum_{i=1}^{m}\left \| \overline{x}^{(i)} - x^{(i)}  \right \|_{2}^{2}$$
$\sum_{i=1}^{m}\left \| \overline{x}^{(i)}-x^{(i)} \right \|_{2}^{2}=\sum_{i=1}^{m}\left \| \omega z^{i}-x^{(i)} \right \|_{2}^{2}=\sum_{i=1}^{m}(\omega z^{(i)})^{T}(\omega z^{(i)})-2\sum_{i=1}^{m}(\omega z^{(i)})^{T}x^{(i)}+\sum_{i=1}^{m}x^{(i)T}x^{(i)}=\sum_{i=1}^{m}z^{(i)T}z^{(i)}-2\sum_{i=1}^{m}z^{(i)T}\omega^{T}x^{(i)}+\sum_{i=1}^{m}x^{(i)T}x^{(i)}=-tr(\omega^{T}(\sum_{i=1}^{m}x^{(i)}x^{(i)T})\omega)+\sum_{i=1}^{m}x^{(i)T}x^{(i)}=-tr(\omega^{T}XX^{T}\omega)+\sum_{i=1}^{m}x^{(i)T}x^{(i)}$
注意：$XX^{T}$是协方差矩阵（的和），$\omega$的每个向量是标准正交基。$\sum_{i=1}^{m}x^{(i)T}x^{(i)}$是一个常量。最小化上式等价于：
$$\underbrace{arg min}_{\omega}-tr\left ( \omega^{T}XX^{T}\omega \right ) s.t. \omega^{T}\omega=1$$
利用拉格朗日函数可以得到：
$$J(\omega) = -tr(\omega^{T}XX^{T} \omega)+\lambda(\omega^{T}\omega-I)$$对$\omega$求导有$$XX^{T} \omega=\lambda \omega$$可以看出$\omega$为$XX^{T}$的n'个特征向量组成的矩阵，而$\lambda$为$XX^{T}$的若干特征值组成的矩阵。当我们将数据集从n维降到n'维时，需要找到最大的n'个特征值对应的特征向量。这n'个特征向量组成的矩阵$\omega$就是我们需要的矩阵。对于原始数据集，我们只需要用$z^{(i)}=\omega^{T}x^{(i)}$，就可以把原始数据降维到最小投影距离的n'维数据集。

### PCA的推导：基于最大投影方差
> 假设m个n维数据$\left ( x^{1},x^{2},\cdots,x^{m}\right )$都已经进行了中心化，即$\sum_{i=1}^{m}x^{(i)}=0$。经过投影变后得到的新坐标系维$\left ( \omega_{1},\omega_{2}, \cdots, \omega_{n} \right )$,其中$\omega$是标准正交基，即$\left \| \omega \right \|_{2} = 1$,$\omega_{i}^{T}\omega_{j}=0$
如果将数据从n维降到n'维，即丢弃新坐标中的部分坐标，则新的坐标为$\left ( \omega_{1},\omega_{2}, \cdots, \omega_{n'} \right )$，样本点$x^{(i)}$在n'维坐标系中的投影为：$z^{i}=\left (z_{1}^{(i)},z_{2}^{(i)},\cdots,z_{n'}^{(i)} \right )$，其中$z_{j}^{(i)}=\omega_{j}^{T}x^{(i)}$是$x^{(i)}$在低维坐标系里第j维的坐标。
对于任意一个样本$x^{(i)}$，在新的坐标系中的投影为$\omega^{T}x^{(i)}$，在新的坐标系中投影的方差为$\omega^{T}x^{(i)}x^{(i)T}\omega$，要使所有的样本的投影方差和最大，也就是最大化$\omega^{T}XX^{T}\omega$的迹，即：
$$\underbrace{argmax}_{\omega}tr(\omega^{T}XX^{T}\omega) s.t. \omega^{T}\omega=1$$
可得：
$$XX^{T}\omega=(-\lambda)\omega$$
$\omega$为$XX^{T}$的n'个特征向量组成的矩阵，而$-\lambda$为$XX^{T}$的n'个特征向量组成的矩阵，特征值在主队角线上，其余位置为0。当我们将数据集从n维降到n'维时，需要找到最大的n'个特征值对应的特征向量。这n'个特征向量组成的矩阵$\omega$即为我们需要的矩阵。对于原始数据集，我们需要用$z^{(i)}=\omega^{T}x^{(i)}$，就可以把原始数据集降维到最大投影方差的n'维数据集。

### PCA算法流程
> 求样本$x^{(i)}$的n'维的主成分其实就是要求样本集的协方差矩阵$XX^{T}$的前n'个特征值对应特征向量矩阵$\omega$，然后对于每个样本$x^{(i)}$，做如下变换$z^{(i)}=\omega^{T}x^{(i)}$，即达到降维的PCA目的。
算法流程：
输入：n维样本集D=$(x^{(1)},x^{2}, \cdots, x^{m})$，要降维到的维度n'
输出：降维后的样本集D'
1、对所有的样本的协方差进行中心化：$x^{(i)}=x^{(i)}-\frac{1}{m}\sum_{j=1}^{m}x^{(i)}$
2、计算样本的协方差矩阵$XX^{T}$
3、对矩阵$XX^{T}进行特征值分解$
4、取出最大的n'个特征值对应的特征向量$(\omega_{1},\omega_{2},\cdots,\omega_{n'})$，将所有的特征向量标准化之后，组成特征向量矩阵$\omega$。
5、对样本集中的每一个样本$x^{(i)}$，转化为新的样本$z^{(i)}=\omega^{T}x^{(i)}$
6、得到输出样本集D’=$(z^{(1)}, z^{(2)}, \cdots, z^{(m)})$
有时，我们不指定降维后的n'的值，而是换种方式，指定一个降维到的主成分比重阀值t。这个阀值t在(0, 1]之间。假如我们的n个特征值$\lambda_{1}\geq\lambda_{2}\geq\cdots\geq\lambda_{n}$，则n'可以通过下式得到：
$$\frac{\sum_{i=1}^{n'} \lambda_{i}}{\sum_{i=1}^{n}\lambda_{i}}\geq t$$

### 核主成分分析KPCA介绍
在上面的PCA算法中，我们假设存在一个线性的超平面，可以让我们对数据进行投影。但是有些时候，数据不是线性的，不能直接进行降维，这里就需要用到核函数，先把数据集从n维映射到线性可分的高维$N>n$,然后再从N维降维到一个低维度n'，这里的维度之间满足n'<n<N。
假设高维空间的数据是由n维空间的数据通过映射$\phi$产生。
则对于n维空间的特征分解：
$$\sum_{i=1}^{m}x^{(i)}z^{(i)T}\omega=\lambda \omega$$
映射为：
$$\sum_{i=1}^{m} \phi(x^{(i)})\phi(x^{(i)})^{T} \omega = \lambda \omega$$
通过在高维空间进行协方差矩阵的特征值分解，然后用PCA方法进行降维。$\phi$不用显示的计算，而是在需要的时候通过核函数完成

### PCA算法总结
> PCA算法的主要优点有：
> - 仅仅需要以方差衡量信息量，不受数据集以外的因素影响。　
> - 各主成分之间正交，可消除原始数据成分间的相互影响的因素。
> - 计算方法简单，主要运算是特征值分解，易于实现。

>PCA算法的主要缺点有：
> + 主成分各个特征维度的含义具有一定的模糊性，不如原始样本特征的解释性强。
> + 方差小的非主成分也可能含有对样本差异的重要信息，因降维丢弃可能对后续数据处理有影响。

### Python PCA操作
```python
import numpy as np
x -= np.mean(x, axis=0) # 数据中心化
cov = np.dot(x.T, x) / x.shape[0] # 获得协方差
U, S, V = np.linalg.svd(cov) # S是1维特征值(不是奇异值),U是特征向量
xrot = np.dot(x, U) # 我们将原始数据投影到坐标系中(the eigenbasis)
# 如果此时计算xrot的协方差，我们可以看到这是个对角矩阵
# np.linalg.svd得到的U的特征向量是按照特征值的大小按照列来排序的，所以可以直接用来降维
xrot_reduced = np.dot(x, U[:, :100]) # 取前100维
'''通常情况下，通过在PCA缩减数据集上训练线性分类器或神经网络，
可以获得非常好的性能，从而节省空间和时间。'''

xwhilte = xrot / np.sqrt(s+1e-5) # 阻止除0的风险
'''
One weakness of this transformation is that it can greatly exaggerate the noise in the data, since it stretches all dimensions (including the irrelevant dimensions of tiny variance that are mostly noise) to be of equal size in the input. This can in practice be mitigated by stronger smoothing
'''
```

## 白化
> 白化的目的是除去输入数据的冗余信息，降低输入的冗余性
输入数据集X，经过白化处理后，新的数据X'满足两个性质：
- 特征之间相关性较低
- 特征具有相同的方差
仅仅用PCA求出特征向量，不降维，把数据X映射到新的特征空间，就已经满足了白化的第一个性质：除去特征之间的相关性。因此白化算法的实现过程，第一步操作就是PCA，求出新特征空间中X的新坐标，然后在对新的坐标进行方法归一化操作

#### PCA白化
> 对上面的PCA的新坐标X',每一维的特征做一个标准差归一化处理。
$$X^{''}_{PCAwhite}=\frac{X^{'}}{std(X^{'})}$$也可以采取下面的公式：$$X^{''}_{PCAwhite}=\frac{X^{'}}{\sqrt{\lambda_{i}+\varepsilon}}$$

#### ZCA白化
> 具体的实现就是把上面的PCA白化的结果，有变换到原来坐标系下的坐标：
$$Y_{ZCAwhite}=U*Y_{PCAwhite}$$
给人的感觉就像在PCA空间做了处理完后，又把它变换到原始的数据空间
```python
inputs -= np.mean(inputs, axis=0)
cov = np.dot(inputs.T, inputs)/inputs.shape[0]
U, S, V = np.linalg.svd(cov)
epsilon = 0.1
zcaMatrix = np.dot(np.dot(U, np.sqrt(np.diag(1.0/(S+epsilon)))), U.T)
return np.dot(zcaMatrix, inputs) 
```
![avatar](prepro2.jpeg)
