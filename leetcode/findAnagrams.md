#  找到字符串中所有字母异位词

leetcode原理在[这里](https://leetcode-cn.com/problems/find-all-anagrams-in-a-string/)

这种题型需要重点关注，解体大致思路是使用滑动窗口，但在leetcode提交的结果上并不理想，应该有更好的解决办法。

标准滑动窗口的解法如下：

```python
class Solution:
    def findAnagrams(self, s, p):
        a2z = [0]*26
        cura2z = [0]*26

        res = []
        pLen = len(p)

        for c in p:
            a2z[ord(c)-97] += 1

        for i in range(0, len(s)):
            if i >= pLen:
                cura2z[ord(s[i-pLen])-97] -= 1
            cura2z[ord(s[i])-97] += 1

            if cura2z == a2z:
                res.append(i-pLen+1)
        
        return res
```

下面是自己的解法，性能差不多（稍微节省了点内存和计算）但是代码不简洁，体现了自己混乱的思路：

```python
class Solution:
    def findAnagrams(self, s, p):

        dct = {}
        for var in p:
            if var not in dct:
                dct[var] = 0
            dct[var] += 1
        
        test = dict(dct)
        first = {}
        res = []
        left = 0

        for idx, sym in enumerate(s):
            if sym not in test:
                left = idx + 1
                test = dict(dct)
                first = {}
            elif test[sym] == 0:
                origin = left
                left = first[sym] + 1
                for i in range(origin, first[sym]):
                    test[s[i]] += 1
                    if test[s[i]] == dct[s[i]]:
                        del first[s[i]]
                first[sym] = idx
            else:
                test[sym] -= 1
                if sym not in first:
                    first[sym] = idx

                isSucc = True
                for u in test:
                    if test[u] != 0:
                        isSucc = False
                        break
                if isSucc:
                    res.append(left)
                    test[s[left]] += 1
                    del first[s[left]]
                    for j in range(left+1, idx+1):
                        if s[j] == s[left]:
                            first[s[left]] = j
                            break
                    left += 1
                    
        return res
```
