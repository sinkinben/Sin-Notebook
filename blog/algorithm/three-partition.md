## 3-Partition 问题

这是算法考试的最后一题，当时匆匆写了个基于 Subset Sum 的解法，也没有考虑是否可行。

问题描述如下：

> 给定 $n$ 个正整数 $a_1 \dots a_n$ ，设下标的整数集合 $V=\{1,2,3,\dots,n\}$ ,  确定是否有三个不相交的子集 $I,J,K \sub V$ ，满足 $I \cup J \cup K = V$，并且：
> $$
> \sum_{i \in I} a_i = \sum_{j \in J}a_j = \sum_{k \in K} a_k = \frac{sum}{3}
> $$
> 其中, $sum$ 是所有元素之和，要求复杂度是关于 $n$ 和 $sum$ 的多项式时间解法。



## 基于 Subset Sum 的解法

Subset Sum 问题：给定一个整数数组 `nums` 和整数 `target` ，问是否存在 `nums` 的子集，它的和为 `target` ，每个元素只能使用一次。

显然 Subset Sum 问题是背包问题的特殊情况，当背包问题中所有物品的价值等于体积，那么就是 Subset Sum 问题。

思路：

- 假设 `subsetSum(nums, target)` 能够在 `nums` 找到所有和为 `target` 的子集。
- 问题等价于找到 2 个子集 $I, J \sub V$ ，并且 $\sum{a_i} = \sum{a_j} = sum/3$ ，那么剩下的元素必然能保证 $\sum{a_k} = sum/3$ .
- 通过 Subset Sum 找到所有满足目标和为 `sum/3` 的所有子集 $\mathcal{I}$ ，即 `subsetSum(V, target = sum/3)` 。
- 对于每一个的 $I \in \mathcal{I}$ ，令 $V' = V - I$ ，执行 `subsetSum(V', target = sum/3)` ，如果返回值不为空，那么说明存在这样的 $I,J,K$ 满足 3-Partition 的条件。

Subset Sum 问题的判定形式通过动态规划是十分容易解决的，难点在于找出所有这样的子集。

建议先完成下面的「输出所有 LCS 的练习」，再进行后文的阅读。



### 输出所有 Subset Sum

定义 `dp[i, j]` 表示在 `nums[0, ..., i]` 中不超过 `j` 的最大和。

转移方程为：
$$
dp[i, j] = \left\{
\begin{aligned}
& dp[i-1, j] & \text{ if } j < a_i \\
& \max(dp[i-1, j], dp[i-1, j-a_i]+a_i), & \text{ if } j \ge a_i
\end{aligned}
\right.
$$
最后结果为 `dp[n, target] == target` .

**代码实现**

```cpp
vector<vector<int>> subsetSum(vector<int> &nums, int target)
{
    int n = nums.size();
    vector<vector<int>> dp(n + 1, vector<int>(target + 1, 0));
    for (int i = 1; i <= n; i++)
    {
        for (int j = 0; j <= target; j++)
        {
            int x = nums[i - 1];
            if (j >= x) dp[i][j] = max(dp[i - 1][j], dp[i - 1][j - x] + x);
            else dp[i][j] = dp[i - 1][j];
        }
    }

    // print all subsets
    vector<vector<int>> result;
    function<bool(int, int, vector<int>)> getSubsets = [&](int i, int j, vector<int> subset)
    {
        while (i >= 1 && j >= 0)
        {
            int x = nums[i - 1];
            if (j >= x)
            {
                int t = dp[i - 1][j - x] + x;
                if (dp[i - 1][j] > t) i--;
                else if (dp[i - 1][j] < t) j -= x, i--, subset.emplace_back(x);
                else
                {
                    getSubsets(i - 1, j, subset);
                    subset.emplace_back(x), getSubsets(i - 1, j - x, subset);
                    return true;
                }
            }
            else i--;
        }
        result.emplace_back(subset);
        return true;
    };
    if (dp[n][target] == target)
    {
        getSubsets(n, target, vector<int>{});
        return result;
    }
    return {};
}
int main()
{
    vector<int> nums = {1, 3, 7, 8, 9};
    int t = 16;
    auto result = subsetSum(nums, t);
    for (auto &v : result)
    {
        for (int x : v) cout << x << ' ';
        cout << endl;
    }
}
```





### 3-Partition

基于上述的 Subset Sum ，我们可以写出 3-Partition 的代码。

```cpp
bool threePartition(vector<int> &nums)
{
    int sum = accumulate(nums.begin(), nums.end(), 0);
    if (sum % 3 != 0) return false;
    int target = sum / 3;
    auto subsets = subsetSum(nums, target);
    for (auto &I : subsets)
    {
        vector<int> buf(nums.size() - I.size());
        // buf = nums - I
        auto itor = set_symmetric_difference(nums.begin(), nums.end(), I.begin(), I.end(), buf.begin());
        buf.resize(itor - buf.begin());
        if (subsetSum(buf, target).size() != 0) return true;
    }
    return false;
}
int main()
{
    vector<int> nums = {1, 2, 3, 4, 4, 5, 8};
    cout << threePartition(nums) << endl;
}
```

Subset Sum 的时间复杂度为 $O(nt)$，而此处 `t = sum/3` ，因此 3-Partition 的时间复杂度为 $O(kn \cdot sum)$ ，$k$ 是 集合 $I$ 的个数，显然，$k$ 有可能是指数级别的。

显然，基于上述操作，我们同样能找到所有满足条件的 $I,J,K$ .



## 动态规划

上面是比较容易想到的思路，但这里的 3-Partition 是一个判定问题，我们只需要给出 YES or NO，而不需要给出具体的 $I,J,K$ ，因此用动态规划可以使问题变得简单。

### 2-Partition

先考虑 Subset Sum 的一种特殊情况：给定 `nums` ，问是否存在一个子集 `I` , 使得 `sum(I) = sum(nums) / 2` .

其实换汤不换药。

```cpp
int twoPartition(vector<int> &nums)
{
    int sum = accumulate(nums.begin(), nums.end(), 0);
    int n = nums.size();
    if (sum % 2 != 0) return false;
    // dp[i, j] 表示前 j 个数字中，是否存在一个和为 i 的子集（允许为空集）
    vector<vector<int>> dp(sum / 2 + 1, vector<int>(n + 1, false));
    for (int i = 0; i <= n; i++) dp[0][i] = true;
    for (int i = 1; i <= sum / 2; i++) dp[i][0] = false;
    for (int i = 1; i <= sum / 2; i++)
    {
        for (int j = 1; j <= n; j++)
        {
            int x = nums[j - 1];
            if (i >= x) dp[i][j] = dp[i][j - 1] || dp[i - x][j - 1];
            else dp[i][j] = dp[i][j - 1];
        }
    }
    return dp[sum / 2][n];
}
```



### 3-Partition

定义 `dp[j, k]` 表示：**在 `nums[1, ..., n]` 中**，是否存在一个子集，使得它的和为 `j` ；同时存在另外一个不相交子集，它的和为 `k` .

注意这里的前提条件是，在 `[1, ..., n]` 这个范围，而且 `dp[j, k] = true` 当且仅当 2 个子集同时存在。

那么最后的答案是 `dp[sum / 3, sum / 3]` .

转移方程为：
$$
dp[j, k] = dp[j-a_i][k] \text{ or } dp[j][k-a_i], \text{ for any } a_i
$$
类似于自顶向下的填表顺序，转移方程可以改写为：
$$
dp[j, k] = \text{true} \quad \Rightarrow \quad dp[j+a_i, k] = dp[j, k+a_i] = \text{true}, \quad \text{for any } a_i
$$


**代码实现**

```cpp
int threePartition(vector<int> &A)
{
    int sum = accumulate(A.begin(), A.end(), 0);
    int size = A.size();
    if (sum % 3 != 0) return false;
    vector<vector<int>> dp(sum + 1, vector<int>(sum + 1, 0));
    dp[0][0] = true;
    // process the numbers one by one
    for (int i = 0; i < size; i++)
    {
        for (int j = sum; j >= 0; j--)
        {
            for (int k = sum; k >= 0; k--)
            {
                if (dp[j][k])
                {
                    dp[j + A[i]][k] = true;
                    dp[j][k + A[i]] = true;
                }
            }
        }
    }
    return dp[sum / 3][sum / 3];
}
```



## 输出所有 LCS

输出一个 LCS 可以参考：https://www.cnblogs.com/sinkinben/p/14536604.html

思路很简单，在填 `dp` 表的过程中，**转移路径**实际上就是记录了 LCS 的结果，我们只需要通过回溯法，找到所有 `(alen, blen) => (1, 1)` 的路径即可。 

![](https://gitee.com/sinkinben/pic-go/raw/master/img/20210626134952.png)

**代码实现**

```cpp
int lcs(const string &a, const string &b)
{
    int alen = a.length(), blen = b.length();
    vector<vector<int>> dp(alen + 1, vector<int>(blen + 1, 0));
    for (int i = 1; i <= alen; i++)
    {
        for (int j = 1; j <= blen; j++)
        {
            if (a[i - 1] == b[j - 1]) dp[i][j] = dp[i - 1][j - 1] + 1;
            else dp[i][j] = max(dp[i - 1][j], dp[i][j - 1]);
        }
    }
    // print all lcs
    function<void(int, int, string)> printlcs = [&](int i, int j, string str)
    {
        while (i >= 1 && j >= 1)
        {
            if (a[i - 1] == b[j - 1])
                str.push_back(a[i - 1]), i--, j--;
            else
            {
                if (dp[i - 1][j] > dp[i][j - 1]) i--;
                else if (dp[i - 1][j] < dp[i][j - 1]) j--;
                else
                {
                    printlcs(i - 1, j, str);
                    printlcs(i, j - 1, str);
                    return;
                }
            }
        }
        reverse(str.begin(), str.end());
        result.insert(str);
    };
    printlcs(alen, blen, "");
    for (auto &x : result) cout << x << endl;
    return dp[alen][blen];
}
int main()
{
    // string a = "cnblog", b = "belong";
    string a = "xyxxzxyzxy", b = "zxzyyzxxyxxz";
    // string a = "ABCBDAB", b = "BDCABA";
    cout << lcs(a, b) << endl;
}
```



