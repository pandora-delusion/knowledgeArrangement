# SSD：Single Shot MultiBox Detector

Yolo虽然能达到实时的效果，但是由于只做了一次边框回归并打分，这类方法导致了小目标训练的非常不充分，对于小目标的检测效果非常的差。简而言之，YOLO系列对于目标的尺度比较敏感，而且对于尺度变化较大的物体泛化能力比较差。

再此基础上，SSD结合了Yolo和R-CNN的各自优势，使用一个小的卷积过滤器（3x3）来预测目标分类和边框位置的偏移量，使用不同的过滤器来检测不同纵横比的边框；将这些过滤器应用于网络后期的多个特征图上，以便在多个尺度上进行检测。

SSD要比yolo更快，更加精确（实际上与使用region proposals的two-stage方法一样准确）

### The Single Shot Detector（SSD）

#### 模型

SSD方法基于一个前向卷积网络，该网络生成一个固定大小的边界框集合和这些框中对象类实例存在的得分，然后执行一个非最大抑制步骤来生成最终检测（NMS）。

在网络中添加辅助结构，以产生以下关键特征的检测：

1. **多尺度特征图检测** 在截断后的基本网络的末端加入卷积特征层。这些层的大小逐渐减小，可以在多个尺度上预测探测结果。
2. **用于检测的卷积预测器** 每个增加的（或者从基本网络中选择的现成的特征层）都可以使用一组卷积滤波器生成一组固定的预测集合，如下图所示。一个大小为$m \times n$有p个通道的特征层，进行预测的基本组件是一个$3 \times 3 \times p $用来产生分类得分和相对于默认边框坐标的形状偏移量。在每个$m \times n$位置上都会产生输出值。输出值是网络预测的边界框相对于每个特征图位置对应的默认框的偏移量。（和faster r-cnn一样）
3. **默认边框和长宽比** 将每个特征图单元和一组默认的边界框关联起来，应用在网络顶部的多个特征图上。具体说，在特征图的每个位置上的每个边框（k个默认边框）上要计算c个类得分和4个相对于原始默认边框形状的偏置，结果一共有$(c+4)k$个过滤器应用在特征图的每个位置上，在一个特征图上会产生$(c+4)kmn$个输出。

![](F:\mycode\knowledgeArrangement\vision\ssd.png)

#### 训练

训练SSD和训练典型的使用region proposals的检测器的关键不同之处在于，ground truth信息需要被分配给检测器输出集合中的特定输出。一旦确定了分配，就可以端到端地应用损失函数和反向传播。训练还涉及检测默认边框的选择和尺度以及hard negative mining和数据增强策略。

###### 匹配策略

对于每个真值边框，我们从不同的位置，长宽比和尺度的默认边框中选择。开始将每个真值边框和jaccard overlap最大的默认边框匹配，选择那些与真值边框的jaccard overlap大于0.5的默认边框匹配。这简化了学习问题，允许网络预测多个重叠边框的高分，而不是要求它只选择重叠率最大的那个。

##### 训练目标

令$x_{ij}^{p}=\left \{ 1,0 \right \}$指代是否将第i个默认边框匹配给种类为p的第j个真值边框，有$\sum_{i}x^{p}_{ij}\geq $1。总体目标损失函数是定位损失（loc）和置信度损失（conf）的权重和：
$$
L(x,c,l,g)=\frac{1}{N}(L_{conf}(x,c)+\alpha L_{loc}(x,l,g))
$$
N为匹配的默认变边框的数量的数量。定位损失是预测边框（l）和真值边框（g）的Smooth L1损失。和Faster R-CNN类似，回归默认边框（d）的中心点$(cx,cy)$和高度（h）宽度（w）的偏置：
$$
L_{loc}(x,l,g)=\sum_{i\in Pos}^{N}\sum_{m\in \left \{ cx,cy,w,h \right \}}x_{ij}^{k}snooth_{L1}(l_{i}^{m}-\widehat{g}_{j}^{m})\\
\widehat{g}_{j}^{cx}=(g_{j}^{cx}-d_{i}^{cx})/d_{i}^{w}\\
\widehat{g}_{j}^{cy}=(g_{j}^{cy}-d_{i}^{cy})/d_{i}^{h}\\ 
$$

$$
\widehat{g}_{j}^{w}=log(\frac{g_{j}^{w}}{d_{i}^{w}})\\
\widehat{g}_{j}^{h}=log(\frac{g_{j}^{h}}{d_{i}^{h}})
$$

置信度损失：
$$
L_{conf}(x,c)=-\sum_{i \in Pos}^{N}x^{P}_{ij}log(\widehat{c}_{i}^{P})-\sum_{i \in Neg}log(\widehat{c}_{i}^{P}) \\
where \widehat{c}_{i}^{P}=\frac{exp(c_{i}^{P})}{\sum_{p}exp(c_{i}^{P})}
$$
通过交叉验证将权重项设置为1

