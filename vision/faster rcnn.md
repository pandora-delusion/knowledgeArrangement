# Faster R-CNN

Fast-RCNN基本实现端对端（除了proposal阶段外），下一步自然就是要把proposal阶段也用CNN来实现（放到GPU上）。这就有了Faster R-CNN，一个完全端到端的CNN对象检测模型。论文提出，网络中的各个卷积层特征也可以用来预测类别相关的region proposal（不需要实现执行诸如selective search之类的算法），但是如果简单的在前面增加一个专门提取proposal的网络又显得不够优雅，所以最终把region proposal提取和Fast-RCNN部分融合进了一个网络模型 (区域生成网络 RPN层)，虽然训练阶段仍然要分多步，但是检测阶段非常方便快捷，准确率也与原来的Fast-RCNN相差不多，从此，再也不用担心region proposal提取耗时比实际对象检测还多这种尴尬场景了。

![](F:\mycode\knowledgeArrangement\vision\com.png)

参考论文：Faster R-CNN: Towards Real-Time Object Detection with Region Proposal Networks

​	RPN（a Region Proposal Network）与检测网络共享全图卷积特性，从而支持没有计算成本的建议区域。RPN是一个完全卷积的网络，它同时预测每个位置的对象边界和对象得分。对RPN进行端对端训练，生成高质量的建议区域，以供Fast R-CNN进行检测。进一步通过共享卷积特征整合RPN和Fast R-CNN到单个网络中——使用最近流行的神经网络术语“注意力”机制，RPN组建告诉网络在何处查找。



​	基于区域的检测器（如Fast RCNN）使用的卷积特征映射，也可以用来生成建议区域。在这些卷积特征的基础上，通过添加一些额外的卷积层来构造RPN，这些卷积层同时在一个规则网格上的每个位置上归回区域边界和目标打分。因此，RPN是一种全卷积网络，可以针对生成检测建议的任务进行端到端训练。

​	RPN被设计用来有效地预测具有广泛规模和高宽比的建议区域。

​	建议区域的性能成为整个检测器的性能瓶颈

Faster R-CNN结构图：

![](F:\mycode\knowledgeArrangement\vision\fastercnn.jpg)

### Faster R-CNN

Faster R-CNN由两个模块组成。第一个模块是建议区域的深度全卷积网络，第二个模块是使用建议区域的Fast R-CNN检测器。整个系统是一个单个的、统一的目标检测网络。

![](F:\mycode\knowledgeArrangement\vision\faster rcnn.png)

RPN模块使用最近流行的“注意力”机制的神经网络术语，告诉Fast R-CNN模块取哪里看。

#### 网络概述

首先，使用卷积网络（RESNET，VGGNET）对输入图像做特征提取，获得特征映射。将特征映射喂给RPN层，RPN层输入通过一个$3 \times 3$的卷积将输入映射到低维特征（代码中有输出深度512），在将这些特征送给两个并行的全连接层：背景-前景分类器和区域建议（ROI）边框回归器，获得一系列目标得分（背景和前景）和矩形目标建议区域（roi）。将获得的一系列roi送往roi池化层获得固定尺寸的对应roi区域的特征映射，将这些特征映射送往一系列卷积层，再通过一层全局平均池化层。最后，将特征通过两个并行的种类分类器和边框回归器，**损失函数采用多任务损失，将之前计算的背景-前景分类器、区域建议边框回归器、种类分类器和边框回归器以及正则化相加得到损失函数**（tf版本的实现和原论文不同，注意）。在训练时，在RPN阶段通过计算RPN特征不同位置的一系列anchors和ground truth边框的IoU，再应用非最大值抑制（NMS）获得rois的种类（前景-背景）和边框标签。

#### Region Proposal Networks

