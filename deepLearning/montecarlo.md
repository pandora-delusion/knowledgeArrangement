# 蒙特卡罗方法
随机算法：Las Vegas算法和monte carlo算法

##### 采样和蒙特卡罗方法
   monte carlo采样的基础
    当我们无法精确计算和或积分时，通常可以使用monte carlo采样来近似它。这种想法把和或者积分视作某种分布下的期望，然后通过估计对应的平均值来近似这个期望。$$s= \sum _{x}p\left ( x \right )f\left ( x \right )=E_{p}\left [ f\left ( x \right ) \right ]$$ $$s= \int p\left ( x \right )f\left ( x \right )dx=E_{p}\left [ f\left ( x \right ) \right ]$$ 
    P是一个关于随机变量x的概率分布（求和时）或者概率密度函数（求积分时）。
    我们可以通过从p中抽取n个样本$x^{(1)},\cdots,x^{(n)}$来近似s并得到一个经验平均值：$$\widetilde{s}_{n}=\frac{1}{n}\sum _{i=1}^{n}f\left ( x^{(i)} \right )$$ 很明显，$\widehat{s}$是无偏的，而且根据大数定理，若$x^{(i)}$是独立同分布的，那么其平均值必然收敛到期望值，即$$\lim_{n\rightarrow \infty }\widehat{s}_{n}=s$$只需要满足各个单项的方差$Var[f(x^{(i)})]$有界。考虑当n增大时$\widehat{s}_{n}$的方差，只要满足$Var[f(x^{(i)})]<\infty$，方差$Var[\widehat{s}_{n}]$就会减小并收敛到0：$$Var[\widehat{s}_{n}]=\frac{Var[f(x)]}{n}$$ 中心极限定理说明$\widehat{s}_{n}$的分布收敛到以s为均值以$\frac{Var[f(x)]}{n}$为方差的正态分布。这使得我们可以使用正态分布的立即函数来估计$\widehat{s}_{n}$的置信区间。
    当我们无法从p中采样时，一种方案是使用重要采样，另一种是构建一个收敛到目标分布的估计序列（马尔可夫链蒙特卡罗方法）。
### 重要采样
在monte carlo方法中，我们感兴趣的是估计f(x)在概率分布p(x)下的期望。p(x)f(x)可以被写成：$$p(x)f(x)=q(x)\frac{p(x)f(x)}{q(x)}$$我们可以从q分布中采样，然后估计$\frac{pf}{q}$在此分布下的均值。
从上式的关系中可以发现，任意monte carlo估计可以被转化为一个重要采样的估计：$$\widehat{s}_{q}=\frac{1}{n}\sum _{i=1,x^{\left ( i \right )}\sim q}\frac{p\left ( x^{\left ( i \right )} \right )f\left ( x^{\left ( i \right )} \right )}{q\left ( x^{\left ( i \right )} \right )}$$估计的期望于q分布无关：$$E_{q}\left [ \widehat{s}_{q} \right ]=E_{p}\left [ \widehat{s}_{p} \right ]=s$$ 然而，重要性采样的方差对q的选择非常敏感。这个方差可以表示为：$$Var[\widehat{s}_{q}]=Var[\frac{p(x)f(x)}{q(x)}]/n$$方差想要取到最小值，q需要满足$$q^{*}(x)=\frac{p(x)\left | f(x) \right |}{Z}$$Z表示归一化常数，选择适当的Z使得$q^{*}(x)$之和或者积分为1。当f(x)的正负符号不变时，$Var[\widehat{s}_{q^{*}}]=0$，这意味着当使用最优的q分布时，只需要一个样本就足够了。在实践中这种只需要采样一个样本的方法往往是无法实现的。

有偏重要采样(biased importance sampling)，不需要归一化p或者q分布。在处理离散变量时，有偏重要采样估计可以表示为：$$\widehat{s}_{BIS}=\frac{\sum _{i=1}^{n}\frac{p\left ( x^{\left ( i \right )} \right )}{q\left ( x^{\left ( i \right )} \right )}f\left ( x^{\left ( i \right )} \right )}{\sum _{i=1}^{n}\frac{p\left ( x^{\left ( i \right )} \right )}{q\left ( x^{\left ( i \right )} \right )}}=\frac{\sum _{i=1}^{n}\frac{p\left ( x^{\left ( i \right )} \right )}{\widetilde{q}\left ( x^{\left ( i \right )} \right )}f\left ( x^{\left ( i \right )} \right )}{\sum _{i=1}^{n}\frac{p\left ( x^{\left ( i \right )} \right )}{\widetilde{q}\left ( x^{\left ( i \right )} \right )}}=\frac{\sum _{i=1}^{n}\frac{\widetilde{p}\left ( x^{\left ( i \right )} \right )}{\widetilde{q}\left ( x^{\left ( i \right )} \right )}f\left ( x^{\left ( i \right )} \right )}{\sum _{i=1}^{n}\frac{\widetilde{p}\left ( x^{\left ( i \right )} \right )}{\widetilde{q}\left ( x^{\left ( i \right )} \right )}}$$ 其中$\widetilde{p}$和$\widetilde{q}$分别是分布p和q的未经归一化的形式$x^{(i)}$是从分布q中抽取的样本。这种估计是有偏的。只有当$n\rightarrow \infty $且最左边的式子的分母收敛到1时等式才渐进的成立。所以这一估计也没称为渐进无偏的。

