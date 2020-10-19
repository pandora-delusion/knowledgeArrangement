#  R-CNN

### R-CNN的目标检测

目标检测系统包含三个模块：

+ 第一部分生成无关分类的区域建议（region proposals）
+ 第二部分是提取来自每个区域的特征向量的卷积网络
+ 第三部分是SVM线性分类器

区域检测：定义了我们的检测器可用的候选检测集。

### 模型设计

#### 区域建议

生成无关分类的区域建议方法有：objectness，selective search，category-independent object proposals，CPMC，

multi-scale combinatorial grouping等。R-CNN独立于这些区域建议方法。

#### 特征提取

使用CNN从每个区域建议中提取固定长度特征向量。为了计算区域建议的特征，首先将该区域的图像数据转换成与CNN兼容的形式。无论候选区域的大小或者长宽比如何，我们都将它周围的所有像素弯曲到所需的大小。

训练具体流程：输入图片，提取区域建议（应用选择性搜索等方法），扭曲每个区域建议到固定大小。将所有扭曲后的区域建议通过卷积网络提取固定大小的特征向量，最后使用SVM线性分类器来分类区域

### 测试检测

测试流程：提取区域建议，扭曲到固定大小，通过卷积网络提取特征向量，然后使用训练好的SVM计算每个类的得分。给定一章图片的所有得分区域，为每个类独立地应用贪婪的非最大抑制（a greedy non-maximum suppression），如果一个区域和一个更高得分的选择区域的IOU重叠高于学习阈值，就拒绝这个区域。

#### 运行时分析

检测效率高的原因：

+ CNN的参数共享
+ CNN计算的特征向量是低维的

主要时间用在计算区域建议和特区特征上面

R-CNN可以扩展到数千个对象类，而无需使用哈希等逼近技术。

### 训练

#### 有监督的预训练

略略略

#### 特定的微调

除了将CNN imageNet的分类层替换为随机初始化的N+1种分类层（N是目标种类的数量，+1指代背景），CNN的架构不会改变。We treat all region proposals with $\geq $0.5 IoU overlap with a ground-truth box as positives for that box's class and the rest as negatives. 在每次SGD迭代中，统一采样32个正面窗口（所有类）和96个负面窗口来构造一个128大小的小批量。我们倾向于正窗口的采样，因为与背景相比，它们非常罕见。

#### 目标类别分类

仔细选择IoU阈值很重要

在提取特征并应用训练标签后，我们对每个类优化一个线性SVM。

### 分析

#### 可视化学习特征

第一层过滤器可以直接可视化，易于理解。它们捕捉方向的边缘和颜色。理解后面的层具有挑战性。

提出了一个非参数方法，直接显示网络所学。其思想是在网络中挑出一个特定的单元（特征），并把它作为一个单独的对象检测器来使用。也就说，我们计算大量区域建议在该单元上的激活值，将区域按激活值从高到低排序，执行非最大值抑制，然后显示得分地区。

仅使用CNN的卷积层，就可以计算任意大小图像的密集特征映射而言，具有潜在的实用价值。

#### 网络架构

架构的选择对r-cnn检测性能有很大的影响。

#### 边界框回归

在误差分析的基础上，实现了一种减少定位误差的简单方法。受DPM中使用边界盒回归的启发，我们训练了一个线性回归模型来预测一个新的检测窗口，给定提取自选择性搜索区域建议（region proposal）的$pool_{5}$的特征。从来自验证集的随机选择的5000张图片进行*hard negative mining*。

#### 训练数据

R-CNN训练数据需要三个步骤：

1. CNN fine-tuning
2. 检测SVM训练
3. 边框回归训练

首先在验证集和训练集上做CNN微调。在SVM训练阶段，所有来自验证集和训练集的真实边框被用作其对应分类的正样本。在验证集中随机选择5000张图片的子集来进行hard negative mining。没有从训练集中提取负样本，因为训练集负样本不全。在验证集上进行边界框训练。

