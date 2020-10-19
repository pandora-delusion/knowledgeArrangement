# Focal loss和retinaNet

​	在two-stage目标检测模型（比如R-CNN）中，模型在定位候选目标的稀疏集合上应用分类器；而在one-stage目标检测模型中，分类器应用在常规的、密集的候选目标的采样中。这使得one-stage更加快和简单，但损失了精确度。

​	在one-stage模型的训练过程中所遇到的极端的前景-背景类不平衡是主要原因。

​	在R-CNN中类不平衡是通过两个stage级联和启发式采样解决的。proposal stage（selective search）快速地将候选对象位置的数量缩减到一个很小的数目，以过滤掉大部分背景样本。在分类阶段，采用启发式采样，比如固定前景-背景的比率为1：3或者online hard example mining（OHEM）来维持前景和背景样本之间的平衡。

​	相反，one-stage检测器必须要处理在图片中正常采样的更大的候选目标集合。虽然也可以使用类似的抽样启发法，但是效率很低，因为训练过程仍然被容易分类的背景样本主导。这种低效率是目标检测任务中的经典问题，通常用bootstrapping或者hard example mining来解决。

​	focal loss是一个动态缩放的交叉熵损失，当对正确类的置信度增加时，比例因子衰减为0。直观地，这个比例因子可以自动地降低训练过程中容易题学习的样本的权重，并快速地将模型的注意力几种在难以学习的样本上。focal loss使得检测器的性能优于使用启发式采样或者hard example mining的其他方案。 

### 类不平衡

所有one-stage模型在训练过程中都面临类不平衡问题。这些检测器每张图片要评估$10^{4}$到$10^{5}$个候选位置但是只有一小部分包含目标。这种不平衡导致两个问题：

1. 训练时低效的，因为绝大多数位置很容易被标记为负样本，这对于学习没有帮助。
2. 简单的负样本会主导训练并导致模型退化

常见的解决办法就是hard negative mining

focal loss旨在解决分类的不平衡问题，方法是降低简单例子的权重，使它们对总损失的贡献很小，即使它们的数量很大。它将训练集中在一组稀疏的hard example上。

### Focal loss

​	引入$\alpha \in [0,1]$来解决类不平衡问题，实际上$\alpha$可以被设置为类频率的倒数或者是一个可以交叉验证的超参数。容易分类的负样本（背景）构成了大部分的损失并主导梯度。$\alpha$平衡了正负样本的重要性，但是它不能区分简单/困难（容易学习和不容易学习）的样本。focal loss重塑损失函数，减少简单例子的权重，从而将任务训练重点放在学习困难的负面因素上。

​	focal loss给交叉熵损失函数添加$(1-p_{t})^{\gamma}$，$\gamma$大于等于0。我们定义focal loss 如下：
$$
FL(p_{t})=-\left (1-p_{t}  \right )^{\gamma}log\left ( p_{t} \right )
$$
![1567775428267](F:\mycode\knowledgeArrangement\vision\focalloss.png)

focal loss具有以下两个特性：

1. 当样本被分类错误并且$p_{t}$很小的时候，其权重接近1损失不受影响。当$p_{t}$接近1，权重接近0，这种很容易被分类的样本会被降权。
2. 参数$\gamma$平滑地调整了简单样本的降权速率

