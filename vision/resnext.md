# Aggregated Residual Transformations for Deep Neural Networks

​	VGG-net的Inception等网络结构的重要常见属性就是"split-transform-merge"策略。在Inception模块中，输入被分割成一系列低维嵌入层（通过$1 \times 1$卷积降维），然后分别通过不同的过滤器（$3 \times 3，$$5 \times 5$），最后通过连接操作把它们归并起来。Inception的这种模块的"split-transform-merge"策略有望接近大而密的层的表示能力，但是计算复杂度要低得多。

​	Inception需要手动设计各种超参数，在 适应新的数据集/任务时不方便

​	实验证明，增加基数是获得精度的更有效手段，而不是让模型更深更宽，特别是当模型的深度和宽度开始降低现有模型的回报时。

​	利用分组卷积来提高进度的证据很少

​	这种手段和集成模型时不同的，因为我们的手段是通过联合训练获得的，而不是独立训练获得的。

​	本文提出的聚合转换如下：
$$
F(x)=\sum_{i=1}^{C}T_{i}(x)
$$
$T_{i}(x)$可以是任意函数，类似于一个简单的神经网络，$T_{i}(x)$应该将x投影到一个（可选的低维）嵌入层，然后对它做转换。

​	实验表明，基数（分组数）是一个重要的维度，它比宽度和深度更加有效。

![1569033157075](F:\mycode\knowledgeArrangement\vision\resnext.png)

​	上图三中形式严格等价，(c)图是Resnext的分组卷积形式，所有的低维嵌入层可以被一个更大的层替换

​	注意，只有当块的深度大于等于3时，上图才会产生有效的拓扑结构。

参数的复杂性和个数代表了模型的固有容量，因此常被作为深度网络的基本性质来研究。
