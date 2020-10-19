# 图像处理中的L1-normalize和L2-normalize

当一幅图像用某种特征表示出来，一般要进行L1-normalize和L2-normalize。

假设一幅图像表示为Y=[$x_{1}, x_{2}, x_{3},x_{4},x_{5}$]

L1-normalize的结果为：
$$
Y_{i}=\frac{X_{i}}{\sum^{5}_{j=1}X_{j}} 
$$
L2-normalize的结果为：
$$
Y_{i}=\frac{X_{i}}{\sum_{j=1}^{5}X_{j}^{2}}
$$
（。。。。哪个更好有待补充）