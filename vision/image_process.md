# 一些重要的图像处理方法

图片的白化操作相对简单，由于图片的各个通道、像素分布基本相同，不需要对像素值做规范化，所以白化操作只需要减去个通道均值即可

```python
MEAN = [_R_MEAN, _G_MEAN, _B_MEAN]

def tf_image_whitened(image, means=MEAN):
    if image.get_shape().ndims != 3:
        raise ValueError("Input must be of size [height, width, C>0]")
       num_channels = image.get_shape().as_list()[-1]
    if len(means) != num_channels:
        raise ValueError("len(means) must match the number of channels")
    mean = tf.constant(means, dtype=image.dtype)
    image = image - mean
    return image

def tf_image_unwhitened(image, mean=Mean, to_int=True):
    mean = tf.constant(means, dtype=image.dtype)
    image = image + mean
    if to_int:
        image = tf.cast(image, tf.int32)
    return image
```

调整图片到指定大小的几种方法：

```python
# 对图片进行中心裁剪或填充，并调整边框
def resize_image_bboxes_with_crop_or_pad(image, bboxes, target_height, target_width):
    with tf.name_scope("resize_with_crop_or_pad"):
        height, width, _ = image.get_shape().as_list()
        width_diff = target_width - width
        offset_crop_width = max(-width_diff//2, 0)
        offset_pad_width = max(width_diff//2, 0)
        
        height_diff = target_height - height
        offset_crop_height = max(-height_diff//2, 0)
        offset_pad_height = max(height_diff//2, 0)
        
        height_crop = min(target_height, height)
        width_crop = min(target_width, width)
        
        # 将图片进行中心裁剪
        cropped = tf.image.crop_to_bounding_box(image, offset_crop_height, offset_crop_width, height_crop, width_crop)
        
        # 调整边框以适应裁剪后的而图片（这不是重点）
        bboxes = bboxes_crop_or_pad(bboxes, height, width, -offset_crop_height, 
                                   -offset_crop_width, height_crop, width_crop)
        # 再填充图片，没必要就不填充
        resized = tf.image.pad_to_bounding_box(cropped, offset_pad_height, pffset_pad_width, target_height, taregt_width)
        
        # 再次调整边框以适应填充后的图片
        bboxes = bboxes_crop_or_bad(bboxes, height_crop, width_crop, offset_pad_height, offset_pad_width, target_height, target_width)
        return resized, bboxes

# 先缩放图片，再对小的那边进行填充
shape = tf.shape(image)
factor = tf.minimum(tf.to_double(1.0), 
                   tf.minimum(tf.to_double(out_shape[0] / shape[0]),
                             tf.to_double(out_shape[1]/ shape[1])))
resize_shape = factor*tf.to_double(shape[0:2])
resize_shape = tf.cast(tf.floor(resize_shape), tf.int32)

# 使用双线性插值法按比例调整图像大小
with tf.name_scope("resize_image"):
    height, width, channels = image.get_shape().as_list()
    image = tf.expand_dims(image, 0)
    iamge = tf.image.resize_images(image, resize_shape, tf.image.ResizeMethod.BILINEAR, False)
    image = tf.reshape(image, tf.stack([resize_shape[0], resize_shape[1], channels]))
# 填充到预期的大小
image, bboxes = resize_image_bboxes_with_crop_or_bad(image, bboxes, 
                                                     out_shape[0], out_shape[1])

# 会弯曲图像的调整
with tf.name_scope("resize_image"):
    height, width, channels = image.get_shape().as_list()
    image = tf.expand_dims(image, 0)
    iamge = tf.image.resize_images(image, resize_shape, tf.image.ResizeMethod.BILINEAR, False)
    image = tf.reshape(image, tf.stack([resize_shape[0], resize_shape[1], channels]))

        
```

### 常用的图像增强方法

