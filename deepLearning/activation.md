## Activation Function

> sigmoid函数
> $\sigma \left ( x \right )=\frac{1}{\left ( 1+e^{-x} \right )}$
> 饱和神经元，也就是说这个神经元的输出要么非常接近0，要么非常接近1，这些神经元
> 会导致在反向传播算法中出现梯度趋于0的问题。(梯度消失)
> 另一个问题是sigmoid函数的输出不是关于原点中心对称的。这样导致函数的梯度要么
> 全正要么全负，这样会导致收敛的速度非常的慢（fisher矩阵和神经梯度？？？）
> 还有一个问题就是计算耗时

> tanh函数
> 梯度依旧可以饱和

> ReLu函数（修正线性单元Rectified linear unit）
> $f\left ( x \right )=max\left ( 0,x \right )$
> 首先不会饱和，至少在输入为正时不会，只有在很小的有边界的区域被激活时，才会出现
> 这种问题，计算高效
> 相较于sigmoid/tanh函数，ReLu能很好的加速SGD的收敛
> 不是关于原点对称的，若在正向传播的过程中没有被激活，那么反向传播时就会产生梯度消失
> 在0点的梯度不存在。在初始化神经元的时候不能把权重设置为不能使神经元激活的数值，这种
> 情况下神经元是不会训练的。当学习速率太高，神经元在一定范围内波动，可能会发生数据多样
> 性丢失，这种情况下神经元将不会在激活，并且数据多样性的丢失不可逆转。这些死掉的神经元
> 不在不会被更新。

> Leaky ReLu函数
> $f\left ( x \right )=max\left ( 0.01x,x\right )$
> 不会再有神经元失活的问题

> Parametric Rectifier （PReLu）
> $f\left ( x \right )=max\left ( \alpha x,x\right )$
> $\alpha$成为网络中的一个参数，你可以通过反向传播学习这个参数

> Exponential Linear Units(ELU)
> $f\left ( x \right )=\begin{cases}
 x& \text{ if } x> 0 \\ 
 \alpha \left ( exp\left ( x \right )-1 \right )& \text{ if } x\leq 0
\end{cases}$
> 拥有ReLu的所有优势，不会失活，会得到0均值的输出（争议）。 

> Maxout "Neuron"
> 泛化了ReLu和Leaky ReLu
> $max\left ( \omega _{1}^{T}x+b_{1},\omega _{2}^{T}x+b_{2}\right )$ 
> 同样不会失活，分段线性和高效率，两倍参数，ReLu任然是应用最广泛的

