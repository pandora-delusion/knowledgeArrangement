# YOLOv3

### 边框预测

​	基于YOLO9000，YOLOv3使用k-means产生的anchor box来预测边框。网络预测每个边框的4个坐标，$t_{x}, t_{y}, t_{w}, t_{h}$。$(c_{x}, c_{y})$是每个单元格的左上角的坐标，边框先验的宽度和高度为$p_{w}, p_{h}$，那么对应的预测为：
$$
b_{x}=\sigma(t_{x})+c_{x}\\
b_{y}=\sigma(t_{y})+c_{y}\\
b_{w}=p_{w}e^{t_{w}}\\
b_{h}=p_{h}e^{t_{h}}
$$
​	在训练中，我们使用平方和误差损失函数（MSE）作为坐标的的损失函数，在计算损失函数前要对$t_{x},t_{y}$做logistic。YOLOv3使用logistic回归预测每个边框的objectness得分。如果一个边框先验与gt目标的覆盖率大于其他边框先验，那么为正标签。如果边框先验不是最好的但与gt目标的覆盖率大于某个阈值的化，我们就忽略这些预测，这个阈值为0.5。和Faster R-CNN不同，我们给每个gt目标只分配一个边框先验。***If a bounding box prior is not assigned to a ground truth object it incurs no loss for coordinate or class predictions, only objectness.***

### 分类预测

​	分类预测可以支持多标签预测。论文中没有使用softmax，因为对于性能而言它不是必要的，相反，论文中使用独立的逻辑分类（logistic classifiers）。在训练过程中，使用二元交叉熵损失（binary cross-entropy loss）来进行分类预测。对于更复杂的数据集，比如Open Images，这个公式会有所帮助。这个数据集中有许多重叠标签，softmax不适用于这种情况。

```python
import torch
import torch.nn as nn

class YOLOLayer(nn.Module):
    ...
    
    def forward(self, x, targets=None, img_dim=None):
        FloatTensor = torch.cuda.FloatTensor if x.is_cuda else torch.FloatTensor
        
        self.img_dim = img_dim
        num_samples = x.size(0)
        grid_size = x.size(2)
        
        # (num_samples, num_anchors, grid_size, grid_size, num_classes+5)
        prediction = (
            x.view(num_samples, self.num_anchors, self.num_classes+5, grid_size, grid_size).permute(0, 1, 3, 4, 2).contiguous()
        )
        
        # 使用sigmoid函数，将偏移限制在0到1之间
        x = torch.sigmoid(prediction[..., 0])
        y = torch.sigmoid(prediction[..., 1])
        w = prediction[..., 2]
        h = prediction[..., 3]
        # objectness的预测使用logistic regression
        pred_conf = torch.sigmoid(prediction[..., 4])
        # cls使用独立的logistc classification而不是softmax
        pred_cls = torch.sigmoid(prediction[..., 5:])
        
        if grid_size != self.grid_size:
            self.compute_grid_offsets(grid_size, cuda=x.is_cuda)
            
        pred_boxes = FloatTensor(prediction[..., :4].shape)
        pred_boxes[..., 0] = x.data + self.grid_x
        pred_boxes[..., 1] = y.data + self.grid_y
        pred_boxes[..., 2] = torch.exp(w.data)*self.anchor_w
        pred_boxes[..., 3] = torch.exp(h.data)*self.anchor_h
        
        output = torch.cat(
            (
                pred_boxes.view(num_samples, -1, 4)*self.stride,
                pred_conf.view(num_samples, -1, 1),
                pred_cls.view(num_camples, -1, self.num_classes),
            ),
            -1,
        )
        
        if target is None:
            return output, 0
        else:
            iou_scores, class_mask, obj_mask, noobj_mask, tx, ty, tw, th, tcls, tconf = \
            build_targets(
                pred_boxes=pred_boxes, 
                pred_cls=pred_cls,
                target=targets,
                anchors=self.scaled_anchors,
                ignore_thres=self.ignore_thres, 
            )
            
            # 平方和误差损失函数（MSE）作为坐标的的损失函数
            loss_x = self.mse_loss(x[obj_mask], tx[obj_mask])
            loss_y = self.mse_loss(y[obj_mask], ty[obj_mask])
            loss_w = self.mse_loss(w[obj_mask], tw[obj_mask])
            loss_h = self.mse_loss(h[obj_mask], th[obj_mask])
            # 使用二元交叉熵损失函数计算objectness和cls
            loss_conf_obj = self.bce_loss(pred_conf[obj_mask], tconf[obj_mask])
            loss_conf_noobj = self.bce_loss(pred_conf[noobj_mask], tconf[noobj_mask])
            loss_conf = self.obj_scale*loss_conf_obj+self.noobj_scale*loss_conf_noobj
            loss_cls = self.bce_loss(pred_cls[obj_mask], tcls[obj_mask])
            total_loss = loss_x + loss_y + loss_w + loss_h + loss_conf + loss_cls
            
            .....
            
            return output, total_loss
```



### 跨尺度预测

​	YOLOv3在三个不同的尺度上预测边框。系统使用于特征金字塔（feature pyramid networks）相似的概念从这些尺度中提取特征。为了解决YOLO很难精确定位小对象的问题，YOLOv3采用和SSD不同的方法，通过对高层特征图做上采样然后和底层特征图沿着通道维度堆叠，有点类似ResNet的恒等映射。在对堆叠起来的低分辨率的特征图做特征提取和回归检测来提高模型对小型目标的识别能力。该方法允许我们从上采样的特征中获取更有意义的语义信息，并从早期的特征图中获取更细粒度的信息。 然后，再添加几个卷积层来处理这个组合的特征图，并最终预测出一个类似的张量，尽管现在张量是原来的两倍。（具体设计去看代码）

### 特征提取

新网络是YOLOv2，Darknet-19和residual network之间的混合，称为Darknet-53

![1563953173274](F:\mycode\knowledgeArrangement\vision\darknet53.png)

新网络比Darknet-19强得多，但之比ResNet-101和ResNet-152快一点。性能和最先进的分类器相当，但是浮点运算更少，速度更快。

### 训练

在训练过程中没有使用hard negative mining，但使用了多尺度训练，大量数据增强，BN层以及许多标准的新技术。



如何我们增加IOU阈值的值，会发现系统的性能显著下降，这表明YOLOv3很难使边框和目标完全对齐（可以理解为难以精确定位？）通过多尺度预测，YOLOv3对小目标的预测能力相对较高，然而在中、大尺寸物体上性能相对较差，原因不明。



### 一些没能成功的尝试

1. 像Faster R-CNN那样的anchor box的x，y偏置预测。因为作者发现这个公式降低了模型的稳定性，效果不太好
2. 对x，y使用线性预测而不是logistic。这样会导致mAP的下降
3. Focal loss。导致了mAP的下降，可能没有必要
4. 双重IOU阈值和真值分配。Faster R-CNN使用两个IOU阈值，一个是0.7，大于0.7的都是正样本；[0.3,0.7]之间的样本忽略，小于0.3的gt目标是负样本。这种策略在YOLOv3中不成功

