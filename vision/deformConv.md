# Deformable Convolutional Networks

​	本文引入两个新的模块来增强CNN对形状变化的建模能力，可变形卷积和可变形池化层。新的模块可以容易地嵌入现存的普通卷积模型中并可以容易地进行端到端的训练。

​	现有的卷积模型阻止了对具有未知几何变换的新任务的泛化能力，而这些未知的几何变换没有得到适当的建模。现有的卷积模型对几何变换的建模能力主要来自于广泛的数据增广、更大的模型容量和一些简单的模块。这种局限性来源于CNN模块固定的几何结构：卷积单元在固定位置采样输入特征图；池化层以固定尺寸降低空间分辨率；ROI池化层将ROI分离到固定的空间单元中。所以他们缺乏处理几何变换的内部机制。这对于在空间位置上编码语义的高层CNN层来说是不可取的。

​	由于不同的位置可能对应着不同尺度或变形的物体，因此自适应确定尺度或接受域大小是精细定位视觉识别的需要。

### 可变形卷积（deformable convolution）

在标准卷积中给一般的网格采样位置添加二维的偏置，这使得采样网格能够自由变形。通过额外的卷积层从上一层特征图中学习偏置。

在可变形卷积中，使用偏置$\left \{ \Delta P_{n},n=1,\cdots,N\right \}$对固定网格R进行增广:
$$
y\left ( P_{0} \right )=\sum _{P_{n} \in R}w\left ( P_{n} \right )\cdot X\left ( P_{0}+P_{n}+\Delta P_{n} \right )
$$
因为偏移量通常是小数，所以通过双线性插值实现：
$$
X(P) = \sum_{q}G(q,p)\cdot x(q)
$$
其中P指代任意（小数）位置（$P=P_{0}+P_{n}+\Delta P_{n}$），q枚举了特征图所有的空间位置。G为双线性插值核函数

卷积核与当前的卷积层有相同的空间分辨率和膨胀率，输出偏置域和输入特征图具有相同的空间结构

```python
import torch
import numpy as np
from torch.autograd import Variable

def np_repeat_2d(a, repeats):
    assert len(a.shape) == 2
    a = np.expand_dims(a, 0)
    a = np.tile(a, [repeats, 1, 1])
    return a

def th_generate_grid(num_batch, height, width, dtype, is_cuda):
    grid = np.meshgrid(range(height), range(width), indexing="ij")
    grid = np.stack(grod, axis=-1)
    grid = grid.reshape(-1, 2)
    
    grid = np_repeat_2d(grid, num_batch)
    grid = torch.from_numpy(grid).type(dtype)
    
    if is_cuda:
        grid = grid.cuda()
    return Variable(grid, requires_grad=False)

def th_batch_map_offsets(x, offsets, grid=None):
    num_batch = x.size(0)
    height = x.size(2)
	width = x.size(3)
    
    offsets = offsets.contiguous().view(num_batch, -1, 2)
    
    if grid is None:
        grid = th_generate_grid(num_batch, height, width, 
                               offsets.data.type(), offsets.data.is_cuda)
    coords = offsets + grid
    
    mapped_vals = th_batch_map_cooradinates(x, coords)
    return mapped_vals

class ConvOffset2D(torch.nn.Module):
    
    @staticmethod
    def init_weights(weight, std):
        torch.nn.init.normal_(weight, 0.0, std)
        
    def __init__(self, filters, **kwargs):
        self.filters = filters
        super(ConvOffset2D, self).__init__(self.filter, 2, 3, padding=1, bias=False, **kwargs)
        self._grid_param = None
        self._grid = None
        
    @staticmethod
    def _to_bc_h_w_2(x):
        x = x.permute(0, 2, 3, 1)
        return x
    
    def _get_grid(self, x):
        num_batch, height, width = x.size(0), x.size(2), x.size(3)
        dtype, cuda = x.data.type(), x.data.is_cuda
        if self._grid_param == (num_batch, height, width, dtype, cuda):
            return self._grid
        
       	self._grid_param = (num_batch, height, width, dtype, cuda)
        self._grid = th_generate_grid(num_batch, height, width, dtype, cuda)
        return self._grid
    
    def forward(self, x):
        offsets = super(ConvOffset2D, self).forward(x)
        offsets = self._to_bc_h_w_2(offsets)
        
        x_offsets = th_batch_map_offsets(x, offsets, grid=self._get_grid(x))
        return x_offsets
    
```

相当于一个额外的卷积当它扫描特征图上的某个位置和它周围的位置后，就立刻知道该从哪些位置提取特征能够更好地向后面的卷积层传递语义信息。

### 可变形ROI池化层（deformable ROI pooling）

在上一个ROI池化层的每个位置的一般单元添加偏置，类似地，偏移量从前面的特征图或者ROI中学习，从而为具有不同形状的对象实现自适应局部定位

可变形卷积神经网络的一个关键区别是，它们以一种简单高效端到端的方式处理密集的空间转换。

给定输入特征图X和RoI尺寸$w \times h$以及左上角$p_{0}$，RoI池化层将RoI分成$k \times k$个单元然后输出为$k \times k$个特征图y。那么第$(i,j)$个单元是：
$$
y \left ( i,j \right ) = \sum_{p \in bin(i,j)}x(p_{0}+p)/n_{ij}
$$
$n_{i,j}$是每个单元的像素的数量。

类似地，在可变形RoI池化层中，偏置为$\{\Delta p_{ij} | 0 \leq i,j \lt k\}$：
$$
y \left( i,j \right) = \sum_{p \in bin(i,j)}X(p_{0}+p+\Delta p_{ij})/n_{ij}
$$
上式同样由双线性插值实现。

![image-20191113214411090](F:\mycode\knowledgeArrangement\vision\deform_pool.png)

首先，ROI池化生成一个池化后的特征图，从图中，一个全连接层生成规范化后的偏置$\Delta \widehat {P_{ij}}$，然后使用下式转换成$\Delta P$：
$$
\Delta P_{ij} = \gamma \cdot \Delta \ \widehat{P_{ij}}\otimes (w, h) 
$$
一般置$\gamma$为0.1，偏置规范化是必要的，这使得偏置能学到RoI大小的不变性。

位置敏感的RoI池化层（暂不深入理解）



在训练时，这些添加的卷积层和全连接层的权重初始化为0，学习率置为现存层的$\beta$ 倍

### 理解可变形卷积网络

本文的工作是建立在用额外的偏移量扩展卷积核RoI池中的空间采样位置，并从目标任务中学习偏移量的思想。