### 实现和设计细节

#### 目标区域转换

两种将目标区域转换为CNN合法输入的手段：

+ 将每个目标区域包含在最紧的正方形中，然后将正方形中包含的图像按比例缩放到CNN输入大小。该方法的变体，排除了围绕原始目标区域的图像内容
+ 非均值地伸缩目标区域到CNN输入大小

对于每一个变换，考虑在原始目标区域周围包含额外的图像上下文。上下文填充量（p）定义为转换后的输入坐标系中围绕原始目标区域的边框大小。

在所有方法中，如果源矩形超出图像，则用图像均值替换缺失的数据（然后在将图像输入CNN之前减去图像均值）

*A pilot set of experiments showed that warping with context padding (p=16 pixels) outperformed the alternatived by a large margin (3-5 mAP points)*

还有更多方法有待发掘。。。

#### Positive vs. negative example and softmax

***为什么在微调CNN和训练目标检测支持向量机时，正面样本和负面样本的定义不同？***

简要回顾定义：为了进行微调，我们将每个目标区域映射到真实边框实例，当且仅当二者的IoU重叠值最大；如果IoU至少大于0.5，则将该目标区域标记为匹配的真实边框的类的正例。而所有类的负样本则被标记为背景。相反，在训练SVM时，我们只将真实边框作为它们各自的类的正样本；同时将于所有类实例的IoU只小于0.3的区域标记为对应类的负样本。那些落入灰色区域的区域将会被忽略（IoU超过0.3，但不是真实边框）。之所以这样是有历史原因的。。

***在进行了微调之后，为什么还要培训SVM呢？***

在微调后不适用SVM训练也可以获得接近相同水平的性能。

#### Bounding-box regression

在用分类检测SVM为每个选择性搜索区域打分之后，我们预测了一个新的边界框检测使用一个特定类的边界框回归器。$P$指代Region Proposal

训练算法的输入时N个训练对$\left \{ \left ( P^{i},G^{i} \right ) \right \}_{i=1,\cdots,N}$,$P^{i}=(P^{i}_{x},P^{i}_{y},P^{i}_{w},P^{i}_{h})$指定区域中兴的像素坐标以及$P^{i}$边框的宽和高的像素表示。真实边框G的指定方法相同$G=(G_{x},G_{y},G_{w},G_{h})$，上标舍去。我们的目标时学习这个转换，将预测区域边框P映射到真实边框G上。

我们用四个函数来参数化这个变换$d_{x}(P)$，$d_{y}(P)$，$d_{w}(P)$，$d_{h}(P)$。

前两个参数指定了P的边界框中心的尺度不变平移，而后面两个制定了P的边界框的宽度和高度在对数空间上的平移。

