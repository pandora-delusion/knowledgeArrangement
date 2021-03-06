# CNN

我们可以计算输出卷的大小:
指定输入卷大小 w
卷积神经元局部感受野的大小 F
滑动的跨度 S
0填充的数量 P
输出卷的大小:(W - F + 2P)/S + 1

实际上,在反向传播中,卷积中的每个神经元都将计算它权重的梯度,这些梯度将在每个深度切片上相加,并且只更新每个切片的一组权重

参数共享假设是有理由的: 如果在图片的某个位置检测水平边缘是重要的,那么直观上在其他位置也是有用的,因为图片结构的平移不变性.所以没有必要在卷基层的输出卷上再学习检测水平边缘了.

请注意，有时参数共享假设可能没有意义。尤其是当输入到卷积网络的图像具有特定的中心结构时，例如，我们应该期望在图像的一侧学习完全不同的特性。一个实际的例子是当输入是在图像中居中的脸时.您可能期望在不同的空间位置学习不同的眼睛或头发特征,在这种情况下,放松参数共享方案是很常见的,只需将层称为本地连接层(a Locally-Connected Layer)

卷积的矩阵乘法实现:
im2col 太耗内存,但是可以利用矩阵乘法的实现优势降低计算复杂度,并且im2col方法可以在池化操作中重复使用.

在卷积网络体系结构中,在连续的卷积层之间定期的插入一个池化层是很常见的,其作用是逐步减小表示的空间大小,以减少网络中的参数和计算量从而控制过拟合.池化层操作在每个输入的深度切片上独立操作,控制其大小.深度不变

池化层输出卷的大小:
接受卷的大小 $W_{1}*H_{1}*D_{1}$
需要两个超参数: 空间范围F,步长S
输出卷的大小 $W_{2}*H_{2}*D_{2}$:
$W_{2}=(W_{1}-F)/S+1$
$H_{2}=(H_{1}-F)/S+1$
$D_{2}=D_{1}$
池化层不需要0填充,没有参数
值得注意的是，实践中发现的最大池层只有两种常见的变化：一种是f=3，s=2（也称为重叠池）的池层，更常见的是f=2，s=2。具有较大接收字段的池大小太具有破坏性

##### General pooling
除了最大池外,池单元还可以执行其他功能:比如平均池,L2-norm池
平均池过去经常使用,但现在最大池更有效

max(x, y)操作的后向传播有一个简单的解释,即只将梯度路由到前向传播中具有最高值的输入.
丢弃池化层在训练好的生成模型中很重要,比如变分自动编码器(VAE)或生成对抗网络(GAN).未来的体系结构很可能只有很少直到没有池化层

