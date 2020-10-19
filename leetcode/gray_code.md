# 格雷编码

格雷编码是一个二进制数字系统，在该系统中，两个连续的数值仅有一个位数的差异。

给定一个代表编码总位数的非负整数 n，打印其格雷编码序列。格雷编码序列必须以 0 开头。

二进制转各类编码是有公式的，即：
$$
i\wedge \left ( i\gg 1 \right )
$$
代码如下：

```python
class Solution:
    def grayCode(self, n: int) -> List[int]:
        size = 1 << n
        res = []
        for i in range(size):
            res.append(i^(i >> 1))
        if size == 0:
            res.append(0)
        return res
```

