### 目标和

题目地址为：[目标和](https://leetcode-cn.com/problems/target-sum/)

原问题等同于：找到nums的一个正子集和一个负子集，使得总和等于target

我们假设P是正子集，N是负子集。那么如何将其转换为子集求和问题：

```python
sum(P) - sum(N) = target
sum(P) + sum(N) + sum(p) - sum(N) = target + sum(P) + sum(N)
2*sum(P) = target + sum(nums)
```

因此，原来的问题就转化为一个求子集的和问题：找到nums的一个子集P，使得
$$
sum(P) = (target+sum(nums))/2
$$
请注意，上面的公式已经证明target+sum(nums)必须是偶数，否则输出为0

python代码如下：

```python
class Solution:

    def findTargetSumWays(self, nums: List[int], S: int) -> int:

        sums = 0
        
        for num in nums:
            sums += num
        
        if sums < S or (S+sums) % 2 == 1:
            return 0
        else:
            return self._subsetSum(nums,(sums+S)//2)
    
    # 该算法在内存占用上小于dp但速度比dp要慢得多，追求速度性能请使用dp  
    # def _subsetSum(self, nums, idx, length, s):
    #     if idx == length:
    #         if s == 0:
    #             return 1
    #         return 0
        
    #     res = 0
    #     if nums[idx] <= s:
    #         res += self._subsetSum(nums, idx+1, length, s-nums[idx])
    #     res += self._subsetSum(nums, idx+1, length, s)
    #     return res

    def _subsetSum(self, nums, s):
        dp = [0]*(s+1)
        dp[0] = 1
        for num in nums:
            for i in range(s, num-1, -1):
                dp[i] += dp[i-num]
        return dp[s]
```

