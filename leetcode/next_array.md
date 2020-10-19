# 下一个排列

### 全排列的字典序

​	给定多个字符，可以按照任意顺序进行排列，所有排列称为全排列。

​	每一种排列对应一个字符串，如果这些字符串按照字符串大小的顺序进行排序，那么就这种排序是基于字典序的全排列。

### 字典序算法

​	字典序算法用来解决这样一个问题：给定其中一种排列，求基于字典序的下一种排列。

​	比如，"358764"->"364578"

代码如下：

```python
class Solution:
    def nextPermutation(self, nums: List[int]) -> None:
        """
        Do not return anything, modify nums in-place instead.
        """
        size = len(nums)
        if size <= 1:
            return nums
        idx = size - 2
        while idx >= 0:
            if nums[idx+1]>nums[idx]:
                break
            idx -= 1
        if idx >= 0:
            i = size - 1
            while i > idx:
                if nums[i] > nums[idx]:
                    break
                i -= 1
            tmp = nums[idx]
            nums[idx] = nums[i]
            nums[i] = tmp
        
        idx += 1

        var = idx + size - 1
        for j in range(idx, var//2+1):
            tmp = nums[j]
            nums[j] = nums[var-j]
            nums[var-j] = tmp
```

