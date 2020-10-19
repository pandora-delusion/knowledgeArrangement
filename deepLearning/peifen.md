# 直面配分函数

##### 对数似然梯度
通过最大似然学习无向模型特别困难的原因在于配分函数依赖于参数。对数似然相对于参数的梯度具有一项对应于配分函数的梯度：$$\triangledown _{\theta }logp\left ( x;\theta  \right )=\triangledown _{\theta }log\widetilde{p}\left ( x;\theta  \right )-\triangledown _{\theta }logZ\left ( \theta  \right )$$这是机器学习中非常著名的正相和负相的分解。

进一步分析$logZ$的梯度：$$\triangledown _{\theta }logZ=\frac{\triangledown _{\theta }Z}{Z}=\frac{\triangledown _{\theta }\sum _{x}\widetilde{p}\left ( x \right )}{Z}=\frac{\sum _{x}\triangledown _{\theta }\widetilde{p}\left ( x \right )}{Z}$$对于保证所有的x都有$p(x)>0$的模型，我们可以用$exp(log\widetilde{p}(x))$代替$\widetilde{p}(x)$:$$\frac{\sum _{x}\triangledown _{\theta }exp\left ( log\widetilde{p}\left ( x \right ) \right )}{Z}=\frac{\sum _{x}exp\left ( log\widetilde{p}\left ( x \right ) \right )\triangledown _{\theta }log\widetilde{p}\left ( x \right )}{Z}$$ $$=\frac{\sum _{x}\widetilde{p}\left ( x \right )\triangledown _{\theta }log\widetilde{p}\left ( x \right )}{Z}=\sum _{x}p\left ( x \right )\triangledown _{\theta }log\widetilde{p}\left ( x \right )$$ $$=E_{x\sim p\left ( x \right )}\triangledown _{\theta }log\widetilde{p}\left ( x \right )$$
在连续版本的推导中，使用在积分符号内取微分的莱布尼茨法则可以得到等式：$$\triangledown_{\theta }\int \widetilde{p}\left ( x \right )dx=\int \triangledown _{\theta }\widetilde{p}\left ( x \right )dx$$ 条件略，大多数感兴趣的机器学习模型都具有这些性质。

等式：$$\triangledown_{\theta }logZ=E_{x\sim p\left ( x \right )}\triangledown _{\theta }log\widetilde{p}\left ( x \right )$$是使用各种蒙特卡罗方法近似最大化（具有难以计算配分函数模型的）似然的基础。

蒙特卡罗方法为学习无向模型提供了直观的框架，我们能够在其中考虑正相和负相。在正相中，我们增大从数据中采样得到的$log\widetilde{p}(x)$。在负相中，我们通过降低从模型分布中采样的$log\widetilde{p}(x)$来降低配分函数。

##### 随机最大似然和对比散度
上式的一个朴素方法是，每次需要计算梯度时，磨合随机初始化的一组马尔可夫链。训练过程如下:

![naiveMCMC](F:\mycode\knowledgeArrangement\deepLearning\naiveMCMC.png)

内循环中磨合马尔可夫链的计算代价过高，导致这个过程在实际中是不可行的，但是这个过程是其他更加实际的近似算法的基础。可以将最大化似然的MCMC方法视为在两种力之间平衡：一种力拉高数据出现时的模型分布$\triangledown _{\theta }log\widetilde{p}\left ( x;\theta  \right )$，一种拉低模型采样出现时的模型分布$\triangledown _{\theta }logZ$。

因为负相涉及从模型分布中抽样，所以可以认为它在找模型信任度很高的点。因为负相减少了这些点的概率，它们一般被认为代表了模型中不正确的信念。

一个自然的 解决方法是初始化马尔可夫链为一个非常接近模型分布的分布，从而大大减少磨合步骤：

![CDMCMC](F:\mycode\knowledgeArrangement\deepLearning\CDMCMC.png)