在实践中，focal loss如下：
$$
FL\left ( p_{t} \right )=-\alpha_{t} \left ( 1-p_{t} \right )^{\gamma}log \left ( p_{t} \right )
$$
在二分类问题下，focal loss变为：
$$
\left\{\begin{matrix}
-\alpha \left ( 1-p_{t} \right )^{\gamma}log \left ( p_{t} \right )\\ 
-\left ( 1-\alpha \right )p_{t}^{\gamma}log \left ( 1-p_{t} \right )
\end{matrix}\right.
$$
在实践中，损失层的实现将sigmoid操作与损失计算相结合，使得数值稳定性更好

focal loss 的精确形式并不重要，focal loss 的其他实例同样有效

```python
import keras
import tensorflow as tf

def focal(alpha=0.25, gamma=2.0):
    
    def _focal(y_true, y_pred):
        
        # [B, N, num_classes]
        labels = y_true[:, :, :-1]
        # -1 for ignore, 0 for background, 1 for object
        anchor_state = y_true[:, :, -1]
        classification = y_pred
        
        indices = tf.where(keras.backend.not_equal(anchor_state, -1))
        lables = tf.gather_nd(labels, indices)
        classification = tf.gather_nd(classification, indices)
        
        alpha_factor = keras.backend.ones_like(labels)*alpha
        alpha_factor = tf.where(keras.backend.equal(labels, 1), alpha_factor, 1-alpha_factor)
        focal_weight = tf.where(keras.backend.equal(labels, 1), 1-classification, classification)
        focal_weight = alpha_factor*focal_weight**gamma
        
        cls_loss = focal_weight*keras.backend.binary_crossentropy(labels, classification)
        
        normalizer = tf.where(keras.backend.equal(anchor_state, 1))
        normalizer = keras.backend.cast(keras.backend.shape(normalizer)[0], keras.backend.floatx())
        normalizer = keras.backend.maximum(keras.backend.cast_to_floatx(1.0), normalizer)
        
        return keras.backend.sum(cls_loss)/normalizer
  return _focal
```



### 类不平衡和模型初始化

在二分类问题中，默认初始化为等概率。在这样的初始化下，在类不平衡的情况下，频繁的类造成的损失会主导整个损失，导致早期训练的不稳定。为了解决这个问题，在训练开始时引入“先验”的概念，即稀有类（前景）的模型估计的p值。

这使得稀有类样本的评估值p小，这样它的focal loss权重就大。在种类严重不平衡的情况下，这种方法可以提高交叉熵和focal loss的训练稳定性。

two-stage检测器通常训练交叉熵损失，而不使用$\alpha-balance$或者focal loss。它们通过两种机制解决不平衡问题：

两阶段级联，first stage一般是object proposal机制，将近乎无限的可能目标位置降低到1到2千个。重要的是，这种选择不是随机的，而是可能与真实的对象位置相对应的，这消除了绝大多数容易产生的负面样本。在second stage时，bias samping通常用于构建包含正负样本比例为1：3的小batch。

### RetinaNet

​	主干（backbone）负责计算整个输入图像上的卷积特征图，是一个自定义的卷积网络。第一个子模型对骨干网络的输出进行卷积目标分类；第二个子模型执行卷积边框回归。

![1567822276717](F:\mycode\knowledgeArrangement\vision\retinanet.png)

#### Feature Pyramid Network Backbone

​	引入feature pyramid network作为retinaNet的骨干网络。FPN使用自顶向下的直通和侧面连接使网络有效地从单张图片输入构造一个丰富的、多尺度的特征金字塔（FPN的改进是将语义信息较强的深层特征图和分辨率较高的浅层特征图相结合，具体采用深层特征图上采样，浅层特征图用$1 \times 1$卷积，之后相加）。金字塔的每一层都可以用来探测不同尺度的物体。FPN从全卷积网络改进多尺度预测。使用p3到p7层来构建金字塔，$l$指代金字塔层数（$p_{l}$层分辨率比输入低$2^{l}$倍）。所有金字塔层的通道数都为256。虽然许多设计选择并不重要，但强调使用FPN骨干网络。

```python
import keras
import layers
def _create_pyramid_feature(c3, c4, c5, feature_size=256):
    
    p5 = keras.layers.Conv2D(feature_size, kernel_size=1, strides=1, padding="same", name="c5_reduced")(c5)
    # 上采样层，一般使用最邻近采样
    p5_upsampled = layers.UpSampleLike(name="p5_upsampled")([p5, c4])
    p5 = keras.layers.Conv2D(feature_size, kernel_size=3, strides=1, padding="same", name="p5")(p5)
    
    p4 = keras.layers.Conv2D(feature_size, kernel_size=1, strides=1, padding="same", name="c4_reduced")(c4)
    # 融合层，与yolov3的沿通道推叠两种尺度的特征图不同，注意!
    p4 = keras.layers.Add(name="p4_merged")([p5_upsampled, p4])
    p4_upsampled = layers.UpSampleLike(name="p4_upsampled")([p4, c3])
    p4 = keras.layers.Conv2D(feature_size, kernel_size=3, strides=1, padding="same", name="p4")(p4)
    
    p3 = keras.layers.Conv2D(feature_size, kernel_size=1, strides=1, padding="same", name="c3_reduced")(c3)
    p3 = keras.layers.Add(name="p3_merged")([p4_upsampled, p3])
    p3 = keras.layers.Conv2D(feature_size, kernel_size=3, strides=1, padding="same", name="p3")(p3)
    
    p6 = keras.layers.Conv2D(feature_size, kernel_size=3, strides=2, padding="same", name="p6")(c5)
    
    p7 = keras.layers.Activation("relu", name="c6_relu")(p6)
    p7 = keras.layers.Conv2D(feature_size, kernel_size=3, strides=2, padding="same", name="p7")(p7)
    
    return [p3, p4, p5, p6, p7]
```

​	从p3到p7层，anchors面积从$32^{2}$到$512^{2}$。在每个金字塔层使用三种长宽比$\left \{ 1:2,1:1,2:1 \right \}$。同时对与每个层的anchor添加三种尺寸$\left \{ 2^{0},2^{1/3},2^{2/3} \right \}$。这样每层每个位置会有9个anchors，这样所有层的anchor覆盖了输入图像从32像素到813像素。每个anchor都有一个K维分类向量目标（K为种类数）和一个4维回归目标

​	gt边框分配给与gt边框IoU大于0.5的anchors，IoU在$[0, 0.4)$的anchors被标记为背景。每个anchors最多被分配一个目标边框，IoU在$[0.4, 0.5)$的anchors被忽略。

#### 分类子模块

​	分类子模块预测每个anchor的K个目标分类的出现概率。这个子网络相当于一个FCN附在每个FPN上。*此子网的参数在所有金字塔层上共享*  

​	给定通道数为C的金字塔层特征图作为输入，子网络应用4个$3 \times 3$ 过滤器为C的卷积层，在每个卷积层后应用Relu激活层，后接一个$3 \times 3$的卷积层，过滤器数为$K \times A$（K是分类数，A是每个空间位置的anchors数）。最后应用sigmoid激活输出每个空间位置每个分类的概率。（代码中C=256，A=9）

​	**值得注意的是，分类子模块和回归子模块不共享参数。这些高层次的设计决策比超参数的设置更加重要。**

#### 边框回归子模块

​	与目标分类子网络并行，将另一个FCN附加到金字塔层以完成回归任务。边框回归自网络的设计与分类自网络相同，只是每个空间位置的线性输出为4A。使用和R-CNN完全相同的回归策略。**注意，这里使用了一个与分类无关的边框回归器**，这样参数比较少也很有效。目标分类自网络和边框回归自网络虽然结构相同，但使用不同的参数。

​	在分类子网络中使用focal loss。总focal loss为超过10万个anchors的focal loss的和，采用除以前景anchors的数量的方式来归一化。因为绝大多数anchors都是明显的负样本并且在focal loss下这些负样本的损失值几乎忽略不计，所以直接除以前景anchors的个数。注意分配给稀有类的$\alpha$必须与$\gamma$相互作用，一般情况下$\gamma=2, \alpha=0.25$最好。

#### 模型初始化

​	分类子网络下，偏置初始化为$b=-log((1-\pi)/\pi)$，$\pi$指代训练开始时每个anchor被标记为前景的置信度。在这里使用$\pi=0.01$。这种初始化可以防止大量的背景anchors在训练的第一次迭代中产生较大的、不稳定的损失值（实际上就是人为的使稀有类的权重变大）

focal loss可以有效地降低easy negatives的影响，将所有注意力集中在hard negatives的样本上。online hard example mining更强调分类错误的样本，和focal loss一样，但是ohem完全抛弃了简单的例子。

one-stage检测器的一个重要设计原理就是尽可能的用密集的边框覆盖图片的目标。two-stage检测器可以使用一个region pooling operation来对任何位置、长宽比、尺度的anchor进行分类。与之相比，one-stage使用固定的采样网络，这种方法实现边框高覆盖率的流行方法就是在每个空间位置使用多个anchors来覆盖不同尺度和长宽比的边框。

### Focal loss的其他形式

​	focal loss的确切形式并不重要。

​	对交叉熵、focal loss求导：

​	
$$
\frac{\partial CE}{\partial x}=y\left ( p_{t}-1 \right )\\
\frac{\partial FL}{\partial x}=y\left( 1-p_{t} \right)^{\gamma} \left( \gamma p_{t} + p_{t}-1\right)\\
\frac{\partial FL^{*}}{\partial x}=y \left( p_{t}^{*}-1 \right)
$$
​	对于所有的损失函数，在置信度较高的预测，导数趋于0或者-1。但和CE不同的是，FL在$x_{t} > 0$时导数很小