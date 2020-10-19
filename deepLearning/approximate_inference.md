# 近似推断

推断困难指的是难以计算$P(h|v)$或其期望，这些操作在最大似然学习的任务中是必须的。

### 把推断视作优化问题

精确推断问题可以被描述为一个优化问题，有许多方法正是由此解决了推断的困难。通过近似这样一个潜在的优化问题，我们往往可以推导出近似推断算法。

计算观察数据的对数概率$logp(v;\theta)$，可以计算一个$logp(v;\theta)$的下界$L(v,\theta,q)$，被称为***证据下界***或者***变分自由能***：
$$
L(v,\theta,q)=logp(v;\theta)-D_{KL}(q(h|v)||p(h|v;\theta))
$$
其中q是关于h的一个任意概率分布。

$logp(v)$和$L(v,\theta,q)$之间的距离是由KL散度来衡量的，且KL散度总是非负的，所以$L(v,\theta,q) \leq logp(v;\theta)$。当且仅当分布q完全相等于$p(h|v)$时取等号。

对于一些分布q，计算L可以很简单：
$$
\begin{equation}
\begin{aligned}
L(v,\theta,q)&=logp(v;\theta)-D_{KL}(q(h|v)||p(h|v;\theta))\\
&=logp(v;\theta)-E_{h\sim q}log \frac{q(h|v)}{p(h|v)}\\
&=logp(v;\theta)-E_{h\sim q}log \frac{q(h|v)}{\frac{p(h,v;\theta)}{p(v;\theta)}}\\
&=logp(v;\theta)-E_{h\sim q}[logq(h|v)-logp(h,v;\theta)+logp(v;\theta)]\\
&=-E_{h\sim q}[logq(h|v)-logp(h,v;\theta)]
\end{aligned}
\end{equation}
$$
这也给出了证据下界的标准定义：
$$
L(v,\theta,q)=E_{h\sim q}[logp(h,v)]+H(q)
$$
对于任意分布q的选择来说，L提供了似然函数的一个下界。越好地近似$p(h|v)$的分布$q(h|v)$，得到的下界就越紧，就与$logp(v)$越近。当$q(h|v)=p(h|v)$时，这个近似就是完美的，也意味着$L(v,\theta, q)=logp(v;\theta)$。

这样我们可以将近似推断问题看作找一个分布q使得L最大的过程。我们可以通过限定分布q的形式或者使用并不彻底的优化方法来使得优化的过程更加高效（却更粗略），但是优化的结果是不完美的，不求彻底地最大化L，而要显著地提升L。

### 期望最大化

期望最大化算法（expectation maximization, EM），EM并不是一个近似推断算法，而是一种能够学到近似后验的算法。

EM算法交替迭代，直到收敛的两步运算组成：

+ E步(expectiation step)：令$\theta^{（0）}$，表示在这一步开始时的参数值。对任何我们想要训练的（对所有的或者小批量数据均成立）索引为i的训练样本$v^{(i)}$，令$q(h^{(i)}|v)=p(h^{(i)}|v^{(i)};\theta^{(0)})$。通过这个定义，我们认为q在当前参数$\theta^{(0)}$下定义。如果我们改变$\theta$，那么$p(h|v;\theta)$将会相应地变化，但是$q(h|v)$还是不变并且等于$p(h|v;\theta^{(0)})$。
+ M步(maximization step)：使用选择的优化算法完全地或者部分地关于$\theta$最大化$\sum_{i}L(v^{(i)}, \theta, q)$

坐标上升法在最大化L。

###  最大后验推断

***推断***：指代给定一些其他变量的情况下计算某些变量概率分布的过程

当训练带有潜变量的概率模型时，我们通常关注于计算$p(h|v)$。另一种可选的推断形式是计算一个缺失变量的最可能值来代替在所有可能值的完整分布上的推断。在潜变量模型中，这意味着计算：
$$
h^{*}=argmax_{h}p(h|v)
$$
这被称之为最大后验推断，简称MAP推断。MAP推断并不被视为一种近似推断，它只是精确地计算了最优可能的一个$h^{*}$。如果我们希望设计一个最大化$L(v,h,q)$的学习过程，那么把MAP推断视作是输出一个q值的学习过程是很有帮助的。在这种情况下，我们将MAP推断视为近似推断，因为它并不能提供一个最优q。

精确推断，指的是关于一个在无限制的概率分布族中的分布q使用精确的优化算法来最大化：
$$
L(v,\theta,q)=E_{h\sim q}[logp(h,v)]+H(q)
$$
我们通过限定分布q数据某个分布族，能够使得MAP推断称为一种形式得近似推断。具体地说，我们令分布q满足一个Dirac分布：
$$
q(h|v)=\delta (h-\mu)
$$
我们可以通过$\mu$来完全控制分布q。将L中不随$\mu$变化的项丢弃，我们只需解决一个优化问题：
$$
\mu^{*}=argmax_{\mu}logp(h=\mu,v)
$$
这等价于MAP推断问题：
$$
h^{*}=argmax_{h}p(h|v)
$$

### 变分推断和变分学习

变分学习的核心思想就是在一个关于q的有约束的分布族上最大化L。选择这个分布族时应该考虑到计算$E_{q}logp(h,v)$的难易度。一个典型的方法就是添加分布q如何分解的假设。

一种常用的变分学习的方法是加入一些限制使得q是一个因子分布：
$$
q(h|v)=\prod q(h_{i}|v)
$$
这被称为***均值场***方法。更一般的说，可以通过选择分布q的形式来选择任何图模型的结构，通过选择变量之间相互作用的多少来灵活地决定近似程度的大小。这种完全通用的图模型方法被称为***结构化变分推断***

变分法的优点是，我们不需要为分布q设定一个特定的参数化形式。我们设定它如何分解，之后通过解决优化问题来查出这些分解限制下最优的概率分布。使用**变分法**（对离散型潜变量没啥用）。

因为$L(v, \theta, q)$被定义成$logp(v;\theta)-D_{KL}(q(h|v)||p(h|v;\theta))$，可以认为关于q最大化L的问题等价于最小化$D_{KL}(q(h|v)||p(h|v;\theta))$。这种情况下，我们用q来拟合p。当我们使用最大似然估计来用模型拟合数据时，我们最小化$D_{KL}(P_{data}||P_{model})$。

最小化$D_{KL}(p||q)$：选择一个q，使得它在p具有高概率的地方具有高概率

最小化$D_{KL}(q||p)$：选择一个q，使得它在p具有低概率的地方具有低概率

这意味着最大似然鼓励模型在那个数据达到高概率的地方达到高概率，而基于优化的推断则鼓励了q在每一个真实后验分布概率低的地方概率较小。

（未完待续。。。。）

##### 离散型潜变量

##### 变分法

##### 连续型潜变量

##### 学习和推断之间的相互作用

### 学成近似算法