##### Normalization Layer
在实践中,normalization层的贡献非常小[cuda-convnet library API](http://code.google.com/p/cuda-convnet/wiki/LayerParams#Local_response_normalization_layer_(same_map))

##### Fully-connected layer
....


转化FC层到卷积层
对于任何卷积层,都有一个fc层实现相同的正向函数.权重矩阵是一个很大的矩阵,大部分为0,除了在某些块(由于局部联通性),其中许多块中的权重相等(参数共享)
任何fc层都可以转换为卷积层.

将FC层转换为卷积层的能力在实践中特别有用.
[Net Surgery](https://github.com/BVLC/caffe/blob/master/examples/net_surgery.ipynb)演示如何在实践中执行转换

常见的卷积网络结构模式如下:
input->[[conv -> RELU]*N -> POOL?]*M -> [FC -> RELU]*K -> FC
一般情况下0<=N<=3, M>=0,K>=0(K<3)

倾向于使用小的滤波器卷基层
比如堆叠三个3x3的卷积层,前两层相当于一个5x5的输入卷积,前三层相当于一个7x7的输入卷积
单层7x7输入卷积和3层3x3输入卷积的接收域大小一致但是有几个缺点:相比于三层,使特征缺乏更具表现性的非线性;使用的参数更多,单层:cx(7x7xc)=$49c^{2}$,三层:3x(cx(3x3xc))=27$c^{2}$.三层使用更少的参数表达更加强大的输入特性,同时减少了过拟合.只是使用了更多的内存

注意:
传统的卷积层的线性堆叠模式正在受到挑战(Google's inception architectures和Residual networks)

实践:
使用imageNet上最有效的工具.不要为一个问题回滚你自己的体系结构,应该去看现在imageNet上最有效的体系结构,下载一个预训练的模型,并在自己的数据上进行微调.很少需要从开头训练一个卷积网络或者从头设计一个卷积网络

#### Layer Sizing Patterns
输入层(包含图像),应该可以被2整除,常用数字32, 64, 96, 224, 384, 512
卷积层应使用小过滤器(3x3,5x5),步幅为s=1, 最重要的是,用0填充,这样卷积层就不会改变输入的空间尺寸
池化层负责对输入的空间维度进行降采样,最常见的是2x2接收域的降采样,步长为2,这将完全丢弃输入体积中75%的激活值;另外一种不常见的设置是使用3x3的接收域,步长为2,最大池的接收字段大小大于3是非常罕见的,因为损耗太大,太过激进,会导致性能下降

所有的conv层都应该保留其输入空间大小,而池化层单独负责对体积进行下采样.
在一种替代方案中,使用大于1的步长且在卷积层的输入中不填充0,必须非常小心的跟踪输入体积.在整个CNN架构中,确保所有的步长和过滤器都能正常工作,并且卷积架构是良好且对称的连接

为什么使用步长为1的卷积?
小的步长s=1在实践中工作的很好,步长为1允许我们将所有空间向下采样保留到池化层,卷积层只转换输入的深度

为什么使用填充?
在卷积之后保持空间大小不变,提高了性能.如果conv层不是0填充,只执行有效的卷积,那么在每次卷积之后,卷的大小将减少一部分,并且边界处的信息被冲走的太快

RESNet是最先进的卷积神经网络模型[Residual Nets](http://torch.ch/blog/2016/02/04/resnets.html)

计算上的考量
构建卷积网络时要注意的最大的瓶颈是内存瓶颈.
有三个主要的内存来源可以追踪:

 - 中间卷的大小
 - 参数的大小
 - 维护内存(批处理大小)

##### 空间批量正则化(Spatial Batch Normalization, SBN)
深度网络的训练是一个复杂的过程,只要网络的前面几层发生微小的改变,那么后面几层就会被累积放大下去.一旦网络某一层的输入数据的分布发生改变,那么这一层网络就需要去适应学习这个新的数据分布.所以,在训练过程中,如果训练数据的分布一直发生变化,那么网络的训练速度将会受到影响.基于"批量正则化层(BN)"的理念,我们引入了"空间批量正则化SBN层".
BN说到底还是防止训练过程中的"梯度消失"

1*1卷积核的作用：
- 当卷积核的个数小于输入channels数量时，降维；升维
- 在保持特征隐射尺度不变的前提下增加非线性特性（利用后接的非线性激活函数）
- 跨通道信息交互



### 转置卷积的计算

转置卷积的需要通常来自于使用与普通卷积方向相反的变换的愿望。比如，从具有某种卷积的输出形状的东西到具有其输入形状的东西，同时保持与上述卷积建通的连接模式。例如，可以使用诸如卷积自动编码器的解码层之类的转换，或者将特征映射投影到高维空间。

####  卷积的矩阵运算形式

如果将输入和输出从左到右，从上到下展开成向量，卷积可以表示为稀疏矩阵C，其中非0元素为和的元素$w_{i,j}$：
$$
\begin{pmatrix}
w_{0,0} & w_{0,1} & w_{0,2} & 0 & w_{1,0} & w_{1,1} & w_{1,2} & 0 & w_{2,0} & w_{2,1} & w_{2,2} & 0 & 0 & 0 & 0 & 0 \\ 
0 & w_{0,0} & w_{0,1} & w_{0,2} & 0 & w_{1,0} & w_{1,1} & w_{1,2} & 0 & w_{2,0} & w_{2,1} & w_{2,2} & 0 & 0 & 0 & 0 \\ 
0 & 0 & 0 & 0 & w_{0,0} & w_{0,1} & w_{0,2} & 0 & w_{1,0} & w_{1,1} & w_{1,2} & 0 & w_{2,0} & w_{2,1} & w_{2,2} & 0 \\ 
0 & 0 & 0 & 0 & 0 & w_{0,0} & w_{0,1} & w_{0,2} & 0 & w_{1,0} & w_{1,1} & w_{1,2} & 0 & w_{2,0} & w_{2,1} & w_{2,2}
\end{pmatrix}
$$
这个线性操作将输入矩阵扁平化为一个16维的向量，并生成一个4维向量，之后将其重新塑造为$2 \times 2$的输出矩阵。利用这种方法，通过转置很容易得到反向传播；误差通过将损失与$C^{T}$相乘来实现反向传播:
$$
\begin{align*}
\frac{\partial loss}{\partial x_{i}} &=\sum_{j}\frac{\partial Loss}{\partial y_{j}}\cdot \frac{\partial y_{j}}{\partial x_{i}} \\
&= C_{*，i}^{T}\cdot \frac{\partial Loss}{\partial y}
\end{align*}
$$
则
$$
\frac{\partial Loss}{\partial x}=C^{T}\cdot \frac{\partial Loss}{\partial y}
$$
C由内核W决定。

#### 转置矩阵

​	转置卷积——也被称为逆卷积（deconvolutions）或者fractionally strided convolutions——通过交换卷积的正向和反向传递来实现。注意：核函数(kernel)定义了卷积，但是它是直接卷积还是转置卷积是由计算正向和反向传递的方式决定的。

​	比如，即使核w定义了前向和后向传播分别由乘以$C$和$C^{T}$来计算的卷积，它也定义了一个转置卷积，这个转置卷积的前向和后向传播分别由乘以$C^{T}$和$(C^{T})^{T}$来计算。

​	最后注意，总是可以用直接卷积来模拟转置卷积。缺点是总是涉及添加许多行和列的0到输入中去，导致一个效率低得多的实现。

​	转置卷积可以被认为是恢复这个初始特征映射形状的运算。

​	另一种获得转置卷积结果的方法是应用一个等效的——但是效率低得多的——直接卷积。

描述为$s=1,p=0$和$k$的卷积的相关联的转置卷积被描述为$k^{'}=k,s^{'}=s$和$p^{'}=k-1$，它的输出的大小为：
$$
o^{'}=i^{'}+(k-1)
$$
描述为$s=1,k,p$的卷积的相关联的转置卷积被描述为$k^{'}=k,s^{'}=s,p^{'}=k-p-1$，它的输出大小为：
$$
o^{'}=i^{'}+(k-1)-2p
$$
描述为$k=2n+1,s=1,p=\left \lfloor k/2 \right \rfloor=n$的卷积的相关联的转置卷积被描述为$k^{'}=k,s^{'}=s,p^{'}=p$，并且它的输出大小为：
$$
\begin{align*}
o^{'}&=i^{'}+(k-1)-2p \\
&=i^{'}+2n-2n \\
&=i^{'}
\end{align*}
$$
描述为$s=1,k,p=k-1$的卷积的相关联的转置卷积被描述为$k^{'}=k,s^{'}=s,p^{'}=0$，它的输出大小为：
$$
\begin{align*}
o^{'}&=i^{'}+(k-1)-2p \\
&=i^{'}-(k-1)
\end{align*}
$$
当没有0填充，非1stride时，转置卷积的实现：

​	在输入单元之间插入0，这使得核的移动速度比单位步长慢，这是对应关系如下：

描述为$p=0,k,s$的卷积的相关联的转置卷积被描述为$i^{'},k^{'}=k,s^{'}=1,p^{'}=k-1$，它的输出大小为：
$$
o^{'}=s(i^{'}-1)+k
$$
![deconv](F:\mycode\knowledgeArrangement\deepLearning\deconv.png)

当$i+2p-k$是s的倍数时：

描述为$k,s,p$的卷积的相关联的转置卷积被描述为$i^{'},k^{'}=k,s^{'}=1,p^{'}=k-p-1$，它的输出大小为：
$$
o^{'}=s(i^{'}-1)+k-2p
$$
![](F:\mycode\knowledgeArrangement\deepLearning\deconv1.png)

输入i的大小可以通过映入参数$\alpha \in \{ 0, \cdots, s-1 \}$来允许在s不同的情况下得到相同的$i^{'}$:

描述为$k,s,p$的卷积的相关联的转置卷积被描述为$\alpha, i^{'},k^{'}=k,s^{'}=1,p^{'}=k-p-1$，在每个输入单元之间要添加$s-1$个0，$\alpha =(i+2p-k) \% s$代表添加到输入底部和右边的0的数量，输出大小为：
$$
o^{'}=s(i^{'}-1)+\alpha+k-2p
$$
![](F:\mycode\knowledgeArrangement\deepLearning\deconv2.png)

### 混合卷积（Miscellaneous convolutions）

#### Dilated convolutions

或者叫”atrous convolutions“（膨胀卷积）。膨胀卷积通过在内核元素之间插入空间来膨胀内核，膨胀率由额外的超参数d来控制。实现可能有所不同，通常有d-1个空间插入到内核元素之间，d=1对应普通的卷积。膨胀卷积用于在不增加内核大小的情况下，廉价地增加输出单元的接受域，当多个膨胀卷积叠加在一起时，这种方法尤为有效。

大小为k的核被因子d膨胀后具有有效大小:
$$
\widehat{k}=k+(k-1)(d-1)
$$
对于任何描述为$i,k,p,s$，膨胀速率为d的膨胀卷积，其输出大小为：
$$
o=\left \lfloor \frac{i+2p-k-(k-1)(d-1)}{s} \right \rfloor+1
$$
![](F:\mycode\knowledgeArrangement\deepLearning\mconv.png)

### Depthwise Convolutions and Depthwise separable convolution

[原文](https://eli.thegreenplace.net/2018/depthwise-separable-convolutions-for-machine-learning/)

Depthwise卷积：

![](F:\mycode\knowledgeArrangement\deepLearning\depthwise.png)

Depthwise separable卷积：（中间没有非线性层）

![](F:\mycode\knowledgeArrangement\deepLearning\depthwise_sep.png)

Depthwise Separable convolution近年来在DNN模型中变得流行，两个原因：

+ 相比于一般卷积层有更少的参数，因此不容易过度拟合
+ 使用更少的参数，也需要更少的操作来计算，因此更便宜和更快

### Xception

原论文: Xception： Deep Learning with Depthwise Separable Convolution

Inception模块：

![](F:\mycode\knowledgeArrangement\deepLearning\Inception.png)

首先对输入进行$1 \times 1$的卷积，获得多个跨通道的特征映射，然后通过relu非线性层。接着在不重合的输出通道上做不同大小的空间卷积。

实际上，Inception背后的基本假设是，跨通道关联和空间关联已经充分解耦，因此最好不要联合映射它们。而在Inception的基础上做一个更强的假设，即跨通道相关性和空间相关性可以完全独立地映射。于是引入 depthwise separabe convolution，用separable convolution组件替换Inception组件。

卷积核深度可分离卷积位于一个离散的谱系的两个极端，Inception是两者之间的一个中间点。