```python
import numpy as np
import cv2

def clip(image):
    return np.clip(image, 0, 255).astype(np.uint8)

def adjust_constrast(image, factor):
    """调整图像的明暗对比"""
    mean = image.mean(axis=0).mean(axis=0)
    return clip((image-mean)*factor+mean)

def adjust_brightness(image, delta):
    """调整图像的亮度"""
    return clip(image+delta*255)

def adjust_hue(image, delta):
    """调整图像的颜色，delta在-1和1之间"""
    image[..., 0] = np.mod(image[..., 0]+delta*180, 180)
    return image

def adjust_saturation(image, factor):
    """调整色彩饱和度"""
    image[..., 1] = np.clip(image[..., 1]*factor, 0, 255)
    return image

class VisualEffect:
    """
    创建图像调整类，管理各种参数并方便调用
    """
    def __init__(self, constrast_factor, 
                brightness_delta, 
                hue_delta,
                saturation_factor):
        self.constrast_factor = constrast_factor
        self.brightness_delta = brightness_delta
        self.hue_delta = hue_delta
        self.saturation = saturation_factor
	
    def __call__(self, image):
        """
        注意image的通道顺序为BGR，在调节图像颜色和色彩饱和度时要将
        图片从BGR格式转化为HSV格式
        """
        if self.constrast_factor:
            image = adjust_constrast(image, self.constrast_factor)
        if self.brightness_delta:
            image = adjust_brightness(image, self.brightness_delta)
            
        if self.hue_delta or self.saturation_factor:
            
            image = cv2.cvtColor(image, cv2.COLOR_BGR2HSV)
            
            if self.hue_delta:
                image = adjust_hue(image, self.hue_delta)
            if self.saturation_factor:
                image = adjust_saturation(image, self.saturation_factor)
            
            image = cv2.cvtColor(image, cv2.COLOR_HSV2BGR)
        return image

def uniform(val_image):
    return np.random.uniform(val_range[0], val_range[1])

def ramdom_visual_effect_generator(
			constrast_range=(0.9, 1.1),
			brightness_range=(-0.1, 0.1),
			hue_range=(-0.05, 0.05),
			saturation_range=(0.95, 1.05)):
    
    def _generator():
        while True:
            yield VisualEffect(
            	constrast_factor=uniform(constrast_range),
            	brightness_delta=uniform(brightness_range),
            	hue_delta=uniform(hue_range),
            	saturation_factor=uniform(saturation_range))
    return _generator()

def transform_coordinate(transform, aabb):
   	# 边框坐标系变换 
    xmin, xmax, ymin, ymax = aabb
    
    points = transform.dot([
        [xmin, ymax, xmin, xmax],
        [ymin, ymax, ymax, ymin],
        [1, 1, 1, 1],
    ])
    
    min_corner = points[0:2, :].min(axis=1)
    max_corner = points[0:2, :].max(axis=1)
    
    return [min_corner[0], max_corner[0], min_corner[1], max_corner[1]]

def rotation(angle):
    """构造一个二维旋转矩阵"""
    return np.array([
        [np.cos(angle), -np.sin(angle), 0],
        [np.sin(angle), np.cos(angle), 0],
        [0, 0, 1]
    ])

def random_rotation(min, max, default=np.random):
    return rotation(default.uniform(min, max))

def translation(translation):
   	"""构造一个二维平移矩阵"""
    return np.array([
        [1, 0, translation[0]],
        [0, 1, translation[1]],
        [0, 0, 1]
    ])

def random_vector(min, max, default=np.random):
    min = np.array(min)
    max = np.array(max)
    assert(min.shape == max.shape)
    return default.uniform(min, max)

def random_translation(min, max, default=np.random):
    return translation(random_vector(min, max, default))

def shear(angle):
    """构造一个二维shear矩阵"""
    return np.array([
        [1, -np.sin(angle), 0],
        [0, np.cos(angele), 0],
        [0, 0, 1]
    ])

def random_shear(min, max, default=np.random):
    return shear(default.uniform(min, max))

def scaling(factor):
    """构造二维缩放矩阵"""
    return np.array([
        [factor[0], 0, 0],
        [0, factor[1], 0],
        [0, 0, 1]
    ])

def random_scaling(min, max, default=np.random):
    return scaling(random_vector(min, max, default))

def random_flip(flip_x, flip_y, default_np.ramdom):
    """构造二维随机翻转矩阵"""
    _x = default.uniform(0, 1) < flip_x
    _y = default.uniform(0, 1) < flip_y
    _x = int(_x)
    _y = int(_y)
    return np.array([
        [(-1)**_x, 0, _x],
        [0, (-1)**_y, _y],
        [0, 0, 1]
    ])

def random_transform(default=np.random, 
                    min_rotation=0,
                    max_rotation=0,
                    min_translation=(0, 0),
                    max_translation=(0, 0),
                    min_shear=0,
                    max_shear=0,
                    min_scaling=(1, 1),
                    max_scaling=(1, 1),
                    flip_x_chance=0,
                    flip_y_chance=0):
    res = np.linalg.multi_dot([
        random_rotation(min_rotation, max_rotation, default),
        random_translation(min_translation, max_translation, default),
        random_shear(min_shear, max_shear, default),
        random_scaling(min_scaling, max_scaling, default),
        random_flip(flip_x_chance, flip_y_chance, default)
    ])
    return res

def random_transform_generator(default=np.random, **kwargs):
    
    def _generator(width, height):
        res = random_transform(default, **kwargs)
        if "flip_x_chance" in kwargs:
            res[0, 2] *= width
        if "flip_y_chance" in kwrags:
            res[1, 2] *= height
        return res
    return _generator

import cv2

class TransformParameters:
    
    def __init__(self, fill_mode="nearest",
                interpolation="linear",
                cval=0,
                relative_translation=True):
        # 边界填充模式
        self.fill_mode = fill_mode
       	self.cval = cval
        # 插值模式
        self.interpolation = interpolation
        self.relative_translation = relative_translation
    
    def cvBorderMode(self):
        if self.fill_mode == "constant":
            return cv2.BORDER_CONSTANT
        elif self.fill_mode == "nearest":
            return cv2.BORDER_REPLICATE
        elif self.fill_mode == "reflect":
            return cv2.BORDER_REFLECT_101
        elif self.fill_mode == "wrap":
            return cv2.BORDER_WRAP
        
    def cvInterpolation(self):
        # 最邻近插值
        if self.interpolation == "nearest":
            return cv2.INTER_NEAREST
        # 线性插值
        elif self.interpolation == "linear":
            return cv2.INTER_LINEAR
        # 三次样条插值
        elif self.interpolation == "cubic":
            return cv2.INTER_CUBIC
        # 区域插值
        elif self.interpolation == "area":
            return cv2.INTER_AREA
        # lanczos插值
        elif self.interpolation == "lanczos4":
            return cv2.INTER_LANCZOS4
        
"""
填充方法：
常数：常数填充对于自然状态下的图像可能不奏效，但是可以用在单色背景下拍摄的图像
边缘：可以将图像的边缘值拓展到图像边界之外，这种方法也适用于轻微的图像平移
反射：可以沿着图像边界反射图像的像素值。这种方法对于包含树林，山脉等连续自然背景的图像，非常有用
对称：和反射类似，除了一点：在反射边界上拷贝边缘像素。正常情况下，反射和对称可以交替使用，但处理非常小的图像或模式时，差异会非常明显。
包裹：在超出图像边界之外的地方重复填充图像，就像平铺图像。由于对很多场景并无意义，不如其他方法那么常用。
"""    
def apply_transform(matrix, image, params=TransformParameters()):
    # 填充0像素并不总是合理的
    res = cv2.warpAffine(image, matrix[:2, :], 
                         dsize=(image.shape[1], image.shaoe[0]),
                         flags=params.cvInterpolation(),
                         borderMode=params.cvBorderMode(),
                         borderValue=params.cval)
    return res
```

