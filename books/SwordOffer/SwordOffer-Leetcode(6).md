## [leetcode] 剑指 Offer 专题（六）

《剑指 Offer》专题第六部。

## 46 🎈把数字翻译成字符串

题目：[剑指 Offer 46. 把数字翻译成字符串](https://leetcode-cn.com/problems/ba-shu-zi-fan-yi-cheng-zi-fu-chuan-lcof/)。

参考[题解](https://leetcode-cn.com/problems/ba-shu-zi-fan-yi-cheng-zi-fu-chuan-lcof/solution/mian-shi-ti-46-ba-shu-zi-fan-yi-cheng-zi-fu-chua-6/)。

**状态定义**

`dp[i]` 是以第 `i` 个字符结尾的子串 `s[1, ..., i]` 的翻译方法数，`s` 下标从 1 开始。

**转移方程**

根据 `s[i-1, i]` 是否可组合翻译，可以分为以下 2 种情况：

+ 可组合：`dp[i] = dp[i-2] + dp[i-1]` .
+ 不可组合：`dp[i] = dp[i-1]`

解析如下图所示（图源自上述题解👆）。

<img src="https://gitee.com/sinkinben/pic-go/raw/master/img/20201102205139.png" style="width:60%;" />

注意到，当 `s[i-1] = 0` 时，与 `s[i]` 组成的两位数是无法翻译的，比如 `00, 01, 02, ..., 09` ，因此有转移方程：
$$
dp[i] = 
\left \{
\begin{aligned}
&dp[i-1]+dp[i-2], \quad &if \quad 10 \le 10*s_{i-1}+s_i \le 25 \\
&dp[i-1], \quad &otherwise
\end{aligned}
\right.
$$


**初始状态**

`dp[0] = 1, dp[1] = 1` ， `dp[0]` 表示空串，`dp[1]` 表示只有一个字符的翻译次数。

```cpp
class Solution {
public:
    int translateNum(int num) {
        string s = to_string(num);
        int len = s.length();
        vector<int> dp(len+1, 0);
        dp[0] = 1, dp[1] = 1;
        for (int i=2; i<=len; i++)
        {
            if (valid(s[i-2], s[i-1])) dp[i] = dp[i-1] + dp[i-2];
            else dp[i] = dp[i-1];
        }
        return dp.back();
    }
    inline bool valid(char a, char b)
    {
        int k = (a-'0')*10+(b-'0');
        return 10<=k && k<=25;
    }
};
```



## 55-II 平衡二叉树

题目：[剑指 Offer 55 - II. 平衡二叉树](https://leetcode-cn.com/problems/ping-heng-er-cha-shu-lcof/)。

### 递归

半暴力解法。通过一个 `map` 来记录节点的高度，结合后序遍历，减少高度的重复计算。

计算各个节点的高度遍历了一次树，求解是否平衡又需要一次后序遍历，因此这里一共遍历了 2 次。

```cpp
class Solution {
public:
    unordered_map<TreeNode*, int> m = {{nullptr, 0}};
    bool result = true;
    bool isBalanced(TreeNode* root) 
    {
        postorder(root);
        return result;
    }
    void postorder(TreeNode *p)
    {
        if (p == nullptr || !result) return;
        postorder(p->left);
        postorder(p->right);
        int l = height(p->left), r = height(p->right);
        if (abs(l - r) >= 2) result = false;
    }
    int height(TreeNode *p)
    {
        if (p == nullptr || m.count(p) == 1) return m[p];
        return m[p] = 1 + max(height(p->left), height(p->right));
    }
};
```

### 书本解法

只需遍历一次。

后序遍历。当前节点的高度与左右子树的高度有关，因此使用后序遍历，自底向上求解高度，实现 $O(N)$ 时间内求解所有节点的高度。

```cpp
class Solution {
public:
    bool isBalanced(TreeNode* root) 
    {
        int depth = 0;
        return isBalanced(root, depth);
    }

    bool isBalanced(TreeNode *p, int &depth)
    {
        if (p == nullptr)
        {
            depth = 0;
            return true;
        }
        int ldepth = 0, rdepth = 0;
        if (isBalanced(p->left, ldepth) && isBalanced(p->right, rdepth))
        {
            if (abs(ldepth - rdepth) <= 1)
            {
                depth = 1 + max(ldepth, rdepth);
                return true;
            }
        }
        return false;
    }
};
```

## 56-I 数组中数字出现的次数

题目：[剑指 Offer 56 - I. 数组中数字出现的次数](https://leetcode-cn.com/problems/shu-zu-zhong-shu-zi-chu-xian-de-ci-shu-lcof/)。

要求空间复杂度是 $O(1)$，我们不能使用哈希计数了。

如果出现一次的数字只有一个，那么就是所有元素异或后的结果。

智力题，直接看[题解](https://leetcode-cn.com/problems/shu-zu-zhong-shu-zi-chu-xian-de-ci-shu-lcof/solution/shu-zu-zhong-shu-zi-chu-xian-de-ci-shu-by-leetcode/)。

假设只出现一次的 2 个数字是 `a` 和 `b` 。那么所有元素异或后的结果 `k == a^b` .

+ 找到 `k` 中任意一位为 1 的比特（这里找的是最低的 1 比特位）。该 1 要么来自于 `a` 要么来自于 `b` 。设这个比特位为 `div` 。
+ 把数组分为 2 组：`div` 位为 0 的和 `div` 位为 1 的。显然， `a` 和 `b` 必然来自不同的组。
+ 单独看这 2 组数字，相当于**找出唯一的，只出现一次的数字**  。

代码实现：

```cpp
class Solution {
public:
    vector<int> singleNumbers(vector<int>& nums) 
    {
        int k = 0;
        for (int x: nums) k^=x;
        // now, k == a ^ b
        uint32_t div = 1;
        while ((div & k) == 0) div = (div << 1);
        int a = 0, b = 0;
        for (int x: nums)
        {
            if ((x & div) != 0) a^=x;
            else b^=x;
        }
        return {a,b};
    }
};
```

## 56-II 数组中数字出现的次数 II

题目：[剑指 Offer 56 - II. 数组中数字出现的次数 II](https://leetcode-cn.com/problems/shu-zu-zhong-shu-zi-chu-xian-de-ci-shu-ii-lcof/)。

数组元素展开为比特串，将一维数组看作看作是一个 `n x 32` 的矩阵。那么，每一列的 1 比特的数目之和，对 3 模运算后（结果必定是 0 或 1 ，不可能是 2 ），剩下的 1 就必定来自于出现一次的数字。

<img src="https://gitee.com/sinkinben/pic-go/raw/master/img/20201102210312.png" style="width:60%;" />

```cpp
class Solution {
public:
    int singleNumber(vector<int>& nums) 
    {
        int table[32] = {0};
        for (int x: nums)
        {
            for (int i=0; i<32; i++)
                table[i] += ((x >> i) & 1);
        }
        int res = 0;
        for (int i=0; i<32; i++)
        {
            table[i] %= 3;
            res |= (table[i] << i);
        }
        return res;
    }
};
```

## 57 和为 s 的两个数字

题目：[剑指 Offer 57. 和为s的两个数字。](https://leetcode-cn.com/problems/he-wei-sde-liang-ge-shu-zi-lcof/)

数组呈升序排列，采用头尾双指针法。如果和比 `target` 大，那么右指针向中间移动，否则左指针向中间移动。

```cpp
class Solution {
public:
    vector<int> twoSum(vector<int>& nums, int target) 
    {
        int len = nums.size();
        int i=0, j=len-1;
        while (i < j)
        {
            int t = nums[i] + nums[j];
            if (t ==  target) break;
            else if (t > target) j--;
            else i++;

        }
        return {nums[i], nums[j]};
    }
};
```

## 57-II 和为 s 的连续正数序列

题目：[剑指 Offer 57 - II. 和为s的连续正数序列](https://leetcode-cn.com/problems/he-wei-sde-lian-xu-zheng-shu-xu-lie-lcof/)。

滑动窗口算法。`[i,j]` 表示当前的窗口范围，`sum` 表示该区间的和。

+ 当 `sum < target` 时，窗口向右端移动。
+ 当 `sum == target` 时，保存序列 `[i,j]`，窗口左端移动，寻找下一个序列。
+ 当 `sum > target` 时，窗口左端移动。

```cpp
class Solution {
public:
    vector<vector<int>> findContinuousSequence(int target) 
    {
        auto generator = [](int x, int y) {
            vector<int> v;
            for (int i=x; i<=y; i++) v.push_back(i);
            return v;
        };
        int i=1, j=1;
        int sum = 0;
        int limit = target/2 + 1;
        vector<vector<int>> ans;
        while (j <= limit)
        {
            sum += j, j++;
            while (sum > target)
                sum -= i, i++;
            if (sum == target)
            {
                ans.push_back(generator(i, j-1));
                sum -= i, i++;
            }
        }
        return ans;
    }
};
```

##  58-I 翻转单词顺序

题目：[剑指 Offer 58 - I. 翻转单词顺序](https://leetcode-cn.com/problems/fan-zhuan-dan-ci-shun-xu-lcof/)。

利用 `stringstream` 将单词分离，同时也能去除多余空格，分离出来的单词压入栈即可。

```cpp
class Solution {
public:
    string reverseWords(string s) 
    {
        stringstream ss(s);
        string t;
        stack<string> buf;
        while (ss >> t) buf.push(t);
        t = "";
        while (!buf.empty()) t += buf.top() + " ", buf.pop();
        while (t.back() == ' ') t.pop_back();
        return t;
    }
};
```

使用纯字符串处理的笨方法。

```cpp
class Solution {
public:
    string reverseWords(string s) 
    {
        int i=0, j=0, len=s.length();
        while (s[i] == ' ') i++;
        string res = "";
        while (j<len)
        {
            j=i;
            while (j<len && s[j]!=' ') j++;
            auto t = s.substr(i, j-i);
            if (t.back() != ' ') t.push_back(' ');
            res = t + res;
            while (j<len && s[j] == ' ') j++;
            i=j;
        }
        while (res.back() == ' ') res.pop_back();
        return res;
    }
};
```

## 58-II 左旋转字符串

题目：[剑指 Offer 58 - II. 左旋转字符串](https://leetcode-cn.com/problems/zuo-xuan-zhuan-zi-fu-chuan-lcof/)。

使用自带的 `rotate` 库函数。

```cpp
std::rotate(s.begin(), s.begin()+n, s.end());
return s;
```

C++ 还可以这样：

```cpp
return (s + s).substr(n, s.length());
```

也可以：

```cpp
return s.substr(n) + s.substr(0, n);
```

如果是 Python，我们还可以：

```python
return s[n:] + s[:n];
```

书本解法（💬这个规律到底是怎么发现的）：

```cpp
class Solution {
public:
    string reverseLeftWords(string s, int n) 
    {
        reverse(s.begin(), s.begin() + n);
        reverse(s.begin() + n, s.end());
        reverse(s.begin(), s.end());
        return s;   
    }
};
```

## 59-I 滑动窗口的最大值

题目：[剑指 Offer 59 - I. 滑动窗口的最大值](https://leetcode-cn.com/problems/hua-dong-chuang-kou-de-zui-da-zhi-lcof/)。

### 暴力查找

问就是暴力，罗永浩：又不是不能用.jpg。时间复杂度 $O(nk)$ .

```cpp
class Solution {
public:
    vector<int> maxSlidingWindow(vector<int>& nums, int k) 
    {
        vector<int> res;
        int size = nums.size();
        if (size == 0) return res;
        if (size < k) return {*max_element(nums.begin(), nums.end())};
        for (int i=0;i<=size-k;i++)
        {
            int cur = nums[i];
            for (int j=i; j<i+k; j++) cur=max(cur, nums[j]);
            res.push_back(cur);
        }
        return res;
    }
};
```

### 单调队列

参考[题解](https://leetcode-cn.com/problems/hua-dong-chuang-kou-de-zui-da-zhi-lcof/solution/mian-shi-ti-59-i-hua-dong-chuang-kou-de-zui-da-1-6/)。

```cpp
class Solution {
public:
    vector<int> maxSlidingWindow(vector<int>& nums, int k) {
        size_t size = nums.size();
        if (size == 0 || k == 0) return {};
        if (size < k) return {*max_element(nums.begin(), nums.end())};
        vector<int> res;
        deque<int> q;
        for (int i=0; i<k; i++)
        {
            while (!q.empty() && nums[i] > q.back()) q.pop_back();
            q.push_back(nums[i]);
        }
        res.push_back(q.front());
        for (int i=k; i<size; i++)
        {
            if (!q.empty() && q.front() == nums[i-k]) q.pop_front();
            while (!q.empty() && nums[i] > q.back()) q.pop_back();
            q.push_back(nums[i]);
            res.push_back(q.front());
        }
        return res;
    }
};
```

## 59-II 队列中的最大值

题目：[剑指 Offer 59 - II. 队列的最大值](https://leetcode-cn.com/problems/dui-lie-de-zui-da-zhi-lcof/)。

跟上面的 59-I 一毛一样。

```cpp
class MaxQueue {
public:
    deque<int> data, helper;
    MaxQueue() { }
    int max_value() {
        if (!helper.empty()) return helper.front();
        return -1;
    }   
    void push_back(int value) {
        data.push_back(value);
        while (!helper.empty() && helper.back() < value) helper.pop_back();
        helper.push_back(value);
    }    
    int pop_front() {
        if (data.empty()) return -1;
        int t = data.front();
        data.pop_front();
        if (!helper.empty() && helper.front() == t) helper.pop_front();
        return t;
    }
};
```



## 61 扑克牌中的顺子

题目：[剑指 Offer 61. 扑克牌中的顺子](https://leetcode-cn.com/problems/bu-ke-pai-zhong-de-shun-zi-lcof/)。

排序。

大小王（用 0 表示）可以变成任意的数字，因此先计算 0  的个数 `zero` 。然后判断空缺的数目 `gap` 是否小于等于 `zero` 。

比如 `[1,3,5]` 的空缺数目是 2 ，缺失了 2 和 4 两张牌。

特殊情况：顺子中不能有 2 个相同的数字。

```cpp
class Solution {
public:
    bool isStraight(vector<int>& nums) 
    {
        sort(nums.begin(), nums.end());
        int size = nums.size();
        int i = 0;
        while (i < size && nums[i] == 0) i++;
        int zero = i;
        int gap = 0;
        i++; // i 指向第二个非 0 的数
        for (; i<size; i++)
        {
            if (nums[i] == nums[i-1]) return false;
            gap += (nums[i] - nums[i-1] - 1);
        }
        return gap <= zero;
    }
};
```



## 62 约瑟夫环

题目：[剑指 Offer 62. 圆圈中最后剩下的数字](https://leetcode-cn.com/problems/yuan-quan-zhong-zui-hou-sheng-xia-de-shu-zi-lcof/)。

### 模拟

使用链表模拟，TLE。

```cpp
class Solution
{
public:
    int lastRemaining(int n, int m)
    {
        list<int> li;
        for (int i = 0; i < n; i++) li.push_back(i);
        auto p = li.begin();
        while (li.size() > 1)
        {
            for (int i = 1; i < m; i++)
            {
                p++;
                if (p == li.end()) p = li.begin();
            }
            auto next = ++p;
            if (next == li.end()) next = li.begin();
            li.erase(--p);
            p = next;
        }
        return li.back();
    }
};
```

### 数学

看了老多数学证明，还是不怎么理解那个递推公式。

倒是这里的[题解](https://leetcode-cn.com/problems/yuan-quan-zhong-zui-hou-sheng-xia-de-shu-zi-lcof/solution/c-dao-tui-fa-mian-shi-ti-62-yuan-quan-zhong-zui-ho/)，简单明了。

一剑封喉。

> 最终剩下一个人时的安全位置肯定为 0，反推安全位置在人数为 n 时的编号.
> 人数为 1 ： 0
> 人数为 2 ： (0+m) % 2
> 人数为 3 ： ((0+m) % 2 + m) % 3
> 人数为 4 ： (((0+m) % 2 + m) % 3 + m) % 4
> ...
> 迭代推理到 n 就可以得出答案

老 IQ 题了，不适合我这种老人家 👴 做。

```cpp
class Solution {
public:
    int lastRemaining(int n, int m) {
        int f = 0;
        for (int i=2; i<=n; i++) f = (f + m) % i;
        return f;
    }
};
```

## 63 股票的最大利润

题目：[剑指 Offer 63. 股票的最大利润](https://leetcode-cn.com/problems/gu-piao-de-zui-da-li-run-lcof/)。

做烂了。

```cpp
class Solution {
public:
    int maxProfit(vector<int>& prices) 
    {
        int size = prices.size();
        int profit = 0;
        int minval = 0x7fffffff;
        for (int x: prices)
        {
            minval = min(minval, x);
            profit = max(profit, x - minval);
        }
        return profit;
    }
};
```

## 64 求 SUM(1,n)

题目：[剑指 Offer 64. 求1+2+…+n](https://leetcode-cn.com/problems/qiu-12n-lcof/)。

### 骚方法递归

```cpp
class Solution {
public:
    int sumNums(int n) {
        n && (n = n + sumNums(n-1));
        return n;
    }
};
```

### 利用二进制乘法特性

参考[题解](https://leetcode-cn.com/problems/qiu-12n-lcof/solution/qiu-12n-by-leetcode-solution/)。

```cpp
class Solution {
public:
    int sum = 0;
    int sumNums(int n) {
        uint32_t x = n, y = n+1;
        for (int i=0; i<32; i++)
            (x & 1) && (sum += y), x >>= 1, y <<= 1;
        return sum >> 1;
    }
};
```

### 求和公式

```cpp
return (pow(n, 2) + n) >> 1;
```

或者：

```cpp
bool a[n][n+1];
return sizeof(a) >> 1;
```

### 构造函数

利用 `static` 的特性。

```cpp
class Sum
{
public:
    static int sum, n;
    Sum() { sum += (++n); }
};
int Sum::sum = 0;
int Sum::n = 0;
class Solution
{
public:
    int sumNums(int n)
    {
        Sum *s = new Sum[n];
        int t = Sum::sum;
        Sum::sum = 0, Sum::n = 0;
        delete[] s;
        s = nullptr;
        return t;
    }
};
```

### 虚函数多态

利用动态联编。

```cpp
class A
{ public: virtual int sum(int n) { return 0; } };
static A *func[2];
class B : public A
{ public: int sum(int n) { return n + func[!!n]->sum(n - 1); } };
class Solution
{
public:
    int sumNums(int n)
    {
        A a;
        B b;
        func[0] = &a, func[1] = &b;
        return func[!!n]->sum(n);
    }
};
```

其实也可以用函数指针的形式：

```cpp
class Solution {
public:
    int sumNums(int n) { return mySum(n); }
    static int mySum(int n)
    {
        static auto terminator = [](int n) {return 0;};
        function<int(int)> func[2]  = {terminator, mySum};
        return n + func[!!n](n-1);
    }
};
```

### 模版元编程

```cpp
template<uint64_t n> struct Sum
{
    enum Value {N = Sum<n-1>::N + n};
};
template<> struct Sum<1>
{
    enum Value {N = 1};
};
// return Sum<100>::N;
```

缺点：模版参数必须为一个编译器就能确定的常量，不是动态输入。

## 65 不用加减乘除实现加法

利用异或是**不带进位加法**这一规则。

需要考虑如何处理进位？当且仅当 a 和 b 的某一位均为 1 的时候产生进位，因此进位是 `t = (a & b) << 1` .

详细解析请看[题解](https://leetcode-cn.com/problems/bu-yong-jia-jian-cheng-chu-zuo-jia-fa-lcof/solution/mian-shi-ti-65-bu-yong-jia-jian-cheng-chu-zuo-ji-7/) .

```cpp
class Solution {
public:
    int add(int a, int b) 
    {
        while (b != 0)
        {
            uint32_t t = (uint32_t)(a & b) << 1;
            a ^= b;
            b = t;
        }
        return a;
    }
};
```

