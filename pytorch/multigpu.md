# pytorch下多GPU训练

### DataParallel类

​	在模块级别实现数据并行的类，继承自torch.nn.Module。该容器通过将输入batch分割到指定的设备上来实现并行。除了batch，其他对象在每个设备上都复制一次。在向前传播的过程中。mudule被复制到每个设备上，每个副本处理一个输入部分。在反向传播期间，每个副本的将会在原始module上求和。batch的大小应该大于所使用的GPU的数量。

​	除了几个这顶的类型，任何输入都允许被传入DataPallel类对象，tensor将会沿着第0个维度（或者指定的维度 ，默认为0）被分割。元组，列表，字典类型将会被浅拷贝。*其他类型的对象将在不同进程见共享，但是如果在模型的前向传播中写这些对象会破坏进程同步*

​	属性module在运行DataParallel前必须在device_ids[0]上有参数和缓冲。

警告：

1. 属性module和它的子模型的forward和backward的hooks将被调用len(device_ids)次，每次的输入是对应的设备中的部分。注意，hooks只能保证在对应的设备内按正确的顺序执行。例如，无法保证由register_forward_pre_hook所设置的hooks在所有设备的forward方法前调用，但是能保证其他对应设备的forward方法前调用的。
2. 当属性module返回一个张量（比如0维），包装器将会返回一个长度为数据并行设备数量的向量，包含每个设备的结果。
3. 