**对比散度**（CD，或者是具有k个Gibbs步骤的CD-k）算法在每个步骤中初始化马尔可夫链为采样自数据分布中的样本。初始时，数据分布并不接近模型分布，因此负相不是非常准确，但正相仍然可以准确地增加数据的模型概率。进行正相阶段一段时间之后，模型分布会更接近于数据分布，并且负相开始变得准确。

==关于对比散度不是很懂==

随机最大似然算法：

![sml](F:\mycode\knowledgeArrangement\deepLearning\sml.png)

在k太小或$\epsilon$太大时，随机梯度算法移动模型的速率比马尔可夫链在迭代步中混合更快，此时SML容易变得不准确。这些值的容许范围高度依赖于具体问题。现在还没有方法能够正式地测试马尔可夫链是否能够在迭代步骤之间成功混合。

##### 伪似然

蒙特卡罗近似配分函数及其梯度需要直接处理配分函数。有些其他方法通过训练不需要计算配分函数的模型来绕开这个问题。这些方法大多数都基于以下观察：无向概率模型中很容易计算概率的比率。这是因为配分函数同时出现在比率的分子和分母中，互相抵消：
$$
\frac{p(x)}{p(y)}=\frac{\frac{1}{Z}\widetilde{p}(x)}{\frac{1}{Z}\widetilde{p}(y)}=\frac{\widetilde{p}(x)}{\widetilde{p}(y)}
$$
假设我们将x分为a、b和c，其中a包含我们想要的条件分布的变量，b包含我们想要条件化的变量，c包含除此之外的变量：
$$
p(a|b)=\frac{p(a,b)}{p(b)}=\frac{p(a,b)}{\sum_{a,c}p(a,b,c)}=\frac{\widetilde{p}(a,b)}{\sum_{a,c}\widetilde{p}(a,b,c)}
$$

为了计算对数似然，我们需要边缘化很多变量。如果总共有n个变量，那么我们必须边缘化n-1个变量。根据概率的链式法则，我么有：
$$
logp(x)=logp(x1)+logp(x2|x1)+\cdots+logp(xn|x_{1:n-1})
$$
在这种情况下，我们已经使得a尽可能小，但是c可以达到$x_{2:n}$。如果简单地将c移到b中以减小计算代价，那么会发生什么呢？这便产生了伪似然目标函数，给定所有其他特征$x_{-i}$，预测特征$x_{-i}$的值：
$$
\sum_{i=1}^{n}logp(x_{i}|x_{-i})
$$
可以证明最大化伪似然的估计是渐进一致的。在数据集不趋近于大采样极限的情况下，伪似然可能表现出与最大似然估计不同的结果。

用广义伪似然估计来权衡计算复杂度和最大似然表现的偏差。广义伪似然估计使用m个不同的集合$S^{(i)},i=1,\cdots,m$作为变量的指标出现在条件棒的左侧。在$m=1$和$S^{(1)}=1,\cdots,n$的极端情况下，广义伪似然估计会变为对数似然。在$m=n$和$S^{(i)}=\left \{ i \right \}$在极端情况下，广义伪似然会恢复为伪似然。广义伪似然目标函数如下所示：
$$
\sum_{i=1}^{m}logp(X_{S^{(i)}}|X_{-S^{(i)}})
$$
基于伪似然的方法的性能在很大程度上取决于模型是如何使用的。对于完全联合分布$p(x)$模型的任务，伪似然效果通常 不好。对于在训练期间只需要使用条件分布的任务而言，它的效果比最大似然要好，例如填充少量的缺失值。如果数据具有规格结构，使得S索引集可以被设计为表现最为重要的相关性质，同时略去相关性可忽略的变量，那么广义伪似然策略会非常有效。

（未完待续。。。。 ）

##### 得分匹配和比率匹配

##### 去噪得分匹配

##### 噪声对比估计

##### 估计配分函数

