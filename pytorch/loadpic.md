# Pytorch图片的加载

当使用pytorch的预训练权重时，图片在输入神经网络前，除了各种增强，Resize之外，必须要先归一到$[0,1]$区间内，然后再做Normalize处理，如下：

```python
import pytorch
import torchvision

def simple_process():
    train_augment = torchvision.transforms.Compose([
        # 必要的其他处理
        torchvision.transforms.ToTensor(), # 将像素值归一化到[0, 1]区间内
        torchvision.transforms.Normalize([0.485, 0.456, 0.406],
                                         [0.229, 0.224, 0.225])
    ])
    
    val_augment = torchvision.transforms.Compose([
        torchvision.transforms.ToTensor(),
        torchvision.transforms.Normalize([0.485, 0.456, 0.406],
                                         [0.229, 0.224, 0.225])
    ])
   	return train_augment, val_augment
```

