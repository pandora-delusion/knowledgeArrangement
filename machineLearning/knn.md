# KNN

## L1(Manhattan) distance
$d_{1}\left ( I_{1},I_{2} \right )= \sum_{p}\left | I_{1}^{p}-I_{2}^{p} \right |$

## L2(Euclidean) distance
$d_{2}\left ( I_{1},I_{2} \right )= \sqrt{\sum_{p}\left ( I_{1}^{p}-I_{2}^{p} \right )^{2}}$

#### (待解决)L1 distance和L2 distance有什么区别，各自有什么优缺点？

## 其他几种距离
## Minkowski distance(明可夫斯基距离)
$dist\left ( X,Y \right )=\left ( \sum_{i=1}^{n}\left | x_{i}-y_{i} \right | ^{p}\right )^{1/p}$

## chebyshev distance(切比雪夫距离)
$dist\left ( X,Y \right )=\lim_{p\rightarrow \infty }\left ( \sum_{i=1}^{n}\left | x_{i}-y_{i} \right |^{p} \right )^{1/p}=max\left | x_{i}-y_{i} \right |$

## Mahalanobis distance[马氏距离](https://www.cnblogs.com/DPL-Doreen/p/8183909.html)

对于KNN算法，K值的选择十分重要，较小的K值容易受到噪声干扰，较大的K值会导致边界上样本的分类有歧义