好的q分布的选择可以显著提高monte carlo估计的效率，而一个糟糕的q分布选择会使效率更糟。q分布经常会取一些简单常用的分布使得能够从q分布中方便的采样。当x时高维数据时，q分布的简单性使得它很难与p或者p|f|相匹配。

### 马尔可夫链蒙特卡罗方法
即MCMC方法。
MCMC技术最标准、最一般的理论保证只适用于那些个状态概率都不为0的模型。事实上，MCMC方法可以被广泛应用在包含0概率状态的许多概率分布中，然而，在这种情况下，关于MCMC方法性能的理论保证只能依据不同类型的分布具体分析证明。
马尔可夫链有一个随机状态x和一个转移分布$T(x'|x)$定义而成，$T(x'|x)$是一个概率分布，运行一个马尔可夫链意味着根据转移分布$T(x'|x)$采出的值x'来更新状态x。

我们可以用一个向量v来描述这个概率分布q：$$q(x=i)=v_{i}$$考虑更新单一的马尔可夫链，从状态x到新状态x'。单一状态转移到x'的概率可以表示为：$$q^{(t+1)}(x')=\sum_{x}q^{(t)}(x)T(x'|x)$$ 将转移算子T表示成一个矩阵A。矩阵A的定义如下：$$A_{i,j}=T(x'=i|x=j)$$ 可以用v和A来描述当我们更新时（并行运行的）不同马尔可夫链上的整个分布式如何变化的：$$v^{(t)}=Av^{(t-1)}$$ 重复的使用马尔可夫链更新相当于重复地与矩阵A相乘，这一过程就是关于A的幂乘：$$v^{(t)}=A^{(t)}v^{(0)}$$ A --- 随机矩阵。如果对于任意状态x到任意其他状态x'存在一个t使得转移概率不为0。那么Perron-Frobenius定理可以保证这个矩阵的最大特征值是实数且大小为1。我们可以看到所有的特征值随着时间呈现指数变化：$$v^{\left ( t \right )}=\left ( V diag\left ( \lambda  \right )V^{-1} \right )^{t}v^{\left ( 0 \right )}=Vdiag\left ( \lambda  \right )^{t}V^{-1}v^{\left ( 0 \right )}$$这个过程导致了所有不等于1的特征值都会衰减到0。在一些额外的较为宽松的假设下，我们可以保证矩阵A只有一个对应特征值为1的特征向量。所以这个过程收敛到平稳分布。收敛时，我们得到$$v'=Av=v$$。作为收敛的稳定点，v一定时特征值为1所对应的特征向量。这个条件保证了收敛到了平稳分布以后，在重复转移采样过程不会改变所有不同马尔可夫链上状态的分布。

如果我们正确地选择了转移算子T，那么最终的平稳分布q将会等于我们所希望采样的分布p。无论状态时连续的还是离散的，所有的马尔可夫链方法都包括重复、随机地更新直到最后状态开始从均衡分布中采样。运行马尔可夫链直到它达到均衡分布的过程通常被称为马尔可夫链的磨合过程。

另一个难点时无法预知马尔可夫链需要运行多少步才能达到均衡分布。这段时间通常被称为混合时间。

### Gibbs采样
我们通常使用马尔可夫链从定义为基于能量的模型的分布$P_{model}(x)$中采样。在这种情况，我们希望马尔可夫链的q(x)分布就是$P_{model}(x)$。为了得到所期望的q(x)分布，我们必须采取合适的T(x'|x)。
Gibbs构造一个从$P_{model}(x)$中采样的马尔可夫链，其中在基于能量的模型中从$T(x'|x)$采样是通过选择一个变量$x_{i}$，然后从$P_{model}$中该点关于在无向图G（定义了基于能量的模型结构）中邻接点的条件分布中采样。只要一些变量在给定相邻变量时是条件独立的，那么这些变量就可以被同时采样。同时更新许多变量的Gibbs采样通常被称为块吉布斯采样。

### 不同的峰值之间的混合挑战
（暂时不看，看不懂）深度学习P365