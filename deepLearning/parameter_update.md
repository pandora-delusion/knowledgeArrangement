### 参数更新
```python
# Vanilla update
x += - learning_rate *dx
```
##### Momentum update
```python
# Momentum update
v = mu*v - learning_rate*dx # 整合速度
x += v # 整合位置
```
这个性质允许momentum在低曲率方向上累积动量在多次迭代中保持不变，与梯度下降相比，在这类方向上会加速进展。

变量v被初始化为0，mu是一个额外的超参数，一个典型的取值为0.9。其物理意义与摩擦系数相一致，这个系数会降低速度，降低系统的动能，否则系统永远不会停下。在交叉验证中，参数的设置范围是[0.5, 0.9, 0.95, 0.99]，一个典型的设置是从大约0.5的动量开始，并在多个时期将其退火到0.99左右。

##### Nesterov Momentum 
它对凸函数有较强的理论收敛保证，在实际应用中，它的工作效率略好于标准momentum
Nesterov加速梯度是一种一阶最优化方法，被证明相比于一般凸函数的梯度下降有更好的收敛速率保证。
![avatar](nesterov.jpeg)
Nesterov Momentum，不是在当前位置评估梯度（红色圆圈），我们知道我们的momentum将把我们带到绿色箭头处，我们将在那个位置评估梯度
```python
x_ahead = x + mu*v
# 评估x_ahead处的梯度
v = mu*v - learning_rate*dx_ahead
x += v
```
见[Training Recurrent Neural Networks](http://www.cs.utoronto.ca/~ilya/pubs/ilya_sutskever_phd_thesis.pdf)7.2章
通过变换写成下面的形式：
```python
# 有时间看一下原论文，理解推导公式
v_prev = v
v = mu*v - learning_rate*dx
x += -mu * v_prev + (1 + mu)*v
```
热处理(Annealing)学习速率：
在训练深度网络时，在整个时间内退火学习速率是很有帮助的。衰减学习速率的时机是非常严苛的:缓慢的衰退你将会浪费计算量并且没有改进，但衰减太过激进系统又将过早冷却，无法到达能达到的最佳位置。有三种常见的实现学习速率衰减的常见方案：
 + Step decay 每个epoch在学习速率上乘以衰减因子。典型的值是每5个epoch衰减一半，每20个epoch衰减到0.1。这个值依赖于问题的类型和模型。在实践中有一种启发式算法，即在一固定的学习速率进行训练时观察验证误差，并在验证误差停止改进时将学习速率降低一个常量
 + Exponential dacay 有一个数学形式$\alpha=\alpha_{0}e^{-kt}$，$\alpha_{0}$，k是超参数，t是迭代次数（epoch）
 + 1/t decay 数学形式$\alpha=\alpha_{0}/(1+kt)$，$\alpha_{0}$，k是超参数，t是迭代次数

##### [Second order methods](http://cs231n.github.io/neural-networks-3/)

##### Adagrad 
一种自适应学习速率方法
```python
cache += dx**2
x += - learning_rate * dx / (np.sqrt(cache) + eps)
```
请注意，接受高梯度的权重将降低其有效学习率，而接受小的或不经常更新的权重将提高其有效学习率。没有平方根算法的执行将非常糟糕。eps防止被除0。adagrad的一个缺点是，在深度学习的情况下，单调的学习速率通常被证明是过于激进的，并且过早地停止学习。

##### [RMSProp](http://www.cs.toronto.edu/~tijmen/csc321/slides/lecture_slides_lec6.pdf)
RMSProp修正了Ada的激进。
```python
cache = decay_rate * cache + (1 - decay_rate) * dx**2
x += - learning_rate * dx / (np.sqrt(cache) + eps)
```
decay_rate一般取值维[0.9, 0.99, 0.999]
RMSProp仍然根据其梯度的大小调节每个权重的学习速率，这里有一个有益的均衡效果，但与adagrad不同，更新不会单调的变小

##### [Adam](https://arxiv.org/pdf/1412.6980.pdf)
```python
m = beta1*m + (1-beta1)*dx
v = beta2*v + (1-beta2)*(dx**2)
x += - learning_rate * m / (np.sqrt(v) + eps)
```
推荐eps=1e-8,beta1=0.9,beta2=0.999
adam是当前推荐使用的默认算法，有些时候，值得尝试SGD+Nesterov Momentum
完整的Adam更新还包括一个偏差校正机制，该机制补偿了这样一个事实：在最初的几个时间步骤中，向量m，v都是初始化的，因此在它们完全“预热”之前偏向于零。在偏差修正机制下，更新如下：
```python
# t 是迭代计数
m = beta1*m + (1-beta1)*dx
mt = m / (1-beta1**t)
v = beta2*v + (1-beta2)*(dx**2)
vt = v / (1-beta2**t)
x += - learning_rate * mt / (np.sqrt(vt) + eps)
```
请注意，更新现在是迭代的函数，也是其他参数的函数。

##### 超参数最优
最常见的网络超参数有：
- 初始学习速率
- 学习速率衰减策略
- 正则强度（L2 惩罚，dropout强度）

实现：
一个特殊的设计是不断地对随机超参数进行采样并执行最优化，通过训练，在每个epoch后追踪验证集性能，设置模型检查点，将检查点搜集到的统计数据（loss，输入分布等等）写入文件方便检查和排序进展。然后还有第二个程序，在计算集群中启动或者停止系统，还可以额外检查检查点并绘制训练统计数据等。

选择交叉验证的折：
在大多数情况下，一个大小合适的验证集大大简化了代码库，而无需使用多个折叠进行交叉验证。

超参数范围：
在对数刻度上搜索超参数
一个典型的学习速率采样如下：
```python
learning_rate = 10**uniform(-6, 1)
```
同样的策略也应用于正则化强度。直观的说，这是因为学习速度和正则化强度对训练动态性有乘数影响，但是一些参数，比如dropout，还是一般用原始度量搜索
```python
dropout = uniform(0, 1)
```

随机搜索比网格搜索更好：
随机搜索比网格搜索更加有效率也更加容易实验
- 注意边界上的最佳值
- 从粗到细的搜索阶段
此外，在只训练1个或更少的时间段的情况下执行初始粗糙搜索也很有帮助，因为许多超参数设置可能导致模型根本无法学习，或者立即损失爆炸。然后，第二个阶段可以用5个时间段执行更窄的搜索，最后一阶段可以在最终范围内对更多时间段执行详细的搜索
- 贝叶斯超参数优化
贝叶斯超参数优化是一个研究领域，致力于提出更有效地导航超参数空间的算法。核心思想是在查询不同超参数下的性能时，适当的平衡exploration-exploitation，很多库可以使用，比如[Spearmint](https://github.com/JasperSnoek/spearmint), [SMAC](http://www.cs.ubc.ca/labs/beta/Projects/SMAC/)，[Hyperopt](http://jaberg.github.io/hyperopt/)

##### 评估
模型集成
在实践中，将神经网络的性能提高几个百分点的一个可靠方法是训练多个独立的模型，并在测试时对它们的预测进行平均。随着集成中模型数量的增加，性能通常单调地提高（尽管回报递减）
有几种方法可以组成一个集成：
+ 相同的模型，不同的初始化
使用交叉验证确定最佳超参数，然后使用最佳超参数集训练多个模型，但使用不同的随机初始化。这种方法的危险在于，变化只是由于初始化。
+ 交叉验证期间发现的顶级模型
使用交叉验证确定最佳的超参数，然后选出最好的几个模型组成集成。
+ 单个模型的不同检查点
+ 训练期间参数的运行平均值