​	RPN接受任意大小的图像作为输入和输出一组矩形目标建议（object proposal），每个都有一个目标得分。我们的最终目标是与一个Faster R-CNN目标检测网络共享计算（共享输入特征），所以我们假设这两个网络共享一组公共的卷积层。为了生成区域建议，我们在卷积特征映射输出上通过最后一个共享卷积层滑动一个小网络。这个小网络是一个$n \times n$的卷积（代码中是$3 \times 3$、输出通道数为512），我们在输出上运行两个并行的卷积层：一个是一个卷积核为$1 \times 1$，深度为anchors数量的2倍的卷积（用来计算对应位置处不同的anchor的种类的得分），一个卷积核为$1 \times 1$，深度为anchors数量的4倍的卷积（用来预测对应位置处不同的anchor的边框）。在训练阶段，前者得到的输出被送到分类器中，后者被送到回归器中。训练RPN层是为了使RPN层来判断哪些区域可能是前景需要重点关注，哪些区域是背景需要辨别。通过RPN层挑选特定的区域，让网络注意特定的区域，再从特定区域获得特征映射送给网络的下层来计算，这就所谓的”注意力“机制。

​	RPN层训练阶段的标签是根据计算每个输入位置处所有anchors和gt边框的IoU来决定的，通过计算得到每个anchors的最大IoU和对应的gt边框，最大IoU低于0.5的被判定为背景，否则为前景。同时，选用IoU最大的gt和对应的anchor计算回归用的标签。

##### Anchors

​	在通过RPN层$n \times n$的卷积网络之后，我们同时在输出的每个位置（即特征映射上的每个滑动窗口）预测多个区域建议。anchor以滑动窗口为中心，并于尺度和长宽比相关联。

##### anchors的平移不变性

​	如果将一张图片上的目标移动，那么区域建议就会移动同样，函数就应该能够预测移动后的建议区域。这就是平移不变性。平移不变性也减小了模型的大小。

##### 多尺度anchor作为回归参考

​	用来解决多尺度和多长宽比的问题。本论文的方法参照多个尺度和长宽比的anchor对边界框进行分类和回归。在特征映射的同一个位置上，多个尺度的anchor共享特征的设计没有处理尺度的额外开支。

##### 损失函数

​	为了训练RPN，每个anchor都有两个类标签，我们给满足以下条件的anchor正标签：

1. 和一个gt边框有最高IoU的anchor（保证每个gt都有一个重叠率最高的anchor）
2. 和任何gt的IoU大于0.7的标签

注意单个gt边框可以给多个anchors分配正标签。我们给那些对所有gt的IoU都小于0.3的anchors负标签。除此之外的anchor对于训练目标没有贡献。跨越图像边界的anchor丢弃不用。

利用这些定义，遵循Fast R-CNN中的多任务损失，我们的损失函数定义如下：
$$
L(\left \{ p_{i} \right \}, \left \{ t_{i} \right \})= \frac{1}{N_{cls}} \sum_{i}L_{cls}(p_{i}, p_{i}^{*}) + \lambda \frac{1}{N_{reg}} \sum_{i}p_{i}^{*}L_{reg}(t_{i}, t_{i}^{*})
$$
$i$指代小批量的anchor的序号，$p_{i}$是anchor被预测为一个前景的概率，$p_{i}^{*}$是gt标签，1是前景，0是背景。$t_{i}$是代表预测边框的4个参数化坐标的向量。$t_{i}^{*}$是前景标签对应的gt边框。对于回归损失，我们使用：
$$
L_{reg}(t_{i}, t_{i}^{*})=R(t_{i}-t_{i}^{*})
$$
$R$为鲁棒的损失函数（smooth L1），和Fast R-CNN一样。

对于边框回归，采用以下四个坐标参数化：
$$
t_{x}=(x-x_{a})/w_{a}, t_{y}=(y-y_{a})/h_{a},\\
t_{w}=log(w/w_{a}), t_{h}=log(h/h_{a}), \\
t_{x}^{*}=(x^{*}-x_{a})/w_{a}, t_{y}^{*}=(y^{*}-y_{a})/h_{a},\\
t_{w}^{*}=log(w^{*}/w_{a}), t_{h}^{*}=log(h^{*}/h_{a}),
$$

这可以看作是从anchor到附近gt真值的边框回归。$t_{i}^{*}$表示gt边框相对于anchor边框的偏移，学习的目标就是学得后者到前者得映射。

在本文中用来回归的特征是特征映射中$3 \times 3$的空间范围内的特征。由于anchor的设计，即使特征的大小/比例是固定的，仍然可以预测各种大小的盒子。

##### 训练RPN

