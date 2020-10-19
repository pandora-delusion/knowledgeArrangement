# Gradient Harmonized Single-Stage Detector

​	正负样本，难易样本在数量上的巨大差异是困扰one-stage目标检测的主要问题。大量容易判别的背景样本往往压倒了培训。

​	Focal loss是一种静态损失，不适应数据分布的变化，数据分布随着训练过程的变化而变化。

​	本文首先指出，分类的不平衡可以概括为判别难度的不平衡，难度的不平衡可以概括为梯度范数的不平衡

​	如果正样本能被很好地分类，那么那它就是一个简单的样本，模型从中获益甚少。而一个错误分类的样本应该引起模型的注意，不管它属于哪个类。因此，从宏观上看，大量的负面样本往往很容易分类，而困难的样本通常是正样本。

​	本文认为不同属性的样本的不平衡可以通过梯度范数的分布得到表现。如下图：

![image-20191119102530664](F:\mycode\knowledgeArrangement\vision\gradient_harmonized_1.png)

​	左上图显示了在一个已经收敛的模型中，梯度范数很小的样本密度很大，这与大量容易学习的负样本相对应。即使一个容易学习的样本对总梯度的贡献比一个难学习的样本小，大量容易学习的样本的总贡献超过了少数难学习样本的贡献，训练过程将是低效的。同时，本文发现梯度范数非常大的样本（难以学习的样本）的密度略大于难度中等样本的密度。这些难以判别的样本大多是异常值，因为它们即使在模型收敛时也是稳定存在的。这些异常值可能会影响模型的稳定性，因为它们们的梯度可能与其他常见的例子有很大的差异。

​	在分析梯度范数分布的基础上，提出了一种基于梯度协调机制的one-stage目标检测模型的训练方法。GHM首先对具有相似属性的样本的数量进行统计。然后根据密度对每个样本的梯度加入一个调和参数。使用GHM进行训练时，简单的样本产生的大量累积梯度可以很大程度上降权，异常值也可以相对降权。最后，平衡个样本的贡献，使训练更加高效，稳定。

​	在实践中，通过重构损失函数来等效地实现梯度的修正，将GHM嵌入到分类损失中，即GHM-C损失。

### Gradient Harmonizing Mechanism