##### 高斯噪声

当神经网络试图学习可能并无用处的高频特征时（即频繁发生的无意义模式），常常会发生过拟合。具有零均值特征的高斯噪声本质上就是在所以频率上都有的数据点，能有效使得高频特征失真，减弱它对模型的影响。这也意味着低频成分（通常也是我们关心的数据）也会失真，但是神经网络能够通过学习忽略这部分影响。添加正确数量的噪声就能增强神经网络的学习能力。

一个相对弱化的版本就是椒盐噪声，它是以随机的白色及黑色像素点铺满整个图像。给图像添加椒盐噪声的作用和添加高斯噪声时一样的，但产生的失真效果相对较弱。

```python
import tensorflow as tf

# 一张图片的placeholder
shape = [height, width, channels]
x = tf.placeholder(dtype=tf.float32, shape=shape)

# 添加高斯噪声
noise = tf.random_normal(shape=tf.shape(x), mean=0.0, stddev=1.0， dtype=tf.float32)
output = tf.add(x, noise)
```

##### 高级增强方法

条件式生成对抗网络

风格迁移

### 插值运算

​	在图像几何变换中，经过几何变换后的图像像素可能在原始图像中并没有对应的像素点，那么，在目标图像中这些没有对应点的像素灰度值该如何取值？通过图像插值，对于目标图像像素点的任何连续位置，都可以获得一个较为精确的插值。而插值函数需要尽可能的保留图像的细节，并且尽可能的减少引入人为噪声。