##### 为默认边框选择尺寸和长宽比

来自底层的特征图可以提高语义分割的质量，因为底层可以获得输入对象的更精细的细节。网络中来自不同层次的特征图具有不同的感受域大小。SSD中特征图中每个点默认生成6个边框。通过多个特征图所有位置具有不同尺度和长宽比的默认边框，我们得到了一组不同的覆盖不同输入对象大小和形状的预测。

为了处理不同尺度的目标，SSD采用了不同层次的特征图来模拟学习。所以每个层次的默认边框需要不同的尺度和比例

尺度：

​	假设有m个不同层的特征图来做预测，最底层的Scale值为$S_{min}=0.2$，最高层为$S_{max}=0.9$。其他层通过下面的计算公式进行计算：
$$
S_{k}=S_{min}+\frac{S_{max}-S_{min}}{m-1}(k-1)，k \in [1,m]
$$
比率：

​	使用不同ratio值$\alpha_{r} \in \left \{ 1,2,\frac{1}{2},3,\frac{1}{3} \right \}$计算默认边框的高度和宽度。$w_{k}^{\alpha}=s_{k}\sqrt{\alpha_{r}}$，$h_{k}^{\alpha}=\frac{s_{k}}{\sqrt{\alpha_{r}}}$，对于ratio=1的情况，scale为$s_{k}^{'}=\sqrt{s_{k}s_{k+1}}$，所以总共有6种不同的默认边框。

##### Hard negative mining

不适用所有的负面样本，而是将每个默认边框的最大置信损失进行排序，并选择最高的那个这样使得负样本和正样本之间的比例最多为3：1。这将导致更快的优化和更稳定的训练。

##### 数据增强

为了使模型对于各种输入对象的大小和形状更具鲁棒性，每个训练图像都由以下任意一个选项随机采样：

+ 使用整个原始输入图像
+ 采样一个补丁，该补丁与目标的最小jaccard重叠率为0.1，0.3，0.5，0.7或者0.9
+ 随机采样一个补丁

每个采样补丁的大小是原始图片大小的[0.1, 1]，之间的长宽比为0.5和2。如果gt边框的中心位于采样的补丁中，那么我们将保留gt边框的重叠部分。*After the aforementioned sampling step, each sampled patch*
*is resized to fixed size and is horizontally flipped with probability of 0.5, in addition to*
*applying some photo-metric distortions similar to those described in [[14]](https://arxiv.org/ftp/arxiv/papers/1312/1312.5402.pdf).*

### 实验结果

论文的实验是基于VGG16，现在ILSVRC CLS-LOC数据集上进行预训练。

![](F:\mycode\knowledgeArrangement\vision\vgg16.png)

将fc6和fc7层转换为卷积层，从fc6到fc7层降采样参数，将$2 \times 2$-stride 2的池化层pool5转换为$3 \times 3$-stride 1，并使用膨胀卷积来填充小的空洞。移除掉所有dropout层和fc8层。

SSD-VGG16-300结构如下：

```python
import tensorflow as tf
import tensorflow.contrib.slim as slim

@add_arg_scope
def pad2d(inputs, pad=(0, 0), mode="CONSTANT", 
          data_format="NHWC", trainable=True, scope=None):
    with tf.name_scope(scope, 'pad2d', [inputs]):
        if data_format == "NHWC":
            paddings = [[0, 0], [pad[0], pad[0]], [pad[1], pad[1]], [0, 0]]
     	elif data_format == "NCHW":
            paddings = [[0, 0], [0, 0], [pad[0], pad[0]], [pad[1], pad[1]]]
        net = tf.pad(inputs, paddings, mode=mode)
        return net
    
def ssd_multibox_layer(inputs, num_classes, 
                      sizes, ratio=[1], normalization=1, bn_normalization=False):
    net = inputs
    if normalization > 0:
        net = l2_normalization(net, scaling=True)
    num_anchors = len(sizes) + len(ratios)
    
    # 预测每个位置的所有尺寸的anchor的形状
    num_loc_pred = num_anchors * 4
    loc_pred = slim.conv2d(net, num_loc_pred, [3, 3], activation_fn=None, scope="conv_loc")
    
    # 分类预测
    num_cls_pred = num_anchors*num_classes
    cls_pred = slim.conv2d(net, num_cls_pred, [3, 3], activation_fn=None, scope="conv_cls")
    cls_pred = tf.reshape(cls_pred, tensor_shape(cls_pred, 4)[:-1]+[num_anchors, num_classes])
    return cls_pred, loc_pred

end_points = {}
with tf.variable_scope(scope, 'ssd_300_vgg', [inputs], reuse=reuse):
    net = slim.repeat(inputs, 2, slim.conv2d, 64, [3, 3], scope="conv1")
    net = slim.max_pool2d(net, [2, 2], scope="pool1")
    net = slim.repeat(net, 2, slim.conv2d, 128, [3, 3], scope="conv2")
    net = slim.max_pool2d(net, [2, 2], scope="pool2")
    net = slim.repeat(net, 3, slim.conv2d, 256, [3, 3], scope="conv3")
    net = slim.max_pool2d(net, [2, 2], scope="pool3")
    net = slim.repeat(net, 3, slim.conv2d, 512, [3, 3], scope="conv4")
    end_points["block4"] = net
    net = slim.max_pool2d(net, [2, 2], scope="pool4")
    net = slim.repeat(net, 3, slim.conv2d, 512, [3, 3], scope="conv5")
    net = slim.max_pool2d(net, [3, 3], stride=1, scope="pool5")
    
    # 额外的SSD模块
    # 膨胀卷积
    net = slim.conv2d(net, 1024, [3, 3], rate=6, scope="conv6")
    net = tf.layers.dropout(net, rate=dropout_keep_prob, training=is_training)
    net = slim.conv2d(net, 1024, [1, 1], scope="conv7")
    end_points["block7"] = net
    net = tf.layers.dropout(net, rate=dropout_keep_prob, training=is_training)
    
    end_point = "block8"
    with tf.variable_scope(end_point):
        net = slim.conv2d(net, 256, [1, 1], scope="conv1x1")
        net = pad2d(net, pad=(1, 1))
        net = slim.conv2d(net, 512, [3, 3], stride=2, scope="conv3x3", padding="VALID")
    end_points[end_point] = net
    
    end_point = "blcok9"
    with tf.variable_scope(end_point):
        net = slim.conv2d(net, 128, [1, 1], scope="conv1x1")
        net = pad2d(net, pad=(1, 1))
        net = slim.conv2d(net, 256, [3, 3], stride=2, scope="conv3x3", padding="VALID")
    end_points[end_point] = net
    
    end_point = "block10"
    with tf.variable_scope(end_point):
        net = slim.conv2d(net, 128, [1, 1], scope="conv1x1")
        net = slim.conv2d(net, 256, [3, 3], scope="conv3x3", padding="VALID")
    end_points[end_point] = net
    
    end_point = "block11"
    with tf.variable_scope(end_point):
        net = slim.conv2d(net, 128, [1, 1], scope="conv1x1")
        net = slim.conv2d(net, 256, [3, 3], scope="conv3x3", padding="VALID")
    end_points[end_point] = net
    
    predictions = []
    for i, layer in enumerate(feat_layers):
        with tf.variable_scope(layer+"_box"):
            # 构造该特征层对应的每个位置的每个anchor的位置和分类
            p, l = ssd_multibox_layer(end_points[layer],
                                     num_classes,
                                     anchor_sizes[i],
                                     anchor_ratios[i],
                                     normalizations[i])
        predictions.append(prediction_fn(p))
        logits.append(p)
        localisations.append(l)
```

相比于其他特征层，block4有不同的特征尺度，这里引入L2 normalization来拉伸特征图每个位置的特征范数到20，并且从反向传播中学习这个拉伸尺度。（具体原因见[ParseNet](https://arxiv.org/pdf/1506.04579.pdf)这里不深究)

*相较于R-CNN，SSD有更少的定位误差，这表明SSD能够更好地定位对象因为它可以直接学习对象形状的回归和对象类别的分类，而不是使用两个解耦的步骤。然而，SSD与类似的对象类别（尤其是动物）有更多的混淆，部分原因是我们为多个类别共享位置。SSD在较小对象上的性能比在较大对象上差得多，这并不奇怪，因为这些小对象甚至可能在最顶层都没有任何信息。增加输入大小（比如从300x300到512x512）可以帮助改进对小目标的检测，但仍然有很大的改进空间。SSD在大型对象上执行得非常好。它对不同对象得尺寸比非常鲁棒，因为我们在每个特征图位置上使用不同长宽比的默认框。*

#### 模型分析

**数据增强非常重要**

**默认边框越多越好** 使用各种默认的边框形状似乎使预测边框的任务变得更加容易

**膨胀卷积更快**如果使用完整的VGG16，保持pool5原来的$2 \times2$-s2不变并且从fc6到fc7不使用降采样，添加conv5_3来预测，结果相同但速度要慢20%

**不同分辨率的多个输出层效果更好**

#### 用来小目标精确度的数据增强

没有像Faster R-CNN那样的后续特征重采样步骤，对于SSD来说小目标的分类任务是比较困难的。该策略生成的随机图像是一个放大操作，可以生成许多更大的训练示例。为了实现创建更多训练示例的缩小操作，我们首先在画布上随机放置一张图像，画布的大小为原始图像的16倍，其中填充了平均值，然后再执行任何随机裁剪操作。因为通过引入这种新的拓展数据增强技术，我们有了更多的训练图像，所以我们必须将训练迭代加倍。