遵循Fast-RCNN的以图像为中心的采样策略来训练这个网络。每个小批次都来自一个包含许多正面和负面样本anchor的图像。如果每张图得所有anchor都去参与优化损失函数，那么最终会因为负样本过多导致最终得到得模型对正样本预测准确率很低。因此在一幅图像中随机采样256个anchor来计算一个小批量的损失函数，其中抽样的正anchor和负anchor的比列高达1：1。若正样本少于128个，就用负样本填充。

注意：在到达全连接层之前，卷积层和池化层对图片输入大小其实没有size得限制，因此RCNN系列得模型其实是不需要将图片resize到固定大小得。n=3看起来很小，但是要考虑到这是非常高层的feature map，其size本身也没有多大，因此 3×3×3 9个矩形中，每个矩形窗框都是可以感知到很大范围的。

#### RPN和Fast R-CNN共享特征

下面描述了学习由RPN和Fast RCNN组成的统一网络的算法，共享卷积层的R-CNN。

我们知道，如果是分别训练两种不同任务得网络模型，即使它们的结构、参数完全一致，但各自的卷积层内的卷积核也会想着不同的方向改变，导致无法共享网络权重，于是论文中描述了三种共享特征的训练方案：

1. 交替训练。此方法其实就是一个不断迭代的训练过程，既然分别训练RPN和Fast-RCNN可能让网络朝不同的方向收敛，a)那么我们可以先独立训练RPN，然后用这个RPN的网络权重对Fast-RCNN网络进行初始化并且用之前RPN输出proposal作为此时Fast-RCNN的输入训练Fast R-CNN。b) 用Fast R-CNN的网络参数去初始化RPN。之后不断迭代这个过程，即循环训练RPN、Fast-RCNN
2. 近似联合训练。在训练过程中，将RPN网络和Fast R-CNN网络合并为一个网络。在每个SGD迭代中，将前向传播生成的区域建议直接应用在训练Fast R-CNN检测器中。反向传播时，共享层将RPN损失和Fast R-CNN损失的反向传播信号结合起来。这很容易实现，但是忽略了对应建议边框坐标的导数（在代码的实现中是在反向传播中忽略了在roi池化层阶段对rois坐标的求导，不清楚这样做的原因。。。。），因为它也是网络的响应，所以时近似的。相比第一种方案训练事件缩短了25%到30%
3. 非近似联合训练。如上所述，RPN预测的边界框也是输入的函数。Fast R-CNN中的RoI池化层接受卷积特征和预测的边框作为输入，因此理论上有效的反向传播求解器应该包含边框坐标的梯度。上述近似联合训练忽略了这些梯度。在非近似联合训练解决方案中，我们需要一个对于边框坐标是可微分的roi池化层。

4步交替训练：原论文采用的方法。

1. 训练RPN，使用ImageNet预训练模型初始化网络并为region proposal任务做端到端微调。
2. 使用由第一步RPN生成的proposals训练独立的Fast R-CNN检测网络（检测网络当然用ImageNet初始化）。该网络也由imageNet预训练模型初始化。此时，两个网络并不共享卷积（的参数）。
3. 使用检测网络来初始化RPN网络，但是固定共享的卷积层（把共享的卷积层的学习速率设置为0，也就是不更新），仅仅调优RPN特有的层（那个小的滑动卷积层、分类器和回归器），重新训练。现在，两个网络就共享卷积层了。
4. 最后，保持共享卷积层不变，我们调优Faster R-CNN独有的层。

这样，两个网路共享相同的卷积层，形成了统一的网络。

#### 实现细节

在单一尺度的图像上训练区域建议和目标检测。

要小心处理跨越图像边界的anchor，在训练中忽略了所有的跨边界anchor以免造成损失。如果训练中不忽略跨界离群点，则会在目标中引入大量难以纠正的误差项，使得训练不会收敛。对高度重叠的RPN proposals，我们根据region proposals的分类器评分对propoals采用非最大抑制。NMS不会损害最终的检测精度，但会大大减少proposals的数量。在NMS之后，使用排名前N的proposal regions进行检测。

4步交替训练中，如果在第二步就停止，使用分离的网络会稍微降低结果。实验观察到当第三步的被检测器调优的特征被用来调优RPN时，proposals的质量得到了提高。