​	对于目标图像中的每一个离散的像素位置$(u^{'}, v^{'})$，通过逆变换函数，对应于原始图像中连续的位置$(x,y)$。目标图像中的$(u^{'},v^{'})$像素灰度值可以根据原始图像坐标$(x,y)$的某些邻域插值获得。

#### 一维插值

一维平面中的简单插值算法，使根据离散函数$g(u)$（u属于整数），插值生成任意连续位置对应的数值。

##### 最邻近插值

最邻近插值算法中，对连续坐标向下取整，得到与之最近的整数$u_{0}$，在根据离散采样函数$g(u_{0})$作为连续坐标x的估算值。即：
$$
\widehat{g}(x)=g(u_{0})\\
u_{0}=round(x)=\left \lfloor x+0.5 \right \rfloor
$$

##### 线性插值

线性插值的估算值是根据最近两个采样点的采样值$g(u_{0})$和$g(u_{0}+1)$，$u_{0}$为x向下取整，根据采样点离连续x的距离进行加权求和，得到最终的估计值
$$
\begin{align}
\widehat{g}(x)&=g(u_{0})+(x-u_{0})\cdot (g(u_{0}+1)-g(u_{0}))\\
			&=g(u_{0})\cdot (1-(x-u_{0})) + g(u_{0}+1)\cdot (x-u_{0})
\end{align}
$$

#### 二维插值

##### 二维最邻近插值

对于给定的连续坐标点$(x_{0},y_{0})$，与其最近的像素坐标即为分别对x坐标方向和y坐标方向进行取整，即：
$$
\widehat{I}(x_{0},y_{0})=I(u_{0}, v_{0})\\
u_{0}=round(x_{0})=\left \lfloor x_{0}+0.5 \right \rfloor \\
v_{0}=round(y_{0})=\left \lfloor y_{0}+0.5 \right \rfloor
$$

##### 双线性插值

对于给定的插值点$(x_{0}, y_{0})$，首先在原始图像中找到其最近的四个像素A，B，C，D：
$$
A=I(u_{0},v_{0})\\
B=I(u_{0}+1, v_{0})\\
C=I(u_{0}, v_{0}+1)\\
D=I(u_{0}+1, v_{0}+1)
$$
$u_{0}$为$x_{0}$向下取整，$v_{0}$为$y_{0}$向下取整，四个像素点的灰度值分别从水平方向和竖直方向进行插值。中间点E、F是由插值点坐标$(x_{0},y_{0})$与$u_{0}$之间的水平距离决定：
$$
E=A+(x_{0}-u_{0})\cdot (B-A)\\
F=C+(x_{0}-u_{0})\cdot (D-C)\\
G=(1-y_{0}+v_{0})E + (y_{0}-v_{0})F
$$

##### 双三次样条插值

（。。。。）