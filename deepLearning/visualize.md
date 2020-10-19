##### Retrieving images that maximally acticate a neuron
一种可视化技术是获取一个大的图像数据集，将数据喂给网络，并跟踪哪些图像能够最大程度地激活某些神经元。然后，我们可以将这些图像形象化，从而了解神经元在接受区寻找什么。
[Rich feature hierarchies for accurate object detection and semantic segmentation](https://arxiv.org/pdf/1311.2524.pdf)

##### Embedding the codes with t-SNE
ConvNets可以解释为将图像逐渐转换为一个表示形式，其中类可由线性分类器分离。我们可以通过将图像嵌入到二维空间中来大致了解这个空间的拓扑结构，这样它们的低维表示就比高维表示具有近似相等的距离。在保持点的两两距离的同时，将高维向量嵌入到低维空间中，这是一种直观的嵌入方法。在这些方法中，[t-SNE](http://lvdmaaten.github.io/tsne/)是最著名的方法之一，它始终能产生令人愉悦的视觉效果。

为了产生嵌入，我们可以取一组图像，使用ConvNet来提取CNN代码(例如，在AlexNet中分类器之前的4096维向量，关键是包括ReLU非线性)。然后我们可以把这些代入t-SNE得到每个图像的二维向量。相应的图像可以在网格中可视化[CNN代码的t-SNE可视化](https://cs.stanford.edu/people/karpathy/cnnembed/)

##### Occluding parts of the image
假设一个ConvNet将一个图像分类为一条狗。我们如何确定它实际上是在接收图像中的狗而不是背景或其他杂七杂八的物体的上下文线索呢?研究某些分类预测来自图像的哪个部分的一种方法是绘制感兴趣的类(例如dog类)作为遮挡器对象位置的函数的概率。也就是说，我们遍历图像的区域，将图像的patch设为0，然后观察类的概率。我们可以把这个概率想象成二维的热图。这种方法已经在Matthew Zeiler的[卷积网络可视化和理解](https://arxiv.org/pdf/1311.2901.pdf)中得到了应用

[笔记正文](https://arxiv.org/pdf/1311.2901.pdf)