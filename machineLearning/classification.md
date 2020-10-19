 ## softmax分类器和SVM分类器

> softmax分类器(Multinomial Logistic Regression)
> softmax只是二项logistic regression在多维度上的推广

### 线性SVM分类器
> hinge损失函数
> $L_{i}=\sum_{j\neq y_{i}}max\left ( 0,S_{j}-S_{y_{i}}+\Delta  \right )$
> $L_{i}$表示第i个样本的loss函数，$\Delta$的具体值没有意义，有意义的是它相对于其他分类上的得分的大小。（查看周志华《机器学习》）
> SVM分类器对于一小撮接近于分类边界的样例比较敏感，对远离边界很远的样例点不敏感；而softmax是一个基于所有数据的函数，对于每一个样例点都有所考量  SVM损失函数没有被严格证明是完全可导的
$\frac{1}{n}\sum_{i=1}^{n}\sum_{j\neq y_{i}}max\left ( 0,S_{j}-S_{y_{i}}+\Delta  \right )+\frac{\lambda }{2}\left \| \omega  \right \|^{2}$

### [Attribute classification, Regression, Structured prediction](http://cs231n.github.io/neural-networks-2/)

### Softmax分类器
> $P_{k}=\frac{e^{f_{k}}}{\sum_{j}e^{f_{j}}}$
损失函数：
$L_{i}=-log\left ( \frac{e^{s_{y_{i}}}}{\sum_{j}e^{s_{j}}} \right ) $ -- 交叉熵损失函数
$L=\frac{1}{N}\sum_{i}L_{i}+\frac{1}{2}\lambda \sum_{k}\sum_{l}\omega _{k,l}^{2}$
$\frac{\partial L}{\partial f_{k}}=\frac{1}{N}(P_{k}-1\left ( y_{i}=k \right ))$
数值稳定性问题：
实际计算中，会遇到数值稳定性（Numerical Stability）的问题，因为$e^{f_{k}}$和$\sum_{j}e^{f_{j}}$太大了。大数相除很容易导致计算结果误差很大。所以这里需要用到下面的技巧：
$\frac{e^{f_{k}}}{\sum_{j}e^{f_{j}}}=\frac{Ce^{f_{k}}}{C\sum_{j}e^{f_{_{j}}}}=\frac{e^{f_{k}+log^{C}}}{\sum_{j}e^{f_{j}+log^{C}}}$
实践中经常把C取为$log^{C}=-max_{j}f_{j}$。也就是说，再计算损失函数之前，要把输出向量里的每个值都减去该向量里的最大值
[Hierarchical softmax](https://arxiv.org/pdf/1310.4546.pdf)适用于标签集很大的情况
> HOG/SIFT 特征

