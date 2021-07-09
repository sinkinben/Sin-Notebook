## [leetcode] 剑指 Offer 专题（七）

《剑指 Offer》专题第七部。

## 66 构建乘积数组

题目：[剑指 Offer 66. 构建乘积数组](https://leetcode-cn.com/problems/gou-jian-cheng-ji-shu-zu-lcof/)。

定义:
$$
\begin{aligned}
V_1[i] = \prod_{k=0}^{i-1} a[k] = a[0] \times a[1] \times \dots \times a[i-1], \quad&V_1[0] = 1 \\
V_2[i] = \prod_{k=i+1}^{n-1} a[k] = a[i+1] \times \dots \times a[n-1], \quad&V_2[n-1] = 1
\end{aligned}
$$
那么有：
$$
B[i] = V_1[i] \times V_2[i], 0 \le i \le n-1
$$
代码实现：

```cpp
class Solution {
public:
    vector<int> constructArr(vector<int>& a) 
    {
        int n = a.size();
        vector<int> v1(n, 1);
        int p = 1;
        for (int i=0; i<n; i++) v1[i] = p, p *= a[i];
        p = 1;
        vector<int> v2(n, 1);
        for (int i=n-1; i>=0; i--) v2[i] = p, p *= a[i];
        for (int i=0;i<n;i++) v1[i] *= v2[i];
        return v1;
    }
};
```

优化空间 `v2` ：

```cpp
class Solution {
public:
    vector<int> constructArr(vector<int>& a) 
    {
        int n = a.size();
        vector<int> v1(n, 1);
        int p = 1;
        for (int i=0; i<n; i++) v1[i] = p, p *= a[i];
        p = 1;
        for (int i=n-1; i>=0; i--) v1[i] *= p, p *= a[i];
        return v1;
    }
};
```

## 67 字符串转整数

题目：[剑指 Offer 67. 把字符串转换成整数](https://leetcode-cn.com/problems/ba-zi-fu-chuan-zhuan-huan-cheng-zheng-shu-lcof/)。

思路很清晰，请看注释。

```cpp
class Solution {
public:
    const uint32_t INTMIN = (1<<31);
    const uint32_t INTMAX = ~INTMIN;
    int strToInt(string str) 
    {
        auto isDigit = [](char c) { return '0' <= c && c <= '9';};
        auto isSign = [](char c) { return c == '+' || c == '-';};
        int len = str.length(), i = 0;
        // 忽律空格
        while (i < len && str[i] == ' ') i++;
        // 第一个非空字符，如果不是 {+, -, 0-9} 那么返回 0 
        if (i == len || (!isSign(str[i]) && !isDigit(str[i]))) return 0;
        // 第一个非空字符是否代表负数
        bool neg = false;
        if (isSign(str[i])) neg = (str[i] == '-'), i++;
        // 求出数字部分的绝对值
        uint64_t val = 0;
        while (i < len && isDigit(str[i]))
        {
            val = val*10 + (str[i]-'0'), i++;
            // 如果超过 [-intmax, +intmax]
            if (val > INTMAX) return neg ? INTMIN: INTMAX;
        }
        return neg ? -val: val;     
    }
};
```

## 68-I 二叉搜索树的最近公共祖先

题目：[剑指 Offer 68 - I. 二叉搜索树的最近公共祖先](https://leetcode-cn.com/problems/er-cha-sou-suo-shu-de-zui-jin-gong-gong-zu-xian-lcof/)。

`p` 和 `q` 要么都在左子树，要么都在右子树。如果不是，说明当前的子树的 `root` 就是最近公共祖先。

```cpp
class Solution {
public:
    TreeNode* lowestCommonAncestor(TreeNode* root, TreeNode* p, TreeNode* q) 
    {
        auto r = root;
        while (r)
        {
            if (p->val < r->val && q->val < r->val) r = r->left;
            else if (p->val > r->val && q->val > r->val) r = r->right;
            else break;
        }
        return r;
    }
};
```

## 68-II 二叉树的最近公共祖先

题目：[剑指 Offer 68 - II. 二叉树的最近公共祖先](https://leetcode-cn.com/problems/er-cha-shu-de-zui-jin-gong-gong-zu-xian-lcof/)。

递归。基于后序遍历，`lca(r, p, q)` 表示是否在子树 `r` 中找到 `p, q` 中的任意一个节点。  

以下图为例：

<img src="https://gitee.com/sinkinben/pic-go/raw/master/img/20201105190630.png"/>

当输入 `p=7, q=4` 时，后序遍历到 7 或 4 时，返回 `left = 7, right = 4` 给 2 ，这时候 2 就会把自己返回给 5 .

在遍历 5 的递归层中，`left = nullptr, right = 2` ，把 `right = 2` 返回给 3 。

同理，在遍历 3 的第一层递归中，`left = 2, right = nullptr` ，那么就会把 `left = 2` 返回给 `lowestCommonAncestor` .

```cpp
class Solution {
public:
    TreeNode* lowestCommonAncestor(TreeNode* root, TreeNode* p, TreeNode* q) {
        return lca(root, p, q);
    }
    TreeNode *lca(TreeNode *r, TreeNode *p, TreeNode *q)
    {
        if (r == NULL || r == p || r == q) return r;
        auto left = lca(r->left, p, q), right = lca(r->right, p, q);
        if (left && right) return r;
        return left ? left : right;
    }
};
```

## n 个骰子的点数

题目：[面试题60. n个骰子的点数](https://leetcode-cn.com/problems/nge-tou-zi-de-dian-shu-lcof/)。

动态规划。

设 $P(k)$ 是骰子点数和为 $k$ 的概率，那么 $P(k)$ 等于 $k$ 出现的次数除以总事件数目。 

状态定义：$dp[i][j]$ 表示有 $i$ 个骰子时，点数和为 $j$ 的事件数。显然有，$i \le j \le 6i$ .

转移方程：
$$
dp[i][j] = \sum_{k=1}^{6} dp[i-1][j-k], \quad if \quad (i-1) \le (j-k) \le 6(i-1)
$$
解析：$dp[i][j]$ 表示需要用 $i$ 个骰子，掷出点数 $j$ 。 考虑第 $i$ 个骰子的点数可以是 $[1, 6]$ 的任意一种，因此 $dp[i][j] = dp[i-1][j-1] + \dots + dp[i-1][j-6]$ .

边界条件：$dp[1][1, \dots, 6] = 1$ .

虽然二维数组 `dp` 是 `n x 6n` 的，但是实际上并不是每个元素都有效，这是由状态方程的定义决定的。前 3 行有效的元素的下标如下：

```
i=1 [1, 2, 3, 4, 5, 6]
i=2 [2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12]
i=3 [3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18]
```

求出 `dp[n][n, ..., 6*n]` 后，事件总数就是 `sum(dp[n][n, ..., 6*n])` . 

```cpp
class Solution {
public:
    vector<double> dicesProbability(int n) 
    {
        if (n == 0) return {};
        int rows = n, cols = 6*n;
        vector<vector<int>> dp(rows+1, vector<int>(cols+1, 0));
        for (int j=1;j<=6;j++) dp[1][j] = 1;
        for (int i=2;i<=rows;i++)
        {
            for (int j=i;j<=6*i;j++)
            {
                for (int k=1; k<=6; k++)
                    if(j-k >= i-1)
                        dp[i][j] += dp[i-1][j-k];
            }
        }
        vector<double> res;
        int total = 0;
        for (int i=n; i<=cols; i++) total+=dp[n][i];
        for (int i=n; i<=cols; i++)
        {
            double x = dp[n][i];
            res.push_back(x/total);
        }
        return res;
    }
};
```

空间优化：

```cpp
class Solution {
public:
    vector<double> dicesProbability(int n) 
    {
        int rows=n, cols=6*n;
        vector<int> dp(cols+1, 0);
        for (int i=1; i<=6; i++) dp[i] = 1;
        for (int i=2; i<=rows; i++)
        {
            for (int j=cols; j>=i; j--)
            {
                int t = 0;
                for (int k=1; k<=6; k++)
                    if (j-k >= i-1) t += dp[j-k];
                dp[j] = t;
            }
        }
        vector<double> res;
        int total = 0;
        for (int i=n; i<=cols; i++) total+=dp[i];
        for (int i=n; i<=cols; i++) res.push_back(((double)dp[i]) / total);
        return res;
    }
};
```

