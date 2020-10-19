# Sequeeze and Excitation Networks

​	seNet通过显式地建模卷积特征之间的相关性来达到提高网络产生的表示（特征）的质量。

​	这种机制允许网络执行特征重新校准（recalibration）， 通过该机制，它可以学习使用全局信息，选择性地强调信息特征，并抑制不太有用的特征。

![1568525542571](F:\mycode\knowledgeArrangement\vision\seNet.png)

​	seNet的结构如上图所示，对于任何给定的转换$F_{tr}$映射输入X到特征图U，$U \in R^{H \times W \times C}$，可以构造对应的SE模块来执行特征重新校准。特征U首先通过squeeze操作，该操作通过聚合跨空间维度的特征映射来生成通道描述符。这个描述符的功能是产生一个全局分布的通道特征响应，允许来自网络的全局接受域的信息被它的所有层使用。之后是excitation操作，采用简单的全连接的形式，将squeeze的输出作为输入，并产生每个通道的调制权值的聚合。这些权值被应用到特征映射U中，生成SE模块的输出，可以直接输入到网络的后续层中。

​	通过简单地在现有的网络模型的基础上叠加SE快，就可以构造一个SE网络。

​	SE块的结构十分简单，可以直接在现有的最先进的体系结构中使用，方法是用SE组件替换原来的组件，从而有效地提高性能。

​	虽然构建块的模板是通用的，但是在整个网络中，它在不同深度执行的职责是不同的。在早期的层中，它以一种与类无关的方式激发信息特征，从而增强了共享的底层信息。在较深的层中，SE模块变得越来越专门化，并以高度特定于类的方式响应不同的输入。这样，SE模块执行的特征重新校准的优势可以同过网络累积。

​	在传统的卷积网络中，由于输出是通过对所有通道的求和产生的，所以通道依赖关系隐式地嵌入到过滤器中，但是与过滤器捕获的本地空间相关性纠缠在一起。卷积所模拟的信道关系本质上是隐式的和局部的。

​	seNet通过显式地对信道相互依赖关系建模来增强对卷积特征的学习，从而使网络能够提到其对信息特征的敏感性，这些信息特征可以通过后续的转换加以利用。

#### Squeeze：全局信息嵌入

​	这里通过使用全局平均池来生成各个通道的统计量来实现的

​	
$$
Z_{c}=F_{sq}(U_{c})=\frac{1}{H \times W} \sum_{i=1}^{H} \sum_{j=1}^{W}U_{c}(i,j)
$$
​	（在以前的特征工程工作中，利用这些信息使很普遍的）

#### Excitation：自适应重构

​	为了利用在Squeeze操作中聚合的信息，Excitation希望能捕获这种通道依赖关系。为了实现这一目标，该职能必须满足两项标准：1.它必须是灵活的（特别是它必须能够学习通道之间的非线性关系）；2.它必须学会一种非相互排斥的关系，因为希望能够保证强调多个通道

​	为了满足以上两个条件，论文提出了以下简单的结构：
$$
s=F_{ex}(z, W)=\sigma (g(z, W))=\sigma (W_{2} \delta (W_{1}z))
$$
​	$\delta$指代ReLU函数，$W_{1} \in R^{\frac{C}{\gamma} \times C}$，$W_{2} \in R^{C \times \frac{C}{\gamma}}$。为了限制模型的复杂性和便于推广，在非线性层周围形成两个全联通层的瓶颈，即降维比为$\gamma$的降维层。设置$\gamma$为16可以很好地平衡精度和复杂性

​	在Excitation的FC层中去掉bias，有利基于信道关系的建模

​	模块的最终输出是通过使用激活s对U进行缩放得到的：
$$
\widetilde{X_{c}}=F_{scale}(U_{c},S_{c})=S_{c}U_{c}
$$
​	SE块本质上输入上的动态条件，它可以被看作是信道上的自注意函数，其关系并不局限于卷积滤波器所响应的局部接受域。

​	注意虽然SE模块本身增加了模型的深度，但是它是以一种非常高效的计算方式增加了深度，并且即使在扩展基础架构的深度达到收益递减的程度时，也会产生良好的收益

![1568531207742](F:\mycode\knowledgeArrangement\vision\se-resnet.png)

```python
from torch import nn

class SELayer(nn.Module):
    
    def __init__(self, channel, reduction=16):
        super(SELayer, self).__init__()
        # [batch, channel, 1, 1]
        self.avg_poool = nn.AdaptiveAvgPool2d(1)
        self.fc = nn.Sequential(
            nn.Linear(channel, channel // reduction, bias=False),
            nn.ReLU(inplace=True),
            nn.Linear(channel // reduction, channel, bias=False),
            nn.Sigmoid()
        )
        
    def forward(self, x):
        b, c, _, _ = x.size()
        # [batch, channel]
        y = self.avg_pool(x).view(b, c)
        # [batch, channel, 1, 1]
        y = self.fc(y).view(b, c, 1, 1)
        return x*y.expand_as(x)
```