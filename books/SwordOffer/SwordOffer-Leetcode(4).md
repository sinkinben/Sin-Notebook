## [leetcode] 剑指 Offer 专题（四）

《剑指 Offer》专题第四部。

## 36 二叉搜索树和双向链表

题目：[剑指 Offer 36. 二叉搜索树与双向链表](https://leetcode-cn.com/problems/er-cha-sou-suo-shu-yu-shuang-xiang-lian-biao-lcof/)。

中序遍历。时间复杂度 $O(N)$，空间复杂度 $O(N)$ .

```cpp
class Solution {
public:
    vector<Node*> res;
    Node* treeToDoublyList(Node* root) {
        if (root == nullptr) return nullptr;
        inorder(root);
        int len = res.size();
        for (int i=0; i<len; i++)
        {
            int next = (i+1)%len;
            res[i]->right = res[next];
            res[next]->left = res[i];
        }
        return res[0];
    }
    void inorder(Node *p)
    {
        if (p == nullptr) return;
        inorder(p->left);
        res.push_back(p);
        inorder(p->right);
    }
};
```

## 37 序列化二叉树

题目：[剑指 Offer 37. 序列化二叉树](https://leetcode-cn.com/problems/xu-lie-hua-er-cha-shu-lcof/).

### 层次遍历

层次遍历实现。More details see：[二叉树的序列化与反序列化](https://www.cnblogs.com/sinkinben/p/13668996.html)。

```cpp
class Codec {
public:
    const string sep = ",";
    const string nil = "null";
    // Encodes a tree to a single string.
    string serialize(TreeNode* root) {
        vector<string> res;
        if (root != nullptr)
        {
            queue<TreeNode*> q;
            q.push(root);
            while (!q.empty())
            {
                auto p = q.front();
                q.pop();
                if (p == nullptr)
                    res.push_back(nil);
                else
                {
                    res.push_back(to_string(p->val));
                    q.push(p->left), q.push(p->right);
                }
            }
        }
        while (!res.empty() && res.back() == nil) res.pop_back();
        string ans = "";
        for (auto &x: res) ans += x + sep;
        if (!ans.empty()) ans.pop_back();
        return "[" + ans + "]";
    }

    inline TreeNode* generateNode(const string &s)
    {
        return s == nil ? nullptr : new TreeNode(stoi(s));
    }

    vector<string> split(string &data, const string &sep)
    {
        size_t l = 0;
        size_t r = data.find(sep, l);
        vector<string> res;
        while (r != string::npos)
        {
            res.push_back(data.substr(l, r - l));
            l = r + sep.length();
            r = data.find(sep, l);
        }
        res.push_back(data.substr(l));
        return res;
    }

    // Decodes your encoded data to tree.
    TreeNode* deserialize(string data) 
    {
        data = data.substr(1, data.length() - 2);
        if (data.length() == 0) return nullptr;

        auto res = split(data, sep);
        int idx = 0, len = res.size();
        auto root = generateNode(res[idx++]);
        queue<TreeNode*> q;
        q.push(root);
        while (!q.empty())
        {
            auto p = q.front();
            q.pop();
            if (p == nullptr) continue;
            if (idx < len) p->left = generateNode(res[idx++]), q.push(p->left);
            if (idx < len) p->right = generateNode(res[idx++]), q.push(p->right);
            if (idx >= len) break;
        }
        return root;
    }
};
```

### 先序遍历

递归实现。

💬 感觉用 C++ 做，题目不难，但字符串处理难。

```cpp
class Codec {
public:
    const string sep = ",";
    const string nil = "null";
    // Encodes a tree to a single string.
    string serialize(TreeNode* root) {
        string s = preorderSerialize(root);
        if (s.back() == sep[0]) s.pop_back();
        return s;
    }
    string preorderSerialize(TreeNode *p)
    {
        if (p == nullptr) return nil + sep;
        string s = to_string(p->val) + sep;
        s += preorderSerialize(p->left);
        s += preorderSerialize(p->right);
        return s;
    }
    vector<string> split(string &s, const string &sep)
    {
        size_t l=0, r=s.find(sep, 0);
        vector<string> v;
        while (r != string::npos)
        {
            v.push_back(s.substr(l, r-l));
            l = r + sep.length();
            r = s.find(sep, l);
        }
        v.push_back(s.substr(l));
        return v;
    }
    inline TreeNode *generate(string &s)
    {
        return s == nil ? nullptr : new TreeNode(stoi(s));
    }

    // Decodes your encoded data to tree.
    TreeNode* deserialize(string data) {
        auto v = split(data, sep);
        int idx = 0;
        return preorderDeserialize(v, idx);
    }

    TreeNode* preorderDeserialize(vector<string> &v, int &idx)
    {
        if (idx >= v.size()) return nullptr;
        auto p = generate(v[idx++]);
        if (p)
        {
            p->left = preorderDeserialize(v, idx);
            p->right = preorderDeserialize(v, idx);
        }
        return p;
    }
};

```



## 38 字符串的排列

题目：[剑指 Offer 38. 字符串的排列](https://leetcode-cn.com/problems/zi-fu-chuan-de-pai-lie-lcof/)。

### 使用库函数

```cpp
class Solution {
public:
    vector<string> permutation(string s) {
        // sort is necessary when using next_permutation
        sort(s.begin(), s.end());
        vector<string> res;
        res.push_back(s);
        while (next_permutation(s.begin(), s.end()))
            res.push_back(s);
        return res;
    }
};
```

### 回溯法

利用一个 `set` 去重，即处理输入 `s = "aab"` 这种情况。

```cpp
class Solution
{
public:
    int n = 0;
    unordered_set<string> ans;
    string buf;
    vector<string> permutation(string s)
    {
        buf = s;
        n = s.length();
        vector<int> v(n, 0);
        helper(v, 0);
        return vector<string>(ans.begin(), ans.end());
    }
    string generate(vector<int> &v)
    {
        string s = "";
        for (int x : v) s.append(1, buf[x]);
        return s;
    }
    inline bool check(vector<int> &v, int idx)
    {
        for (int i = 0; i < idx; i++)
            if (v[idx] == v[i])
                return false;
        return true;
    }
    void helper(vector<int> &v, int pos)
    {
        for (int i = 0; i < n; i++)
        {
            v[pos] = i;
            if (check(v, pos))
            {
                if (pos == n - 1)
                    ans.insert(generate(v));
                else
                    helper(v, pos + 1);
            }
        }
    }
};
```

上面 2 种方法，本质上还是对全排列进行枚举，效率是有点低的。

### 交换法

书本解法。

```cpp
#include <algorithm>
class Solution {
public:
    unordered_set<string> ans;
    vector<string> permutation(string s) 
    {
        helper(s, 0);
        return vector<string>(ans.begin(), ans.end());
    }
    void helper(string &s, int idx)
    {
        int len = s.length();
        if (idx == len)
        {
            ans.insert(s);
            return;
        }
        for (int i=idx; i<len; i++)
        {
            swap(s[i], s[idx]);
            helper(s, idx+1);
            swap(s[i], s[idx]);
        }
    }
};
```

## 39 数组中出现次数超过一半的数字

题目：[剑指 Offer 39. 数组中出现次数超过一半的数字](https://leetcode-cn.com/problems/shu-zu-zhong-chu-xian-ci-shu-chao-guo-yi-ban-de-shu-zi-lcof/)。

这 TM 居然是简单题（用哈希表计数或者排序的话确实简单，但没意思）。

### 快排思想

根据排序后的结果，显然 `arr[n/2]` 就是次数超过一半的数字。但是我们不需要严格的排序，可以根据快排的 `partition` 操作，将数组分为 `arr[..., mid - 1] < arr[mid] < arr[mid+1, ...]` . 这样不需要完全排序，但是依然能保证中间元素是出现次数超过一半的那个数字。

```cpp
class Solution {
public:
    int majorityElement(vector<int>& arr) 
    {
        int idx = partition(arr, 0, arr.size() - 1);
        int mid = arr.size() / 2;
        int start = 0, end = arr.size()-1;
        while (idx != mid)
        {
            if (idx <= mid) start = idx + 1;
            else end = idx - 1;
            idx = partition(arr, start, end);
        }
        return arr[mid];
    }

    int partition(vector<int> &v, int p, int r)
    {
        int x = v[r];
        int i = p-1;
        for (int j=p; j<r; j++)
        {
            if (v[j] < x)
                i++, swap(v[i], v[j]);
        }
        swap(v[i+1], v[r]);
        return i+1;
    }
};
```



### 摩尔投票算法

这是一个叫「摩尔投票法」的算法，首先假定一个数为 `candidate`，扫描整个数组，如果 `x == candidate` 那么 `times++` ， 否则 `times--` 。如果回到 0 ，说明要重新猜一个数为 `candidate` 。

评论区 [@ajslpzcd](https://leetcode-cn.com/u/ajslpzcd/) 有一个十分形象的解析：

> 可以理解成混战极限一换一，不同的两者一旦遇见就同归于尽，最后活下来的值都是相同的，即要求的结果。

代码实现：

```cpp
class Solution {
public:
    int majorityElement(vector<int>& nums) 
    {
        int candidate;
        int times = 0;
        for (int x: nums)
        {
            if (times == 0)
                candidate = x;
            times += ((candidate == x) ? 1 : (-1));
        }
        return candidate;
    }
};
```

## 40 最小的 k 个数

题目：[剑指 Offer 40. 最小的k个数](https://leetcode-cn.com/problems/zui-xiao-de-kge-shu-lcof/)。

### 排序法

别问，问就是排序。时间复杂度 $O(n\log n)$ .

```cpp
class Solution {
public:
    vector<int> getLeastNumbers(vector<int>& arr, int k) {
        vector<int> v;
        sort(arr.begin(), arr.end());
        for (int i=0; i<k; i++)
            v.push_back(arr[i]);
        return v;
    }
};
```

### 快排思想

> 💬 顶不住了，下次继续吧，十分讨厌玩数字类的题目，爬爬爬。想去念诗了 0w0!  2020/10/20, 16:17

根据快排的思想，当主元位置为 `k` 时，那么 `arr[0, ..., k-1]` 就是数组中最小的 k 个数字。时间复杂度 $O(n)$ .

```cpp
class Solution {
public:
    vector<int> getLeastNumbers(vector<int>& arr, int k) 
    {
        if (k == 0) return vector<int>();
        if (k >= arr.size()) return arr;
        int len = arr.size();
        int l = 0, r = len-1;
        int idx = partition(arr, l, r);
        while (idx != k)
        {
            if (idx > k)
                r = idx-1;
            else 
                l = idx+1;
            idx = partition(arr, l, r);
        }
        vector<int> ans(k, 0);
        for (int i=0; i<k; i++)
            ans[i] = arr[i];
        return ans;
    }

    int partition(vector<int> &v, int p, int r)
    {
        int x = v[r];
        int i = p-1;
        for (int j=p; j<r; j++)
            if (v[j] <= x)
                i++, swap(v[i], v[j]);
        swap(v[i+1], v[r]);
        return i+1;
    }
};
```

### 堆

优先队列 `priority_queue` 是通过大顶堆实现的，我们可以把它当作堆来使用。

首先，把 k 个数字放进堆中，扫描剩余的每一个元素 `x` ，如果小于堆中的最大值，即 `x < q.top()` ，说明 `x` 才是最小的 k 个数字之一。

```cpp
class Solution {
public:
    vector<int> getLeastNumbers(vector<int>& arr, int k) 
    {
        if (k == 0) return vector<int>();
        if (k >= arr.size()) return arr;
        priority_queue<int> q;
        int len = arr.size();
        for (int i=0; i<k; i++)
            q.push(arr[i]);
        for (int i=k; i<len; i++)
        {
            if (q.top() > arr[i])
            {
                q.pop();
                q.push(arr[i]);
            }
        }
        vector<int> v(k, 0);
        for (int i=0; i<k; i++)
            v[i] = q.top(), q.pop();
        return v;
    }
};
```

维护一个堆的操作的时间复杂度为 $O(\log k)$，所以总的时间复杂度为 $O(n \log k)$ .

## 41 数据流的中位数

题目：[剑指 Offer 41. 数据流中的中位数](https://leetcode-cn.com/problems/shu-ju-liu-zhong-de-zhong-wei-shu-lcof/)。

维护 2 个堆，并且满足以下性质：

+ 性质 1：大顶堆 `small` 和小顶堆 `large` ，其中 `small` 的最大值小于 `large` 的最小值，即 `small[0] <= large[0]` 恒成立。

+ 性质 2：要求 2 个堆的大小之差只能为 1 或者 0 。如果插入数字的总数为奇数（即该数字的插入序号为奇数），那么向 `small` 插入。否则向 `large` 插入。

这么做可以保证：`small[0]` 大于等于一半的数字，`large[0]` 小于等于一半的数字。

显然，`findMedian` 需要根据插入数字总数的奇偶性来判断，如果是奇数，那么中位数是 `small[0]`，如果是偶数，那么中位数是 `(double)(small[0] + large[0]) / 2`。

```cpp
class MedianFinder
{
public:
    /** initialize your data structure here. */
    int total = 0;
    vector<int> small, large;
    inline void pushHeap(int x, vector<int> &v, bool bigHeap)
    {
        v.push_back(x);
        if (bigHeap)
            push_heap(v.begin(), v.end(), greater<int>());
        else
            push_heap(v.begin(), v.end(), less<int>());
    }

    inline int popHeap(vector<int> &v, bool bigHeap)
    {
        int x = v[0];
        if (bigHeap)
            pop_heap(v.begin(), v.end(), greater<int>());
        else
            pop_heap(v.begin(), v.end(), less<int>());
        v.pop_back();
        return x;
    }
    void addNum(int x)
    {
        total++;
        if (total % 2)
        {
            // insert into the small set
            // 这里需要考虑到：如果插入的 x 比 large[0] 要大，这时候不能直接插入 small，否则破坏性质1
            // 处理的办法：先插入到 large，再从 large 中 pop 一个最小的出来，插入到 small
            pushHeap(x, large, true);
            x = popHeap(large, true);
            pushHeap(x, small, false);
        }
        else
        {
            pushHeap(x, small, false);
            x = popHeap(small, false);
            pushHeap(x, large, true);
        }
    }
    double findMedian()
    {
        if (total == 0)
            return -1;
        if (total % 2)
            return small[0];
        else
            return ((double)small[0] + (double)large[0]) / 2;
    }
};
```

## 42 连续子数组的最大和

题目：[剑指 Offer 42. 连续子数组的最大和](https://leetcode-cn.com/problems/lian-xu-zi-shu-zu-de-zui-da-he-lcof/)。

可以用前缀和结构，然后枚举每一个区间 `[i,j]`，时间复杂度是 $O(n^2)$ .

但这是 DP 水题。

```cpp
class Solution {
public:
    int maxSubArray(vector<int>& nums) 
    {
        vector<int> dp(nums);
        int len = nums.size();
        int maxval = dp[0];
        for (int i=1; i<len; i++)
            dp[i] = max(nums[i], dp[i-1]+nums[i]), maxval = max(maxval, dp[i]);
        return maxval;
    }
};
/**
 *  dp[i]: [0, ..., i] 范围内，选中 a[i] 的最大连续子数组
 *  dp[i] = max(a[i], dp[i-1]+a[i])
 */
```

空间优化：

```cpp
class Solution {
public:
    int maxSubArray(vector<int>& nums) 
    {
        int dp = nums[0];
        int len = nums.size();
        int maxval = dp;
        for (int i=1; i<len; i++)
            dp = max(nums[i], dp+nums[i]), maxval = max(maxval, dp);
        return maxval;
    }
};
```

## 43 🎈整数中 1 出现的次数

题目：[剑指 Offer 43. 1～n 整数中 1 出现的次数](https://leetcode-cn.com/problems/1nzheng-shu-zhong-1chu-xian-de-ci-shu-lcof/)。

对得起 Hard 这个标签 🏷️。

### 动态规划

定义 `dp[i]` 是整数 `i` 的十进制包含的 1 的个数。

转移方程：`dp[i] = dp[i/10] + (i%10 == 1)` .

时间复杂度为 $O(n)$， 但是超时了 😓 。

```cpp
class Solution {
public:
    int countDigitOne(int n) 
    {
        vector<int> dp(n+1, 0);
        for (int i=1; i<=n; i++) dp[i] = dp[i/10] + (i%10 == 1);
        int sum = 0;
        for (int x: dp) sum+=x;
        return sum;
    }
};
```

### 数学分析

先看依次看下面 2 篇题解：

+ 墙外题解：[简洁版](https://leetcode.com/problems/number-of-digit-one/discuss/64382/JavaPython-one-pass-solution-easy-to-understand).
+ 墙内题解：[稍微严谨的数学证明](https://leetcode-cn.com/problems/1nzheng-shu-zhong-1chu-xian-de-ci-shu-lcof/solution/mian-shi-ti-43-1n-zheng-shu-zhong-1-chu-xian-de-2/).

假设输入的 `n` 是一个 `k` 位数，那么把所有数字都看作是具有 `k` 位的数字，不足 `k` 位补上前缀零。那么，我们要求解的是第 `0` 位到第 `k-1` 位上的 1 的个数之和。

以 `n` 是一个 7 位数来举例，设：

```cpp
n = xyzdabc
```

在第 3 位上，1 的个数有：

```cpp
(1) xyz * 1000                     if d == 0
(2) xyz * 1000 + abc + 1           if d == 1
(3) xyz * 1000 + 1000              if d > 1
```

下面是证明过程（所有图示来源于上述 leetcode 讨论区的题解）。

设 $str(n) = str(high) + str(cur) + str(low)$ , 我们把 $n$ 分为 3 部分，其中 $cur$ 是当前求解的某一位，且有 $str(cur) = 1$.

|                          $cur == 0$                          |                          $cur == 1$                          |                         $cur \ge 2$                          |
| :----------------------------------------------------------: | :----------------------------------------------------------: | :----------------------------------------------------------: |
| ![Figure-1](https://gitee.com/sinkinben/pic-go/raw/master/img/20201030211235.png) | ![Figure-2](https://gitee.com/sinkinben/pic-go/raw/master/img/20201030211332.png) | ![Figure-3](https://gitee.com/sinkinben/pic-go/raw/master/img/20201030211348.png) |

如上图 1 所示，如果 `cur == 0` , 那么在该位上的 1 的个数，将由 `high` 和 `low` 可变化的次数决定（简单排列组合问题）。显然在这里，可变化的范围是：高位取 `[0, high-1]`，低位取 `[0, 9]`，组合结果为 `high * 10 = 23 * 10 = 230` .

如上图 2 所示，如果 `cur == 1` , 那么高位可取范围是 `[0, high]`, 在 `[0, 2310)` 区间上，低位可取范围是 `[0, 9]`，在 `[2310, 2314]` 可取的范围是 `[0, 4]`。因此结果是 `high * 10 + low = 235` .

图 3 就不解释了吧。

综上所述，设 $f(cur)$ 是 $cur$ 位置上 1 的个数：
$$
f(cur) = 
\left\{
\begin{aligned}
&high \times digit \quad &if \quad cur=0 \\
&high \times digit + low + 1 \quad &if \quad cur=1 \\
&(high+1) \times digit \quad &if \quad cur\ge2
\end{aligned}
\right.
$$
最终答案为 $\sum_{i=0}^{k-1}f(i)$ .

时间复杂度 $O(\log n)$ .

代码实现：

```cpp
class Solution {
public:
    int countDigitOne(int n) 
    {
        int low = 0, high = n/10;
        int cur = n % 10;
        uint64_t sum = 0, digit = 1;
        // assume that:
        // n = xyzabc
        // d = 100000 is the last loop
        while (digit <= n)
        {
            if (cur == 0) sum += high * digit;
            else if (cur == 1) sum += (high * digit + low + 1);
            else sum += (high + 1) * digit;
            high /= 10, low = cur * digit + low;
            digit *= 10;
            cur = (n / digit) % 10;
            // cur = origin_high % 10;
        }
        return sum;
    }
};
```



## 44 🎈数字序列中的某一位数字

题目：[剑指 Offer 44. 数字序列中某一位的数字](https://leetcode-cn.com/problems/shu-zi-xu-lie-zhong-mou-yi-wei-de-shu-zi-lcof/)。

这是一道 hard 题目 🍃。

### 直接模拟

超时了。`cnt` 用于计算当前数字，`t` 用于模拟当前数到到位置，如果 `t` 为 0 ，说明 `cnt` 的第一位数字就是所求的答案。

```cpp
class Solution {
public:
    int findNthDigit(int n) 
    {
        if (n == 0) return 0;
        int cnt = 0;
        int t = n-1;
        while (t > 0)
        {
            cnt++;
            t -= ((int)log10(cnt) + 1);
        }
        if (t == 0)
            return to_string(cnt+1)[0] - '0';
        else
        {
            t += ((int)log10(cnt) + 1);
            return to_string(cnt)[t] - '0';
        }
    }
};
```

### 数学分析

[看题解啊](https://leetcode-cn.com/problems/shu-zi-xu-lie-zhong-mou-yi-wei-de-shu-zi-lcof/solution/mian-shi-ti-44-shu-zi-xu-lie-zhong-mou-yi-wei-de-6/)。

先看一张图（图源自上述题解）。

<img src="https://gitee.com/sinkinben/pic-go/raw/master/img/20201031183620.png" style="width:80%;" />

对于一位数，产生的序列长度为 $9 \times 1$ .

对于两位数，产生的序列长度为 $90 \times 2$ .

其余依次类推。

> 对于第 n 位对应的数字，我们令这个数字对应的数为 `number`，然后分三步进行：
>
> - 首先找到这个数字对应的数是几位数，用 `len` 表示；
> - 然后确定这个对应的数的数值 `number`；
> - 最后确定返回值是 `number` 中的哪个数字。

以 `n = 365` 为例子：

+ `n = 365 - 9 - 90*2 = 176`，不足以继续减去 `900 * 3` ，因此要找到序列中， `100` 之后的第 176 个数字。
+ 100 之后的每个数字都是 3 个长度，因此对应的数字为 `number = 100 + 176 / 3 = 158` .
+ `idx = 176 % 3 = 2` , 因此是 `number` 中的第二位数字 `5` .

需要注意的是，如果 `idx = 0` ，答案应该为 `number-1` 的最后一位数字。

比如 `n = 99` 时：

+ `n = 99 - 9 = 90`，因此要找到序列中，`10` 之后的第 90 个数字。
+ `number = 10 + 90/2 = 55`
+ 显然，从 10 开始，每个数字的长度均为  2 ，`[10, 54]` 一共 45 个数字，序列中第 90 个字符是 `54` 中的 `4` 。

时间复杂度 $O(\log n)$ .

```cpp
class Solution {
public:
    int findNthDigit(int n) 
    {
        if (n <= 9) return n;
        int64_t base = 9, len = 1;
        while (n - base*len >= 0)
        {
            n -= base*len;
            base *= 10, len++;
        }
        int idx = n % len;
        int number = pow(10, len-1) + n/len;
        if (idx == 0) return to_string(number-1).back() - '0';
        return to_string(number)[idx-1] - '0';
    }
};
```



## 45 把数组排成最小的数

题目：[剑指 Offer 45. 把数组排成最小的数](https://leetcode-cn.com/problems/ba-shu-zu-pai-cheng-zui-xiao-de-shu-lcof/)。

转换为字符串，根据自定义的规则进行排序。

```cpp
class Solution {
public:
    string minNumber(vector<int>& nums) 
    {
        vector<string> vs;
        for (int x: nums) vs.push_back(to_string(x));
        sort(vs.begin(), vs.end(), [&](string s1, string s2) {return s1+s2<s2+s1;});
        string buf = "";
        for (auto &x: vs) buf += x;
        return buf;
    }
};
```