​	对于一个候选边框，令$p \in [0,1]$为模型的预测概率，$p^{*} \in \left \{0, 1 \right \}$为gt标签。考虑二叉熵损失如下：
$$
L_{CE}  ( p,p^{*} )=\left\{\begin{matrix}
-log\left ( p \right ) & if\quad p^{*}=1 \\ 
-log\left ( 1-p \right )& if\quad p^{*}=0
\end{matrix}\right.
$$
令x为模型的直接输出$p=sigmoid(x)$，x对应的梯度为：
$$
\frac{\partial L_{CE}}{\partial x}=\left\{\begin{matrix}
p-1 & if\quad p^{*}=1\\ 
p & if\quad p^{*}=0 
\end{matrix}\right.
$$
定义g如下：
$$
g=\left | p-p^{*} \right |=\left\{\begin{matrix}
1-p & if \quad p^{*}=1\\ 
p & if \quad p^{*}=0
\end{matrix}\right.
$$
g代表了样本的属性（easy or hard），并且显示样本对全局梯度的影响。

![image-20191119112813626](F:\mycode\knowledgeArrangement\vision\gradient_harmonized_2.png)

​	从上图我们可以看到容易学习的样本的数量非常大，对全局梯度有非常大的影响。而且还可以看到收敛的模型依然不能处理一些非常难的样本，它们的数量甚至比中等难度的样本还要多。这些非常困难的样本可以被视为离群值，因为它们的梯度方向往往与大量其他l样本的梯度方向有很大的差异。如果收敛的模型被迫学习如何更好地对这些异常值进行分类，则对大量其他样本的分类往往不是那么的准确

#### 梯度密度

正对梯度范数的不协调问题，提出了一种梯度密度的协调方法，训练样本的梯度密度函数如下：
$$
GD(g)=\frac{1}{l_{\epsilon }(g)}\sum_{k=1}^{N}\delta _{\epsilon}(g_{k}, g)
$$
$g_{k}$是第k个样本的梯度范数，并且：
$$
\begin{align*}
&\delta _{\epsilon }\left ( x,y \right )=\left\{\begin{matrix}
1 & if \quad y-\frac{\epsilon }{2}\leq x\leq y+\frac{\epsilon }{2} \\ 
0 & otherwise
\end{matrix}\right.\\
&l_{\epsilon }\left ( g \right )=min\left (  g+\frac{\epsilon }{2},1\right )-max\left ( g-\frac{\epsilon }{2},0\right )
\end{align*}
$$
g的梯度密度指代以g为中心的长度为$\epsilon$并被区域的合法长度归一化的区域中样本的数量。现在定义梯度密度协调参数为：
$$
\beta_{i}=\frac{N}{GD(g_{i})}
$$
N为样本总数。$GD(g_{i})/N$是一个标准化系数指代第i个样本的邻域梯度的样本的比重，若样本均匀分布，则$GD(g_{i})=N \quad \beta_{i}=1$。否则，密度较大的算例将被归一化器相对降权。

#### GHM-C loss

$$
\begin{align*}
L_{GHM-C}&=\frac{1}{N}\sum_{i=1}^{N}\beta_{i}L_{CE}(p_{i}, p_{i}^{*})\\
&=\sum_{i=1}^{N}\frac{L_{CE}(p_{i}, p_{i}^{*})}{GD(g_{i})}
\end{align*}
$$

下图展示了不同损失的规范化后的梯度范数：

![image-20191119121843136](F:\mycode\knowledgeArrangement\vision\gradient_harmonized_3.png)

​	我们看到focal loss的曲线和GHM-C loss的曲线有相似的趋势，这表明有最好参数的focal loss与GHM类似。而GHM有focal loss不具备的优势：降低离群值的梯度贡献的权重。

​	使用GHM-C损失，大量的容易判别的样本被大量地降权，异常值也被轻微地降权，这同时解决了属性不平衡问题很和异常值问题。

#### Unit Region Approximation

​	上述算法计算量太大，这里提出了一种近似计算的方法。

将g的取值空间分成有限个长度为$\epsilon$的单元区间，那么就会有$M=\frac{1}{\epsilon}$个单元区域。令$r_{j}$为第$j$个单元区域，这样$r_{j}=[(j-1)\epsilon,j\epsilon)$。令$R_{j}$为在$r_{j}$范围内的样本数量。定义$ind(g)=t \quad s.t. \quad (t-1)\epsilon\leq g\lt t\epsilon$,这是单元区域的索引函数。

​	定义近似梯度密度函数如下：
$$
\widehat{GD(g)}=\frac{R_{ind(g)}}{\epsilon}=R_{ind(g)}M
$$
​	近似梯度协调参数如下：
$$
\widehat{\beta_{i}} = \frac{N}{\widehat{GD}(g_{i})}
$$
​	最终损失函数如下：
$$
\begin{align*}
\widehat{h}_{GHM-C}&=\frac{1}{N}\sum_{i=1}^{N}\widehat{\beta_{i}}L_{CE}(p_{i}, p_{i}^{*})\\
&=\sum_{i=1}^{N}\frac{L_{CE}(p_{i}, p_{i}^{*})}{\widehat{GD}(g_{i})}
\end{align*}
$$

##### EMA

上面的近似算法在minibatch的情况下会面对一个问题：当许多极端值被采样到一个minibatch中，那么统计结果将是一个严重的噪声并使得训练不稳定。Exponential moving average(EMA)是一个解决问题的常见方法。因为在近似算法中梯度密度来自单元区域的样本数量，我们可以应用EMA在每个单元区域上来获得更加稳定的样本梯度密度。令$R_{j}^{t}$为第j个单元区域第t个迭代的样本数量，$S_{j}^{t}$是为移动平均值，那么：
$$
S_{j}^{t}=\alpha S_{j}^{t-1} + (1-\alpha)R_{j}^{t}
$$
$\alpha$为momentum参数，使用$S_{j}$替代$R_{j}$：
$$
\widehat{GD}(g) = \frac{S_{ind(g)}}{\epsilon}=S_{ind(g)}M
$$
使用EMA，梯度密度将变得更加平滑和对极端值不敏感。

#### GHM-R loss

​	考虑参数化的偏置$t=(t_{x}, t_{y}, t_{w}, t_{h})$，目标偏置为$t^{*}=(t_{x}^{*}, t_{y}^{*}, t_{w}^{*}, t_{h}^{*})$。回归损失通常采用smooth L1损失函数：
$$
\begin{align*}
L_{reg}=\sum_{i \in \left \{ x,y,w,h \right \}}SL_{1}(t_{i}-t_{i}^{*})\\
SL_{1}(d)=\left\{\begin{matrix}
\frac{d^{2}}{2\delta } & if \left | d \right | \leq \delta \\ 
\left | d \right |-\frac{\delta }{2} & otherwise
\end{matrix}\right.
\end{align*}
$$
​	因为$d=t_{i}-t_{i}^{*}$，smooth L1的梯度可以被表示为：
$$
\frac{\partial SL_{1}}{\partial t_{i}}=\frac{\partial SL_{1}}{\partial d}=\left\{\begin{matrix}
\frac{d}{\delta } & if \quad \left | d \right |\leq \delta \\ 
sgn(d) & otherwise 
\end{matrix}\right.
$$
​	

​	这种梯度使得梯度范数作为区分样本的属性是不可能的，为了方便在回归损失上使用GHM，使用一个更优雅的形式替换传统的SL函数：
$$
ASL_{1}(d)=\sqrt{d^{2}+\mu ^{2}}-\mu
$$
这意味着在所有位置导数都存在，这样$ASL_{1}$的梯度为
$$
\frac{\partial ASL_{1}}{\partial d}=\frac{d}{\sqrt{d^{2}+\mu ^{2}}}
$$
梯度函数的值域为$[0,1)$。当$\mu=0.02$时$ASL_{1}$损失保持和$SL_{1}$同样的性能。这样$ASL_{1}$的梯度范数为$gr=\left \|  \frac{d}{\sqrt{d^{2}+\mu^{2}}} \right \|$

注意回归仅仅在正样本上执行，因此分类和回归的分布趋势不同是合理的，同样，可以在回归损失上使用GHM：
$$
\begin{align*}
L_{GHM-R}&=\frac{1}{N}\sum_{i=1}^{N}\beta_{i}ASL_{1}(d_{i})\\
&=\sum_{i=1}^{N}\frac{ASL_{1}(d_{i})}{GD(gr_{i})}
\end{align*}
$$
在边框回归中，并不是所有的容易学习的样本是不重要的。简单样本在分类问题中通常是有着低预测概率的背景区域，坑定灰白排除来最终候选区域之外，因此这种样本的改进对于精确程度没有贡献。但是在边框回归中，容易学习的样本任然偏离地面真值位置。更好地预测任何实例都将提高最终候选对象的质量。我们的GHM-R损失可以通过增加简单样本的重要部分的权重和减少异常值的权重来协调简单样本和困难样本对边框回归的贡献。

