## 0101 判定字符是否唯一

题目：[面试题 01.01. 判定字符是否唯一](https://leetcode-cn.com/problems/is-unique-lcci/)。

+ 哈希

```cpp
class Solution {
public:
    bool isUnique(string astr) {
        bool table[256] = {0};
        for (char x: astr)
        {
            if (table[x]) return false;
            table[x] = true;
        }
        return true;
    }
};
```

+ 哈希：空间优化

ASCII 字符的取值范围是 `[0, 127]` 。因此，只需要 2 个 `uint64_t` 作为哈希表。

```cpp
class Solution {
public:
    bool isUnique(string astr) {
        uint64_t low = 0, high = 0;
        for (uint8_t x: astr)
        {
            if (x >= 64)
            {
                x -= 64;
                if ((high >> x) & 1) return false;
                else high |= ((uint64_t)1 << x);
            }
            else
            {
                if ((low >> x) & 1) return false;
                else low |= ((uint64_t)1 << x);
            }
        }
        return true;
    }
};
```



## 0102 判定是否互为字符重排  

题目：[判定是否互为字符重排](https://leetcode-cn.com/problems/check-permutation-lcci) 。

哈希哈希~

不过要注意，这里的字符串可能包含有重复字符。

```cpp
class Solution {
public:
    bool CheckPermutation(string s1, string s2) {
        return hashing(s1) == hashing(s2);
    }
    vector<int> hashing(string &s)
    {
        vector<int> v(128, 0);
        for (uint8_t x: s) v[x]++;
        return v;
    }
};
```

## 0103 URL化

题目：[面试题 01.03. URL化](https://leetcode-cn.com/problems/string-to-url-lcci/) 。

```cpp
class Solution {
public:
    string replaceSpaces(string s, int length) {
        int n = 0;
        for (char x: s) n += (x != ' ');
        int len = (length - n) * 3 + n;
        string t;
        for (char x: s)
        {
            if (x == ' ')
                t += "%20";
            else 
                t.push_back(x);
            if (t.length() == len) break;
        }
        return t;
    }
};
```



## 0104 回文排列

题目：[面试题 01.04. 回文排列](https://leetcode-cn.com/problems/palindrome-permutation-lcci/) 。

回文排列必须满足下列条件：出现次数为奇数的字符最多只有一个。

方法：通过哈希计数，然后检查 `table` 中是否只有零个或一个奇数。

```cpp
class Solution {
public:
    bool canPermutePalindrome(string s) {
        vector<int> table(128, 0);
        for (char x: s) table[x]++;
        int idx = 0;
        while (idx < 128 && (table[idx] % 2 == 0)) idx++;
        if (idx == 128) return true;
        for (int i=idx+1; i<128; i++)
            if (table[i] % 2 == 1) return false;
        return true;
    }
};
```



## 0105 一次编辑

题目：[面试题 01.05. 一次编辑](https://leetcode-cn.com/problems/one-away-lcci/)。

「编辑距离」的简化版本。

### 动态规划

对于字符串 `A` 和 `B` , 以下操作是等价的：

+ 在 `A` 中替换一个字符，与在 `B` 中替换一个字符
+ 在 `B` 中增加一个字符，与在 `A` 中删除一个字符
+ 在 `A` 中增加一个字符，与在 `B` 中删除一个字符

状态定义：`dp[i,j]` 表示 `A[0, ..., i-1]` 与 `B[0, ..., j-1]` 的编辑距离。

转移方程：

```python
if A[i-1] == B[j-1]:
    dp[i, j] = min(dp[i-1, j-1], dp[i-1, j]+1, dp[i, j-1]+1)
else:
    dp[i, j] = min(dp[i-1, j], dp[i, j-1], dp[i-1, j-1]) + 1
```

代码：

```cpp
class Solution {
public:
    bool oneEditAway(string first, string second) {
        int len1 = first.length(), len2 = second.length();
        vector<vector<int>> dp(len1+1, vector<int>(len2+1, 0));
        for (int i=0; i<len1; i++) dp[i][0] = i;
        for (int j=0; j<len2; j++) dp[0][j] = j;
        for (int i=1; i<=len1; i++)
        {
            for (int j=1; j<=len2; j++)
            {
                if (first[i-1] == second[j-1])
                    dp[i][j] = min(dp[i-1][j-1], min(dp[i][j-1], dp[i-1][j])+1);
                else
                    dp[i][j] = min(dp[i-1][j-1], min(dp[i-1][j], dp[i][j-1]))+1;
            }
        }
        return dp[len1][len2] <= 1;
    }
};
```

### 分类讨论

要求最多只能操作一次，那么说明两个字符串的长度之差最多为 1 。那么：

+ 长度之差为 0 ，有且只有一个字符不同。
+ 长度之差为 1 ，长串只比短串多一个字符。

```cpp
class Solution {
public:
    bool oneEditAway(string first, string second) {
        if (first.length() < second.length()) swap(first, second);
        int len1 = first.length(), len2 = second.length();
        if (len1 - len2 > 1) return false;
        if (len1 == len2)
        {
            int i = 0, k = 0;
            for (i=0; i<len1; i++)
            {
                k += (first[i] != second[i]);
                if (k > 1) return false;
            }
            return true;
        }
        else
        {
            int i1 = 0, j1 = 0, i2 = len1 - 1, j2 = len2 - 1;
            while (j1 < len2 && first[i1] == second[j1]) i1++, j1++;
            // e.g. "plea", "ple"
            if (j1 == len2) return true;
            while (j2 >= 0 && first[i2] ==  second[j2]) i2--, j2--;
            // e.g. "aple", "ple"
            if (j2 == -1) return true;
            // e.g. "pale", "ple"
            return i1 == i2 && j2 == j1-1;
            // return 'i1 == i2' is also ok.
        }
    }
};
```

## 0106 字符串压缩

题目：[面试题 01.06. 字符串压缩](https://leetcode-cn.com/problems/compress-string-lcci/) 。

模拟就够了。

```cpp
class Solution {
public:
    string compressString(string s) {
        int len = s.length();
        if (len <= 2) return s;
        string buf;
        int i = 0, j = 0;
        while (i < len)
        {
            while (j < len && s[j] == s[i]) j++;
            buf.push_back(s[i]);
            buf += std::to_string(j-i);
            i = j;
        }
        return buf.length() < len ? buf : s;
    }
};
```

## 0107 旋转矩阵

题目：[面试题 01.07. 旋转矩阵](https://leetcode-cn.com/problems/rotate-matrix-lcci/)。

要求就地旋转。「由题意可得」，先沿着对角线交换，然后对每一行 `reverse` 即可。

```cpp
class Solution {
public:
    void rotate(vector<vector<int>>& matrix) {
        int n = matrix.size();
        if (n <= 1) return;
        for (int i=1; i<n; i++)
            for (int j=0; j<i; j++)
                swap(matrix[i][j], matrix[j][i]);
        for (auto &v: matrix) reverse(v.begin(), v.end());
    }
};
```

不使用 `reverse` :

```cpp
class Solution {
public:
    void rotate(vector<vector<int>>& matrix) {
        int n = matrix.size();
        if (n <= 1) return;
        for (int i=1; i<n; i++)
            for (int j=0; j<i; j++)
                swap(matrix[i][j], matrix[j][i]);
        for (auto &v: matrix)
        {
            int size = v.size(), k = v.size() / 2;
            for (int i=0; i<k; i++) swap(v[i], v[size-i-1]);
        }
    }
};
```

## 0108 零矩阵

题目：[面试题 01.08. 零矩阵](https://leetcode-cn.com/problems/zero-matrix-lcci/)。

问就是暴力。

```cpp
class Solution {
public:
    void setZeroes(vector<vector<int>>& matrix) {
        if (matrix.size() == 0 || matrix[0].size() == 0) return;
        int m = matrix.size(), n = matrix[0].size();
        unordered_set<int> s1, s2;
        for (int i=0; i<m; i++)
            for (int j=0; j<n; j++)
                if (matrix[i][j] == 0)
                    s1.insert(i), s2.insert(j);
        for (int x: s1)
            for (int j=0; j<n; j++) matrix[x][j] = 0;
        for (int x: s2)
            for (int i=0; i<m; i++) matrix[i][x] = 0;
    }
};
```

## 0109 字符串轮转

题目：[面试题 01.09. 字符串轮转](https://leetcode-cn.com/problems/string-rotation-lcci/) 。

### 暴力美学

一个字符串旋转 `n` 次后会回到原来的状态，暴力枚举。

```cpp
class Solution {
public:
    bool isFlipedString(string s1, string s2) {
        int len1 = s1.length(), len2 = s2.length();
        if (len1 != len2) return false;
        if (s1 == s2) return true;
        for (int i=0; i<len1; i++)
        {
            std::rotate(s1.begin(), s1.begin()+1, s1.end());
            if (s1 == s2) return true;
        }
        return false;
    }
};
```

### 奇技淫巧

讨论区都是人才。

```cpp
class Solution {
public:
    bool isFlipedString(string s1, string s2) {
        if (s1.length() != s2.length()) return false;
        s2 = s2 + s2;
        return s2.find(s1) != string::npos;
    }
};
```

压缩：

```cpp
return s1.length() == s2.length() && (s2+s2).find(s1) != string:npos;
```

