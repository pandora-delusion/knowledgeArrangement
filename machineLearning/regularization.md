正则化
=======

正则化项一般是模型复杂度的单调递增函数，模型越复杂，正则化值就越大。

>加入权重衰减来修改线性回归的训练标准<br />
>带权重衰减的线性回归最小化训练集上的均方误差和正则项的和$J\left ( \omega  \right )$,其偏好于平方$L^{2}$：<br />
>$J\left ( \omega  \right )=MSE_{train}+\lambda \omega ^{T}\omega $
>直观的说L2正则化要做的就是尽可能地展开w权重，以便考虑所有的输入特征，并且尽可能地利用这些维度，如果能得到相同结果的话，那样比只关注一个信息要好
>$\lambda$控制我们偏好小范数权重的程度，最小化$J\left ( \omega  \right )$可以看做拟合训练数据和偏好小权重范数之间的权衡<br />
>正则化是指修改学习算法，使其降低泛化误差而非训练误差。在深度学习的背景下，大多数正则化策略都会对估计进行正则化。估计的正则化以偏差的增加换取方差的减少。能显著减少方差而不过度增加偏差。控制模型的复杂度不是找到合适规模的模型，最好的拟合模型（从最小泛化误差的意义上）是一个适当正则化的大型模型。
>
>* 参数范数惩罚
>$\widehat{J}\left ( \theta ;X,y \right )=J\left ( \theta ;X,y \right )+\alpha \Omega \left ( \theta  \right )$
>我们通常只对权重做惩罚而不对偏置做惩罚。精确拟合偏置所需的数据通常比拟合权重少得多。不对偏重做正则化也不会导致太大的方差。正则化偏置参数可能会导致明显的欠拟合
>L2参数正则化：通常被称为权重衰减，$\Omega \left ( \theta  \right )=\frac{1}{2}\left \| \omega  \right \|^{2}$使权重更加接近原点，加入权重衰减后会引起学习规则的修改，在每步执行通常的梯度更新之前先收缩权重向量。只有在显著减小目标函数方向上的参数会保留的相对完好，在无助于帮助函数减小的方向上改变参数不会显著增加梯度，这种不重要方向对应的分量会在训练过程中因为正则化而衰减掉。L2正则化能让学习算法感知到具有较高方差的输入x，因此与输出目标的协方差较小（相对增加方差）的特征的权重将会收缩。L2正则化可以直观的解释为严重惩罚峰值权重向量和偏爱弥散的权重向量。
>L1正则化：$\Omega \left ( \theta  \right )=\left \| \omega  \right \|_{1}=\sum _{i}\left | \omega _{i} \right |$相比L2，L1会产生更加稀疏的解（最优值中的一些参数为0），被广泛应用于特质选择机制。具有l1正则化的神经元最终只使用它们最重要输入的稀疏子集，并且几乎不受“噪声”输入的影响。在实践中，如果不关心显示特征选择，那么用L2正则化提供优于L1正则化的性能。
>Elastic net(L1+L2): 
>$R\left ( \omega  \right )=\sum_{k}\sum_{l}\beta \omega _{k,l}^{2}+\left | \omega _{k,l} \right |$
>Max norm constraints（？？？？）
>Dropout:在训练过程中，dropout只通过使用概率p（超参数）保持一个神经元活动来实现，否则设置为零。During testing there is no dropout applied, with the interpretation of evaluating an averaged prediction across the exponentially-sized ensemble of all sub-networks.测试的时候不用dropout，但是在隐藏层激活函数后面要乘以dropout概率p 
>inverted dropout:在训练阶段进行缩放，在测试阶段不需要。      
>dropout属于一种更一般的方法，它在网络的正向传递中引入随即行为。
>在实际应用中（适当的数据预处理），将偏差规范化很少对导致性能显著下降 
>每层正规化：将不同的层规范化为不同的数量并不常见(可能输出层除外)。这一观点在文献中发表的结果相对较少
>[DropConnect](https://cs.nyu.edu/~wanli/dropc/)                              

###### 验证集
>从训练数据中构建验证集，将训练数据分成两个不相交的子集。一个用于学习参数，一个作为验证集，用于估计训练中和训练后的泛化误差，来更新超参数。用于挑选超参数的子集被称为验证集，80%训练，20%验证，尽管验证集的误差通常比训练集误差小，验证集会低估泛化误差。所有超参数优化完成后，泛化误差可能会通过测试集来估计。
一个小规模的测试集意味着平均测试误差估计的统计不确定性，使得很难判断算法A是否比算法B在给定的任务上做得更好。
###### 估计，偏差和方差
>* 点估计
试图为一些感兴趣的量提供单个“最优”预测，感兴趣的量可以是单个参数，或是某些参数模型中的向量参数，为了区分参数估计和真实值，我们习惯将参数$\theta $的点估计表示为$\widehat{\theta }$。令$\left \{ x^{\left ( 1 \right )},x^{\left ( 2 \right )},...,x^{\left ( m \right )} \right \}$是m个独立分布的数据点。点估计或统计量是这些数据的任意函数：
$\widehat{\theta }_{m}=g\left ( x^{\left ( 1 \right )},x^{\left ( 2 \right )},...,x^{\left ( m \right )} \right )$
这个定义不要求g返回一个接近真实$\theta $的值，或者g的值域恰好是$\theta $的允许取值范围，点估计的定义非常宽泛，给了估计量的设计者极大的灵活性。虽然几乎所有的函数都可以称为估计量，但是一个良好的估计量的输出会接近生成训练数据的真实参数$\theta $。我们假设真实参数$\theta $是固定但未知的，而点估计$\widehat{\theta }$是数据的函数。由于数据是随机过程采样出来的，数据的任何函数都是随机的，因此$\widehat{\theta }$是一个随机变量。
点估计也可以指输入和目标变量之间关系的估计，我们将这种类型的点估计称为函数估计。
>* 函数估计
试图从输入向量x预测变量y，假设一个函数f(x)表示y和x之间的近似关系。例如，我们可能假设y=f(x)+e，其中e是y中未能从x预测的一部分。在函数估计中，我们感兴趣的是用模型估计去近似f，或者估计$\widehat{f}$，
。函数估计和估计参数$\theta $是一样的，函数估计$\widehat{f}$是函数空间中的一个点估计。线性回归实例和多项式回归实例都既可以被解释为估计参数$\omega $，又可以被解释为估计从x到y的函数映射$\widehat{f}$
>* 偏差
估计的偏差被定义为：
$bias\left ( \widehat{\theta }_{m} \right )=E\left ( \widehat{\theta }_{m} \right )-\theta$
期望作用在所有数据上（看做从随机变量采用得到的）上
如果$bias\left ( \widehat{\theta }_{m} \right ) = 0$ 则$\widehat{\theta }_{m}$是无偏估计量，这意味着$E\left ( \widehat{\theta }_{m} \right )=\theta$。如果$\lim _{m\rightarrow \infty }bias\left ( \widehat{\theta }_{m} \right )=0$那么$\widehat{\theta }_{m}$是渐近无偏，这意味着$\lim _{m\rightarrow \infty }E\left ( \widehat{\theta }_{m} \right )=\theta $
###### 方差和标准差
> 样本方差的平方根和方差无偏估计的平方根都不是标准差的无偏估计，这两种计算方法都倾向于低估真实的标准差。相较而言，方差无偏估计的平方根较少被低估。对于较大的m这非常合理。均值的标准差在机器学习实验中非常有用，我们通常用测试集样本的误差均值来估计泛化误差。测试集中样本的数量决定了这个估计的精确度。中心极限定理告诉我们均值会接近一个高斯分布
###### 权衡偏差和方差以最小化均方误差
> 偏差度量着偏离真实函数或参数的误差期望，而方差度量着数据上任意特定采样可能导致的估计期望的偏差
> 判断这种权衡最常用的方法是交叉验证，经验上，交叉验证在真实世界的许多任务中都非常成功。另外，我们也可以比较这些估计的均方误差：
> $MSE=E\left [ \left ( \widehat{\theta }_{m}-\theta  \right )^{2} \right ]=Bias\left ( \widehat{\theta }_{m} \right )^{2}+Var\left ( \widehat{\theta }_{m} \right )$ MSE度量着估计和真实参数$\theta $之间平方误差的总体期望偏差。MSE估计包含了偏差和方差。理想的估计具有较小的MSE或是在检查中会稍微约束它们的偏差和方差。用MSE度量泛化误差（偏差和方差对于泛化误差都是有意义的）时，增加容量会增加方差，降低偏差。
###### 一致性
> $p\lim _{m\rightarrow \infty }\widehat{\theta }_{m}=\theta $即对于任意的$\varepsilon > 0$，当$m\rightarrow \infty $时，有$P\left ( \left | \widehat{\theta }_{m}-\theta  \right | > \epsilon \right )\rightarrow 0$，表示的条件被称为一致性。一致性保证了估计量的偏差会随数据样本数目的增多而减少。
###### 容量，过拟合和欠拟合
>训练集和测试集数据都是通过数据集上被称为数据生成过程的概率分布生成
>独立同分布假设：该假设说，每个数据集中的样本都是彼此相互独立的，并且训练集和测试集是同分布的，这个假设使我们能够在单个样本的概率分布描述数据生成过程。相同的分布可以用来生成每一个训练样本和每一个测试样本。这个共享的潜在分布称为数据生成分布，这个概率框架和独立同分布假设允许我们从数学上研究训练误差和测试误差之间的关系。这样随机模型训练误差的期望和该模型测试误差的期望是一样的，因为这两个期望的计算都是用了相同的数据集生成过程。
>欠拟合是指模型不能在训练集上获得足够低的误差，而过拟合是指训练误差和测试误差之间的差距太大。通过调整模型的容量，我们可以控制模型是否偏向于过拟合或者欠拟合。

### 正则化与数据先验分布的关系

从贝叶斯的角度来看，正则化等价于对于模型参数引入先验分布

#### 线性回归

回归模型：$y=Xw+\epsilon $可以被看做下列模型
$$
p(y|X,w,\lambda)=N(Xw, \lambda) with p(\epsilon)=N(0,\lambda)
$$

$$
\begin{align*}
&P(\epsilon^{(i)})=\frac{1}{\sqrt{2 \pi} \delta}exp(- \frac{(\epsilon^{(i)})^{2}}{2 \delta^{2}}) \\
\Rightarrow &P(y^{(i)}|x^{(i)}; \theta)=\frac{1}{\sqrt{2 \pi}\delta}exp(- \frac{(y^{(i)}-w^{T}x^{(i)})^{2}}{2 \delta^{2}}) \\
\end{align*}
$$

由极大似然估计（MLE）:
$$
\begin{align*}
L(w)&=P(\overrightarrow{y}|X;w)\\
&=\prod_{i=1}^{m}P(y^{(i)}|x^{(i)};\theta) \\
&=\prod_{i=1}^{m} \frac{1}{\sqrt{2 \pi} \delta}exp(- \frac{(y^{(i)}-w^{T}x^{(i)})^{2}}{2 \delta^{2}})
\end{align*}
$$
取对数：
$$
\begin{align*}
l(w)&=log L(w) \\
&=log \prod_{i=1}^{m} \frac{1}{\sqrt{2 \pi} \delta}exp(- \frac{(y^{(i)}-w^{T}x^{(i)})^{2}}{2 \delta^{2}}) \\
&= \sum_{i=1}^{m}log \frac{1}{\sqrt{2 \pi} \delta}exp(- \frac{(y^{(i)}-w^{T}x^{(i)})^{2}}{2 \delta^{2}}) \\
&= m \cdot log \frac{1}{\sqrt{2 pi} \delta}- \frac{1}{\delta^{2}}\cdot \frac{1}{2} \sum_{i=1}^{m}(y^{(i)}-w^{T}x^{(i)})^{2}
\end{align*}
$$
即：
$$
w_{MLE}=argmin_{w} \sum_{i=1}^{m}(y^{(i)}-w^{T}x^{(i)})^{2}
$$
这就是原始的最小二乘损失函数，但是这是在我们对参数w没有加入任何先验分布的情况下。在数据维度很高的情况下，我们的模型的参数很多，模型复杂度高，容易发生过拟合。（维度灾难）

#### Ridge Regression

我们对参数w引入协方差为$\alpha$的零均值高斯先验。
$$
\begin{align*}
L(w)&=P(\overrightarrow{y}|X;w)P(w) \\
&= \prod_{i=1}^{m}P(y^{(i)}|x^{(i)};\theta)P(w) \\
&= \prod_{i=1}^{m} \frac{1}{\sqrt{2 \pi}\delta}exp(- \frac{(y^{(i)}-w^{T}x^{(i)})^{2}}{2 \delta^{2}}) \prod_{j=1}^{n} \frac{1}{\sqrt{2 \pi \alpha}}exp(- \frac{(w^{(j)})^{2}}{2 \alpha})
\end{align*}
$$


取对数：
$$
\begin{align*}
l(w)&=logL(w) \\
&=m \cdot log \frac{1}{\sqrt{2 \pi} \delta} + n \cdot log \frac{1}{\sqrt{2 \pi \alpha}} - \frac{1}{\delta^{2}} \cdot \frac{1}{2} \sum_{i=1}^{m}(y^{(i)}-w^{T}x^{(i)})^{2}- \frac{1}{\alpha} \cdot \frac{1}{2} w^{T}w \\
\end{align*}
$$
等价于：
$$
w_{MLE}=argmin_{w} \frac{1}{n} \left \| y-w^{T}X \right \|_{2}+ \lambda \left \| w \right \|_{2}
$$
*老实说 ridge regression 并不具有产生**稀疏解**的能力，也就是说参数并不会真出现很多零。假设我们的预测结果与两个特征相关，L2正则倾向于综合两者的影响，给影响大的特征赋予**高的权重**；而L1正则倾向于选择影响较大的参数，而**舍弃**掉影响较小的那个。实际应用中 L2正则表现往往会优于 L1正则，但 L1正则会大大降低我们的**计算量**。*

现在知道了，对参数引入高斯先验等价于L2正则化。

#### LASSO

注：LASSO - least absolute shrinkage and selection operator（最小绝对收缩和选择算子）

LASSO - Laplacian Prior for the weights w

L1正则等价于对权重w设置Laplacian先验：
$$
w \sim C exp^{- \lambda \left | w \right |}
$$
拉普拉斯分布：
$$
f(x| \mu,b)= \frac{1}{2b}exp(- \frac{\left | x-\mu \right |}{b})
$$
![](F:\mycode\knowledgeArrangement\machineLearning\laplacian.png)

重复之前的推导过程可以得到：
$$
w_{MAP_{laplace}}=argmin_{w}(\frac{1}{\delta^{2}} \cdot \frac{1}{2} \sum_{i=1}^{m}(y^{(i)}-w^{T}x^{(i)})^{2}+\frac{1}{b}\cdot\sum_{j=1}^{n}\left \| w \right \|_{1})
$$
该问题通常被称为LASSO，这仍然时一个convex optimization问题，不具有解析解。它的优良性质时能产生稀疏性，导致w中许多项变成0.

对参数引入拉普拉斯先验等价于L1正则化。

LASSO在解决”small n, large p problem“存在一定缺陷。

![](F:\mycode\knowledgeArrangement\machineLearning\lasso_limit.jpg)

#### Elastic Net

将L1和L2联合起来

（并不想推导。。。。）
$$
\widehat{\beta}=argmin_{\beta}\left \| y-X\beta \right \|_{2}+\lambda_{2}\left \| \beta \right \|_{2}+\lambda_{1}\left \| \beta \right \|_{1}
$$
![](F:\mycode\knowledgeArrangement\machineLearning\elastic.jpg)

#### 总结

正则化参数等价于对参数引入先验分布，使得模型复杂度变小（缩小解空间），对于噪声以及离群点的鲁棒性增强（泛化能力）。整个最优化问题从贝叶斯观点来看就是一种贝叶斯最大后验估计（MAP），其中正则化项对应后验估计中的先验信息，损失函数对用后验估计中的似然函数，两者的乘积即对应贝叶斯最大后验估计的形式。

### L1相比于L2为什么更容易获得稀疏解

[原论文](https://www.zhihu.com/question/37096933)

![](F:\mycode\knowledgeArrangement\machineLearning\L_compare1.jpg)

![](F:\mycode\knowledgeArrangement\machineLearning\L_compare2.jpg)

![](F:\mycode\knowledgeArrangement\machineLearning\L_compare3.jpg)

 

