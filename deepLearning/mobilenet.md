# MobileNet

MobileNet v1中使用的Depthwise Separable Convolution是模型压缩的一个最经典的策略，它是通过将跨通道的$3 \times 3$卷积换成单通道的$3 \times 3$卷积+跨通道的$1 \times 1$卷积来达到此目的的。

MobileNet v2实在v1的Depthwise Separable Convolution的基础上引入了残差结构。并发现了ReLU在通道较少的特征图上有非常严重的信息损失问题，由此引入Linear Bottlenecks和Inverted Residual

### MobileNet v1

传统的卷积网络是跨通道的，对于一个通道数为M的输入特征图，要的到通道数为N的输出特征图。对于一个尺寸为$D_{k} \times D_{k}$的传统卷积的参数个数为$D_{k} \times D_{k} \times M \times N$，它一层的计算代价为：
$$
D_{k} \times D_{k} \times M \times N \times D_{W} \times D_{H}
$$
其中$(D_{W}, D_{H})$为特征图的尺寸。

v1中的Depthwise Seperable Convolution就是解决了传统卷积的参数数量和计算代价过于高昂的问题。Depthwise Seperable Convolution分成Depthwise Convolution和Pointwise Convolution

#### Depthwise卷积

Depthwise卷积是指不跨通道的卷积，也就是说特征图的每个通道都有一个独立的卷积核，并且卷积核作用且仅作用在这个通道上。Depthwise卷积的参数数量为$D_{k} \times D_{k} \times M$。它的计算代价也是传统卷积的$\frac{1}{N}$为：
$$
D_{k} \times D_{k} \times M \times D_{W} \times D_{H}
$$

#### Pointwise卷积

Depthwise卷积的操作虽然高效，但是它仅相当于对当前的特征图的一个通道施加了一个过滤器，并不会合并若干个特征从而生成新的特征，而且由于在Depthwise卷积中输出通道数等于输入通道数，因此它并没有升维和降维的功能。

为了解决这个问题，v1中引入了Pointwise卷积用于特征合并以及升维和降维。使用$1 \times 1$卷积来完成这个功能。Pointwise的参数数量为$M \times N$，计算量为：
$$
M \times N \times D_{W} \times D_{H}
$$

#### Depthwise Separable卷积

合并Depthwise卷积和Pointwise卷积便是Depthwise Separable卷积。它的一组操作（一次Depthwise卷积加一次Pointwise卷积）的参数数量为$D_{k} \times D_{K} \times M + M \times N$是普通卷积的
$$
\frac{D_{k} \times D_{k} \times M + M \times N}{D_{k} \times D_{k} \times M \times N}=\frac{1}{N}+\frac{1}{D_{k}^{2}}
$$
计算量和普通卷积的比值为：
$$
\frac{1}{N}+\frac{1}{D_{k}^{2}}
$$
对于一个$3 \times 3$的卷积而言，v1的参数数量和计算代价均为普通卷积的$\frac{1}{8}$左右。

```python
def simple_mobileNetv1(input_shape, k):
    inputs = Input(shape=input_shape)
    x = Conv2D(filters=32, kernel_size=(3, 3), strides=(2, 2), padding="same", activation="relu", name="m_conv_1")(inputs)
    x = DepthwiseConv2D(kernel_size=(3, 3), padding="same", activation="relu", name="m_dc_2")(x)
    x = Conv2D(filters=64, kernel_size=(1, 1), padding="same", activation="relu", name="m_pc_2")(x)
    x = DepthwiseConv2D(kernel_size=(3, 3), padding="same", activation="relu", name="m_dc_3")(x)
    x = Conv2D(filters=128, kernel_size=(1, 1), padding="same", activation="relu", name="m_pc_3")(x)
    x = DepthwiseConv2D(kernel_size=(3, 3), strides=(2, 2), padding="same", activation="relu", name="m_dc_4")(x)
    x = Conv2D(filters=128, kernel_size=(1, 1), padding="same", activation="relu", name="m_pc_4")(x)
    x = GlobalAveragePooling2D(name="m_gap")(x)
    x = BatchNormalization(name="m_bn_1")(x)
    x = Dense(128, activation="relu", name="m_fc1")(x)
    x = BatchNormalization(name="m_bn_2")(x)
    x = Dense(k, activation="softmax", name="m_output")(x)
    model = Model(inputs, x)
    return model
```



### MobileNetv2

在v2中，作者将v1中加入了残差网络。从流行的角度对v1的缺点进行了分析，认为是ReLU带来了信息损耗，当通道数非常少的时候更加明显，所以有两种解决方案：

1. 既然是ReLU导致的信息损耗，那么将ReLU替换成线性激活函数
2. 如果比较多的通道数能减少信息损耗，那么就使用更多的通道

#### Liear Bottlenecks

当然不能将ReLU全部换成线性激活函数，不然网络将退化成单层神经网络，一个这种方案是在输出特征图通道数较少的时候也就是瓶颈部分使用激活函数，其他时候使用ReLU。代码片段如下：

```python
import numpy as np

def relu6(x):
    return np.minimum(np.maximum(0, x), 6)

def _bottleneck(inputs, nb_filters, t):
    x = Conv2D(filters=bn_filters*t, kernel_size=(1, 1), padding="same")(inputs)
    x = Activation(relu6)(x)
    x = DepthwiseConv2D(kernel_size=(3, 3), padding="same")(x)
    x = Activation(relu6)(x)
    x = Conv2D(filters=nb_filters, kernel_size=(1, 1), padding="same")(x)
    # 不使用激活函数
    if not K.get_variable_shape(inputs)[3] == nb_filters:
        inputs = Conv2D(filters=nb_filters, kernel_size=(1, 1), padding="same")(inputs)
    outputs = add([x, inputs])
    return outputs
```

这里使用了MobileNet中介绍的ReLU6激活函数，它是对ReLU在6上的截断

#### Inverted Residual

当激活函数使用ReLU时，可以通过增加通道数来减小信息的损耗，使用参数t来控制，该层的通道数数输入特征图的t倍。传统的残差块的t一般取小于1的小数，常见的取值为0.1，而在v2中这个值一般是介于5-10之间的数，在作者的是验证中，t=6。Inverted Residual block把short-cut转移到了bottleneck层。

#### MobileNet v2

MobileNet v2的实现可以通过堆叠bottleneck的形式实现，如下：

```python
def mobileNetv2_relu(input_shape, k):
    inputs = Input(shape=input_shape)
    x = Conv2D(filters=32, kernel_size=(3, 3), padding="same")(inputs)
    x = _bottleneck(x, 8, 6)
    x = MaxPooling2D((2, 2))(x)
    x = _bottleneck_relu(x, 16, 6)
    x = _bottleneck_relu(x, 16, 6)
    x = MaxPooling2D((2, 2))(x)
    x = _bottleneck_relu(x, 32, 6)
    x = GlobalAveragePooling2D()(x)
    x = Dense(128, activation="relu")(x)
    outputs = Dense(k, activation="softmax")(x)
    model = Model(inputs, outputs)
    return model

```

