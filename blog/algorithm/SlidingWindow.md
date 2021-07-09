## 滑动窗口 (Sliding Window)

本文简单介绍一下滑动窗口算法。

## 预备知识



## 无重复字符的最长子串

题目：[3. 无重复字符的最长子串](https://leetcode-cn.com/problems/longest-substring-without-repeating-characters/)。

窗口 `[i, j]` 始终保持一个性质：如果 `s[j]` 与窗口内的某个字符重复，那么这个字符一定是 `s[i]` 。也就是说，

```cpp
class Solution {
public:
    int lengthOfLongestSubstring(string s) 
    {
        unordered_set<char> table;
        int len = s.length();
        if (len <= 1) return len;
        int i=0, j=0, maxval=1;
        while (j < len)
        {
            if (table.count(s[j]) == 0)
            {
                table.insert(s[j++]);
                maxval = max(maxval, j-i);
            }
            else
            {
                table.erase(s[i]);
                i++;
            }
        }
        return maxval;
    }
};
```

简洁版：

```cpp
class Solution {
public:
    int lengthOfLongestSubstring(string s) 
    {
        unordered_set<char> table;
        int len = s.length();
        if (len <= 1) return len;
        int i=0, j=0, maxval=1;
        while (j < len)
        {
            while (j<len && table.count(s[j]) == 0) table.insert(s[j++]);
            maxval = max(maxval, j-i);
            table.erase(s[i++]);    
        }
        return maxval;
    }
};
```

## 最大连续 1 的个数 III

题目：[1004. 最大连续1的个数 III](https://leetcode-cn.com/problems/max-consecutive-ones-iii/)。

稍稍转换一下对题目的理解：要求窗口内的 0  数目小于等于 K 。

```cpp
class Solution {
public:
    int longestOnes(vector<int>& A, int K) 
    {
        int size = A.size();
        if (size == 0) return 0;
        if (size == 1) return A[0] == 1 || K > 0;
        int zero = 0;  // 记录窗口内 0 的数目
        int i = 0, j = 0, maxval = 0;
        while (j < size)
        {
            if (A[j] == 0) zero++;
            while (zero > K)
            {
                if (A[i] == 0) zero--;
                i++;
            }
            j++;
            maxval = max(maxval, j-i);
        }
        return maxval;
    }
};
```