在学习了这些函数之后，我们就可以通过应用转换将输入区域P转换为预测的真实边框G：
$$
\widehat{G}_{x}=P_{w}d_{x}(P)+P_{x}\\
\widehat{G}_{y}=P_{h}d_{y}(P)+P_{y}\\
\widehat{G}_{w}=P_{w}exp(d_{w}(P))\\
\widehat{G}_{h}=P_{h}exp(d_{h}(P))
$$
每个函数$d_{*}(P)$(\*指代x，y，h，w)被建模为建议区域P的$pool_{5}$特征（指代为$\phi _{5}(P)$)的线性函数。$\phi _{5}(P)$对数据图像的依赖关系是隐式假设的。因此我们有$d_{*}(P)=W^{T}_{*} \phi _{5}(P)$，$W_{*}$是一个学习模型参数的向量。我们学习$W_{*}$通过优化正则化最小二乘目标（ridge regression）：
$$
W_{*}=argmin_{\widehat{W}_{*}} \sum_{i}^{N} (t^{i}_{*}-\widehat{W}_{*}^{T} \phi _{5}(P^{i}))^{2}+\lambda \left \| \widehat{W}_{*}  \right \|^{2}
$$
训练对（P，G）的回归目标$t_{*}$定义如下：
$$
t_{x}=(G_{x}-P_{x})/P_{w}\\
t_{y}=(G_{y}-P_{y})/P_{h}\\
t_{w}=log(G_{w}/P_{w})\\
t_{h}=log(G_{h}/P_{h})
$$
（至于为什么用这种方法见[Rich feature hierarchies for accurate object detection and semantic segmentation](<https://arxiv.org/pdf/1311.2524.pdf>)这篇论文）

作为一个标准的正则化最小二乘问题，该问题可以用闭形式有效地求解。

选择训练的（P，G）对十分重要，直观上说，如果P原理虽有的基本真值框，那么将P转换为基本真值框G的任务就没有意义了。所以，选择在至少一个真值框附近的区域P来学习（IoU大于某个阈值）。

在测试时，我们为每个区域打分并预测它的新检测窗口一次。

### 优缺点

优点：

+ 结构简单明了，容易理解
+ CNN自动提取特征，省去手工设计特征的复杂操作，以及对经验和运气的依赖性
+ 使用 selective search方法来生成候选区域，显著减少候选区域的数量，增加候选区域的质量（包含目标的可能性更大），因为这相当于一个弱检测器，相比于sliding window穷举搜索的方式肯定要好很多；
+ 预测精度提高了30%。

缺点：

+ 提取特征时，CNN需要在每一个候选区域上跑一遍；候选区域之间的交叠使得特征被重复提取, 造成了严重的
  速度瓶颈, 降低了计算效率
+ 将候选区域直接缩放到固定大小, 破坏了物体的长宽比, 可能导致物体的局部细节损失;
+ R-CNN还不是端到端的模型，训练步骤繁琐multi-stage（先预训练、fine tuning、存储CNN提取的特征；再训练SVM ；再regression）。从fine tuning 到训练SVM时，不能一步到位，要分成两步；
+ 训练SVM时需要将之前CNN提取到的特征全部存储在磁盘上，磁盘读写耗时，且占用空间大，（Pascal 200G）；
+ 使用额外的selective search 算法生成候选区域的过程也很耗时；
+ 预测时间很慢，一张图片要49s。

### 选择性搜索

#### 基于图的图像分割（Graph-Based Image Segmentation）

[论文](<http://cs.brown.edu/people/pfelzens/papers/seg-ijcv.pdf>)该算法是基于图（最小生成树）的贪心聚类算法，实现简单，速度比较快，精度还行。目前不直接用它做分割，是很多算法的基础(Segmentation as Selective Search for Object Recognition)。

该算法是将图像用加权图抽象化表示

##### 最小生成树（Minimum Spanning Tree）

给定需要连接的顶点，选择边权之间最小的树

论文中。初始化时每一个像素点都是一个顶点，然后逐渐合并到一个区域，确切地说是连接这个区域中的像素点的一个MST。

##### 相似性

作为聚类算法，如何判定何时两个区域合二为一，何时继续划定界限？

对于两个孤立的像素点，所不同的是灰度值，用灰度的距离来衡量两点的相似性，论文中使用RGB距离：
$$
d = \sqrt{(r_{1}-r_{2})^{2}+(g_{1}-g_{2})^{2}+(b_{1}-b_{2})^{2}}
$$

##### 全局阈值 >> 自适应阈值、区域的类内差异、类间差异

应该用亮度值来衡量两个像素点之间的差异性。对于两个区域（子图）或者一个区域和一个像素点的相似性，最简单的方法即考虑连接二者的边的不相似度。设定一个阈值，当两个像素之间的差异（不相似度）小于该值时，合二为一。

两个附加定义：

一个区域内的类内差异$Int(C)$：
$$
Int(C)=max_{e\in MST(C,E)}W(e)
$$
可以理解为一个区域内部最大的亮度差异值，定义是MST中不相似度最大的一条边

两个区域的类间差异$Diff(C1, C2)$:
$$
Dif(C_{1},C_{2})=max_{v_{i} \in C_{1},v_{j} \in C_{2}, (v_{i},v_{j}) \in E} W(v_{i}, v_{j})
$$
连接两个区域的所有边中，不相似度最小的边的不相似度，也就是像个区域最相似的地方的不相似度。

直观的判断，当：
$$
D(C_{1},C_{2})=\left\{\begin{matrix}
true & if Dif(C_{1},C_{2})> MInt(C_{1},C_{2})\\ 
false& otherwise 
\end{matrix}\right.
$$
最小内部差异$MInt(C_{1},C_{2})$定义为：
$$
MInt(C_{1},C_{2})=min(Int(C_{1})+\gamma(C_{1}),Int(C_{2})+\gamma(C_{2}))
$$
在极端情况下，当$\left | C \right |=1$时，$Int(C)=0$。使用一个基于区域大小的阈值函数，
$$
\gamma(C)=\frac{k}{|C|}
$$
也就是说，对于小的区域，我们需要更强的边界证据。

##### 算法步骤

分割算法：

输入为图$G=(V,E)$，有n个点和m个边。输出为V分割后的集合$S=(C_{1},\cdots,C_{\gamma})$。

0. 将边集E按照边权重非递减的顺序排序为$\pi=(o_{1},\cdots, o_{m})$。
1. 分割集合$S^{0}$，每个点$v_{i}$都是一个区域。
2. 从$q=1,\cdots,m$重复步骤3
3. 按以下规则用$S^{q-1}$重构$S^{q}$：令$v_{i}$和$v_{j}$为连接$\pi$中第q个边的端点$o_{q}=(v_{i},v_{j})$。若$v_{i}$和$v_{j}$属于$s^{q-1}$

中不相交的两个区域，并且$W(o_{q})$相对于这两个分量的内部差异较小，然后就合并这两个向量，否则什么也不做。更加正式地，令$C_{i}^{q-1}$为$S^{q-1}$中包含$v_{i}$的区域，令$C_{j}^{q-1}$为$S^{q-1}$中包含$v_{j}$的区域。如果$C_{i}^{q-1} \neq C_{j}^{q-1}$并且$W(o_{q}) \leq MInt(C_{i}^{q-1},C_{j}^{q-1})$，那么合并$C_{i}^{q-1}$和$C_{j}^{q-1}$，并令$S^{q}=S^{q-1}$;否则直接令$$S^{q}=S^{q-1}$$。

4. 返回$S=S^{m}$

#### selective search 算法

整理自：[selective search for object detection (C++/python)](<https://www.learnopencv.com/selective-search-for-object-detection-cpp-python/>)

![avarta](hierarchical_grouping_algorithm.png)

选择搜索不是规模不变的，所以产生的区域数量取决于图像分辨率。

##### 相似度计算

（以后用到在补充）

+ 颜色相似度
+ 纹理相似度
+ 尺寸相似度
+ 交叠相似度
+ 最总相似度

非极大值抑制（NMS）

用来抑制那些冗余的框，抑制的过程是一个迭代-遍历-消除的过程。

1. 将所有框的得分排序，选中最高分及其对应的框

2. 遍历其余的框，如果和当前最高分框的重叠面积(IoU)大于一定阈值，我们就将框删除。
3. 从未处理的框中继续选一个得分最高的，重复上述过程。

### mAP （Mean Average Precision）

衡量算法精确度好坏的基准

[目标检测评价指标](<https://blog.csdn.net/syoung9029/article/details/56276567>)

True positives：某类的图片被正确地识别为某类

True negatives: 不是某类的图片没有被识别出来，系统正确地认为它不是某类

False positives: 不是某类的图片被错误地识别为某类

False negatives：某类的图片没有被识别出来，系统错误地认为它们不是某类

*Precision与Recall*

precision是在识别出来的图片中，True positive所占的比率：
$$
precision=\frac{tp}{tp+fp}
$$
$tp+fp$是指系统一共识别出来多少照片

recall是指被正确识别出来的某类与测试集中所有某类的个数的比值
$$
recall=\frac{tp}{tp+fn}
$$
$tp+fn$是指测试集中某类的总数

综合考量召回率和精确率，计算这两个指标的调和平均数，得到$F_{1}$指标：
$$
F_{1} measure = \frac{2}{ \frac{1}{precision}+ \frac{1}{recall} }
$$
之所以使用调和平均数，是因为除了具备平均功能之外，还会对那些召回率和精确率更加接近的模型给予更高的分数；那些召回率和精确率差距过大的学习模型，往往没有足够的使用价值。

*precision-recall曲线*

如果你想评估一个分类器的性能，一个比较好的方法就是：观察当阈值发生变化时，precision与recall值的变化情况。如果一个分类器的性能比较好，那么他应该有如下的表现：让recall值增长的同时保持precision的值在一个很高的水平。而性能比较差的分类器可能会损失很多的precision值才能换来recall值得提高。通常情况下，文章都会使用precision-recall曲线，来显示分类器在precision与recall之间得权衡。

*Approximated Average precision* 
$$
\int_{0}^{1}p(r)dr
$$
p代表precision，r代表recall，p是一个以r为参数得函数，这等价于取曲线下得面积。

实际上转移积分及其接近这一数值：对 每一种阈值分别求Precision值乘以Recall值的变化情况，再把虽有阈值下球的的乘积值进行累加。公式如下：
$$
\sum_{k=1}^{N}P(k)\Delta r(k)
$$
在这一公式中，N代表测试集中所有图片的个数，P(k)表示在能识别出k个图片的时候precision的值，而$\Delta r(k)$则表示识别图片个数从k-1变化到k时（通过调整阈值）recall值的变化情况。

*Interpolated average precision*

每次使用在所有阈值的precision中，最大值的那个precision值与recall的变化值相乘。公式如下：
$$
\sum_{k=1}^{N}max_{\widehat{k} \geq k}P(\widehat{k})\Delta r(k)
$$

### hard negative mining

 当通过bounding box标记来训练分类器时，你的分类器要取做分类的话，你的分类器需要既包含正训练样本（识别目标）和负训练样本（背景）。hard mining的意义在于选出一些对训练网络有帮助的部分负样本，而不是所有负样本，来训练网络。在训练的过程中，被当前模型误判的那些负样本可以被挑出来，参与训练和反向传播。

R-CNN关于hard negative mining的部分引用了两篇论文，部分如下：

*Bootstrapping methods train a model with an initial subset of negative examples, and then collect negative examples that are incorrectly classified by this initial model to form a set of hard negatives. A new model is trained with the hard negative examples, and the process may be repeated a few times.*

*We use the following “bootstrap” strategy that incrementally selects only those “nonface” patterns with high utility value:
1) Start with a small set of “nonface” examples in the training database.
2) Train the MLP classifier with the current database of examples.
3) Run the face detector on a sequence of random images. Collect all the “nonface” patterns that  the current system wrongly classifies as “faces” (see Fig. 5b).Add these “nonface” patterns to the training database as new negative examples.
4) Return to Step 2*

在bootstrapping方法中，我们先用初始的正负样本（一般是正样本+与正样本同规模的负样本的一个子集）训练分类器，然后再用训练出的分类器对样本进行分类，把其中错误分类的那些样本（hard negative）放入负样本集合，再继续训练分类器，如此反复，直到达到停止条件（比如分类器性能不再提升）。

### Ablation study

消融研究：指通过移除某个模型或者算法的某些特征，来观察这些特征对模型效果的影响。

