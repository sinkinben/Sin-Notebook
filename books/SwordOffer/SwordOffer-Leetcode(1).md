## [leetcode] 剑指 Offer 专题（一）

又开了一个笔记专题的坑，未来一两周希望能把《剑指Offer》的题目刷完🤒️。

## 03 数组中重复的数字

题目：[剑指 Offer 03. 数组中重复的数字](https://leetcode-cn.com/problems/shu-zu-zhong-zhong-fu-de-shu-zi-lcof/)。

常见思路：

+ 哈希表计数。遍历，把 `bool table[n]` 中没有遇到过的数字置为 1 。时间复杂度 $O(n)$, 空间复杂度 $O(n)$ .
+ 排序。时间 $O(nlogn)$，空间 $O(1)$ .

书中解法：排序后的某个数字 `k` 必然会在 `nums[k]` 的位置上。如果输入数组每个数字都是唯一的，那么不会出现 `nums[i] == nums[nums[i]]` 的情况，如果有，就是所求的重复数字。时间 $O(n)$，空间 $O(1)$ .

```cpp
class Solution {
public:
    int findRepeatNumber(vector<int>& nums) {
        int len = nums.size();
        for (int i=0; i<len; i++)
        {
            while (nums[i] != i)
            {
                if (nums[i] == nums[nums[i]])
                    return nums[i];
                else
                    swap(nums[i], nums[nums[i]]);
            }
        }
        return -1;
    }
};
```



## 04 二维数组中的查找

题目：[剑指 Offer 04. 二维数组中的查找](https://leetcode-cn.com/problems/er-wei-shu-zu-zhong-de-cha-zhao-lcof/)。

**解法1**

对每一行用一次二分查找。

```cpp
class Solution
{
public:
    bool findNumberIn2DArray(vector<vector<int>> &matrix, int target)
    {
        return method1(matrix, target);
    }

    bool method1(vector<vector<int>> &m, int t)
    {
        for (auto &v : m)
        {
            if (binsearch(v,t))
                return true;
        }
        return false;
    }
    bool binsearch(vector<int> &v, int t)
    {
        int len = v.size();
        if (len == 0)
            return false;
        int l = 0, r = len - 1;
        while (l <= r)
        {
            if (l == r)
                return v[l] == t;
            int m = l + (r - l) / 2;
            if (t < v[m])
                r = m - 1;
            else if (v[m] < t)
                l = m + 1;
            else
                return true;
        }
        return false;
    }
};
```

**解法2**

利用每一列也是升序序列的规律，对列也进行二分。

```cpp
bool method2(vector<vector<int>> &m, int t)
{
    if (m.size() == 0 || m[0].size() == 0)
        return false;
    int rows = m.size();
    int cols = m[0].size();
    int i = 0, j = cols - 1;
    while (i < rows && j >= 0)
    {
        if (m[i][j] == t)
            return true;
        else if (t < m[i][j])
            j--;
        else
            i++;
    }
    return false;
}

```



## 05 替换空格

题目: [剑指 Offer 05. 替换空格](https://leetcode-cn.com/problems/ti-huan-kong-ge-lcof/).

**解法1**

用 `string` 的 API. 时间复杂度 $O(N)$ .

```cpp
class Solution
{
public:
    string replaceSpace(string s)
    {
        const string target = "%20";
        const string src = " ";
        size_t pos = s.find(src, 0);
        while (pos != string::npos)
        {
            s.replace(pos, src.length(), target);
            pos = pos + target.length();
            pos = s.find(src, pos);
        }
        return s;
    }
};
```

**解法2**

首先计算空格数目, 然后数组后增加空间, 总长度为 `len + spaces * 2` .

<img src="https://gitee.com/sinkinben/pic-go/raw/master/img/20201013154339.png" style="width:65%;" />

开辟如上图所示的 2 个指针, 让 `p1` 从尾到头扫描. 

如果 `s[p1]` 不是空格, 那么令 `s[p2--] = s[p1--]` , 把字符复制到填充空格后的位置. 

如果 `s[p1]` 是空格, 那么令:

```cpp
s[p2--] = '0';
s[p2--] = '2';
s[p2--] = '%';
```

扫面的边界条件是: `p1>0 && p1<p2` .

## 06 从尾到头打印链表

题目: [剑指 Offer 06. 从尾到头打印链表](https://leetcode-cn.com/problems/cong-wei-dao-tou-da-yin-lian-biao-lcof/).

**解法1**

使用栈 `stack`. 每次遍历一个节点就压栈, 最后输出栈.

**解法2**

但是要求返回值是一个 `vector` , 所以我们还是直接用 `vector` 依次记录, 最后调用 `reverse` 函数.

```cpp
class Solution
{
public:
    vector<int> reversePrint(ListNode *head)
    {
        vector<int> s;
        auto p = head;
        while (p!= nullptr)
            s.emplace_back(p->val), p = p->next;
        reverse(s.begin(), s.end());
        return s;
    }
};
```

## 07 重建二叉树

题目: [剑指 Offer 07. 重建二叉树](https://leetcode-cn.com/problems/zhong-jian-er-cha-shu-lcof/).

**解法**

这里是给定先序和中序, 但其原理与给定中序和后序是一样的. 请看: https://www.cnblogs.com/sinkinben/p/11455712.html.

```cpp
class Solution
{
public:
    TreeNode *buildTree(vector<int> &preorder, vector<int> &inorder)
    {
        int ridx = 0;
        return innerBuild(preorder, inorder, ridx, 0, inorder.size() - 1);
    }

    inline int vfind(vector<int> &v, int val)
    {
        return find(v.begin(), v.end(), val) - v.begin();
    }

    TreeNode *innerBuild(vector<int> &preorder, vector<int> &inorder, int &ridx, int l, int r)
    {
        if (ridx >= (int)preorder.size() || l > r)
            return nullptr;
        if (l == r)
            return new TreeNode(preorder[ridx++]);
        int pos = vfind(inorder, preorder[ridx]);
        auto p = new TreeNode(preorder[ridx++]);
        p->left = innerBuild(preorder, inorder, ridx, l, pos - 1);
        p->right = innerBuild(preorder, inorder, ridx, pos + 1, r);
        return p;
    }
};
```

## 08 二叉树的下一个节点

Leetcode 题库缺失本题.

题目要求: 给定二叉树和当中的某个节点, 要求返回该节点在中序遍历中的下一个节点 (指针形式) .

题目 API : `TreeNode *getNext(TreeNode *root, TreeNode *node)` .

`TreeNode` 结构包括: `val, left, right, parent` .

在中序遍历中, 是「左 根 右」的顺序遍历.

如果 `node` 存在右子树, 那么其下一个节点就是右子树的最左节点.

如果没有右子树:

+ 如果 `node` 是左子树, 那么下一个节点是 `node->parent` .
+ 如果 `node` 是右子树, 那么下一个节点是 `node` 的祖先, 并且这个祖先的左子树也是 `node` 的祖先. 即下面这种情况:

```
         root
        /
      x1
     /  \
   x2    x3
     \
      x4
        \
         x5
        /
      x6
inorder = [x2, x4, x6, x5, x1, x3, root]
getNext(x5) = x1
```

代码实现: 

```cpp
TreeNode *min(TreeNode *p)
{
    if (p == nullptr) return p;
    while (p->left != nullptr)
        p = p->left;
   	return p;
}

TreeNode *getNext(TreeNode *root, TreeNode *node)
{
    if (node == nullptr) return nullptr;
    if (node->right != nullptr) return min(node->right);
    auto x = node, y = node->parent;
    while (y != nullptr && y->right == x)
        x = y, y = y->parent;
    return y;
}
```

## 09 用两个栈实现队列

题目：[剑指 Offer 09. 用两个栈实现队列](https://leetcode-cn.com/problems/yong-liang-ge-zhan-shi-xian-dui-lie-lcof/) 。

**解法1：暴力**

把 `s1` 的序列倒进 `s2` ，那么 `s2` 就是对列对应的顺序了。最后把 `s2` 倒回 `s1` 保持状态一致性。

```cpp
class CQueue
{
public:
    stack<int> s1, s2;
    CQueue()
    {
    }

    void appendTail(int value)
    {
        s1.push(value);
    }

    int deleteHead()
    {
        while (!s1.empty())
            s2.push(s1.top()), s1.pop();
        if (s2.empty())
            return -1;
        int t = s2.top();
        s2.pop();
        while (!s2.empty())
            s1.push(s2.top()), s2.pop();
        return t;
    }
};
```

**解法2：官方解法**

把栈变为队列顺序的方法：**倒进另外一个栈。**

所以我们使用 `s1` 来保存新加入的元素，`s2` 来保存准备出队的元素。在删除时，只要对 `s2` 进行判断：

+ 如果为空，那么就 `s1` 的所有元素倒过来，丢一个出去。
+ 如果不为空，那么直接丢一个出去。

一图胜千言。

<img src="https://gitee.com/sinkinben/pic-go/raw/master/img/20201014163608.png" style="width:60%;" />

代码实现：

```cpp
class CQueue
{
public:
    stack<int> s1, s2;
    CQueue()
    {
    }

    void appendTail(int value)
    {
        s1.push(value);
    }

    int deleteHead()
    {
        int t = -1;
        if (s2.empty())
        {
            while (!s1.empty())
                s2.push(s1.top()), s1.pop();
            if (!s2.empty())
                t = s2.top(), s2.pop();
        }
        else
        {
            t = s2.top(), s2.pop();
        }
        return t;
    }
};
```

### 附加题：两个队列实现一个栈

题目：[225. 用队列实现栈](https://leetcode-cn.com/problems/implement-stack-using-queues/)。

当栈不为空时，两个队列有且只有一个是有元素的，空的队列作为一个 `buffer` 。当需要获取队列头部元素，就把不空队列的前 n-1 个元素转移到 `buffer`，剩下的一个就是栈顶元素。

看图。

<img src="https://gitee.com/sinkinben/pic-go/raw/master/img/20201014164653.png" style="width:67%;" />

代码实现：

```cpp
class MyStack {
public:
    queue<int> q1, q2;
    /** Initialize your data structure here. */
    MyStack() {

    }
    /** Push element x onto stack. */
    void push(int x) {
        if (q1.empty()) q2.push(x);
        else if (q2.empty()) q1.push(x);
        else assert(0);
    }
    /** Removes the element on top of the stack and returns that element. */
    int pop() {
        if (!q1.empty() && q2.empty())
        {
            while (q1.size() > 1) q2.push(q1.front()), q1.pop();
            int t = q1.front();
            q1.pop();
            return t;
        }
        if (q1.empty() && !q2.empty())
        {
            while (q2.size() > 1) q1.push(q2.front()), q2.pop();
            int t = q2.front();
            q2.pop();
            return t;
        }
        return -1;        
    }    
    /** Get the top element. */
    int top() {
        if (!q1.empty() && q2.empty())
        {
            while (q1.size() > 1) q2.push(q1.front()), q1.pop();
            int t = q1.front();
            q2.push(t), q1.pop();
            return t;
        }
        if (q1.empty() && !q2.empty())
        {
            while (q2.size() > 1) q1.push(q2.front()), q2.pop();
            int t = q2.front();
            q1.push(t), q2.pop();
            return t;
        }
        return -1;
    }
    /** Returns whether the stack is empty. */
    bool empty() {
        return q1.empty() && q2.empty();
    }
};
```

## 10-I 斐波那契数列

```cpp
class Solution {
public:
    int fib(int n) {
        const int mod = 1000000007;
        if (n==0 || n==1) return n;
        int f0=0, f1=1, f2=1;
        for (int i=2; i<=n; i++)
            f2=(f0+f1)%mod, f0=f1, f1=f2;
        return f2;
    }
};
```

## 10-II 青蛙跳台阶

```cpp
class Solution {
public:
    int numWays(int n) {
        const int mod = 1000000007;
        if (n==0 || n==1) return 1;
        int f0=1, f1=1, f2=2;
        for (int i=2; i<=n; i++)
            f2=(f0+f1)%mod, f0=f1, f1=f2;
        return f2;
    }
};
```

本题扩展：

+ 如果 🐸 每次可以跳 1，2，3，...，n 个台阶，那么跳上 $n$ 个台阶的方法数目 $f(n)=2^{n-1}$ .

可通过数学归纳法证明之。

当 $n=1$ 或 $n=2$，$f(n)=n$ .

当 $n=k$，设 $f(k) = 2^{k-1}$ .

当 $n=k+1$，则有 $f(k+1) = 1+\sum_{i=1}^{k}f(k) = 2^{k}$ . 

为什么呢？有 $k+1$ 个台阶时，🐸 王子可以：

+ 直接走 k+1 步
+ 先走 1 步，再走 k 步
+ 先走 2 步，再走 k-1 步
+ ...
+ 先走 k 步，再走 1 步

证毕。 

+ 用 2x1 的矩形填充 2xN 的矩形，有多少种填充方法？

斐波那契数列 $f(n)$ 。

如果竖着填充：

```
|x| | | | ... | |
|x| | | | ... | |
```

如果横着填充：

```
|x|x| | | ... | |
|x|x| | | ... | |
```

所以 $f(n) = f(n-1)+f(n-2)$ .

## 11 旋转数组的最小数字

题目：[剑指 Offer 11. 旋转数组的最小数字](https://leetcode-cn.com/problems/xuan-zhuan-shu-zu-de-zui-xiao-shu-zi-lcof/)。

暴力解法 $O(N)$。

使用二分查找，时间复杂度 $O(logn)$ 。请看[官方题解](https://leetcode-cn.com/problems/xuan-zhuan-shu-zu-de-zui-xiao-shu-zi-lcof/solution/xuan-zhuan-shu-zu-de-zui-xiao-shu-zi-by-leetcode-s/)。

```cpp
class Solution
{
public:
    int minArray(vector<int> &v)
    {
        int len = v.size();
        if (len == 0) return -1;
        if (len == 1) return v.front();
        int l = 0, r = len - 1, m = l;
        while (l <= r)
        {
            if (l == r)
                break;
            m = l + (r - l) / 2;
            if (v[m] < v[r])
                r = m;
            else if (v[m] > v[r])
                l = m + 1;
            else
                r--;
        }
        return v[l];
    }
};
```

## 12 矩阵中的路径

题目：[剑指 Offer 12. 矩阵中的路径](https://leetcode-cn.com/problems/ju-zhen-zhong-de-lu-jing-lcof/)。

显然是 DFS 搜索路径啦。使用 `'.'` 符号来标记该位置已被搜索。

```cpp
class Solution
{
public:
    int rows = 0, cols = 0;
    bool exist(vector<vector<char>> &board, string word)
    {
        if (board.size() == 0 || board[0].size() == 0)
            return word.length() == 0;
        rows = board.size(), cols = board[0].size();
        for (int i = 0; i < rows; i++)
        {
            for (int j = 0; j < cols; j++)
            {
                if (word[0] == board[i][j] && dfs(board, word, 1, i, j))
                    return true;
            }
        }
        return false;
    }

    bool dfs(vector<vector<char>> &b, const string &word, int idx, int x, int y)
    {
        if (idx == (int)word.length())
            return true;
        char tmp = b[x][y], target = word[idx++];
        b[x][y] = '.';
        if (x + 1 < rows && b[x + 1][y] == target && dfs(b, word, idx, x + 1, y))
            return true;
        if (x - 1 >= 0 && b[x - 1][y] == target && dfs(b, word, idx, x - 1, y))
            return true;
        if (y + 1 < cols && b[x][y + 1] == target && dfs(b, word, idx, x, y + 1))
            return true;
        if (y - 1 >= 0 && b[x][y - 1] == target && dfs(b, word, idx, x, y - 1))
            return true;
        b[x][y] = tmp;
        return false;
    }
};
```

## 13 机器人的运动范围

题目：[剑指 Offer 13. 机器人的运动范围](https://leetcode-cn.com/problems/ji-qi-ren-de-yun-dong-fan-wei-lcof/)。

依旧是 DFS 。使用二维数组记录该位置是否被访问。

```cpp
// #include "leetcode.h"
class Solution
{
public:
    int rows = 0, cols = 0, limit = 0;
    int ans = 0;
    int movingCount(int m, int n, int k)
    {
        rows = m, cols = n, limit = k;
        vector<vector<bool>> visited(rows, vector<bool>(cols, false));
        dfs(visited, 0, 0);
        return ans;
    }

    int calculate(int x, int y)
    {
        int t = 0;
        while (x)
            t += (x % 10), x /= 10;
        while (y)
            t += (y % 10), y /= 10;
        return t;
    }

    void dfs(vector<vector<bool>> &visited, int x, int y)
    {
        if (visited[x][y])
            return;
        visited[x][y] = true, ans++;
        if (x + 1 < rows && calculate(x + 1, y) <= limit)
            dfs(visited, x + 1, y);
        if (x - 1 >= 0 && calculate(x - 1, y) <= limit)
            dfs(visited, x - 1, y);
        if (y + 1 < cols && calculate(x, y + 1) <= limit)
            dfs(visited, x, y + 1);
        if (y - 1 >= 0 && calculate(x, y - 1) <= limit)
            dfs(visited, x, y - 1);
    }
};
```

## 14-I 剪绳子

题目：[剑指 Offer 14- I. 剪绳子](https://leetcode-cn.com/problems/jian-sheng-zi-lcof/)。

### 数学解法

基本不等式：**和为定值，积有最大值。当且仅当所有数相等时，取得最大值。**

假设分成 $k$ 份：
$$
\sum_{i=1}^{k} n_i = n \\
$$

并且有：
$$
\frac{(n_1 + n_2 + \dots + n_k)}{k} \ge \sqrt[k]{n_1 \cdot n_2 \dots n_k} \\
$$
设将绳子按照长度 $x$ 等分为 $a$ 段时，所得的积最大，为 $x^a = x^{\frac{n}{x}}$ .

令 $y=f(x)=x^{\frac{n}{x}}$, 其中 $n$ 为常数，那么有：
$$
\ln{y} = \frac{n}{x} \ln{x} \\
\frac{1}{y} \cdot y' = (-\frac{n}{x^2})\ln{x} + \frac{n}{x^2} \\
\frac{1}{y} \cdot y' = \frac{n - n\ln{x}}{x^2}
$$
令 $y'=0$ ，得：$x = e$ . 分析单调性得，$x = e$ 时，$y=f(x)$ 取得最大值。

又因为长度 $x$ 必须为整数，所以取等分长度为 2 或者 3 。

显然，通过某些简单的 n 可以验证等分长度取 3 所得的积才是最大的。例如：n=6, n=10. 

还有一个问题时，如果剩余长度不足 3 怎么办？按等分长度 3 分割，剩余长度可以为：0, 1, 2 。

显然，当剩余 1 长度时，我们比较 $3^a \times 1$ 和 $3^{a-1} \times 4$ 即可（后者大）。

原本以为是动态规划，其实是道数学题，人晕了 🤒️ 。

```cpp
#include <cmath>
class Solution {
public:
    int cuttingRope(int n) 
    {
        if (n <= 3) return n-1;
        int a = n/3, b=n%3;
        if (b==0) return pow(3,a);
        else if (b==1) return pow(3,a-1)*4;
        else return pow(3,a)*2;
    }
};
```

### 动态规划解法

状态定义：`dp[i]` 为分割长度为 `i` 的绳子所得的最大积。

边界：`dp[0] = dp[1] = 0` . 因为不能分割。

转移方程：
$$
dp[i] = \max_{1 \le j < i} (j \times (i-j), j \times dp[i-j])
$$
解析：当从长度为 $i$ 的绳子剪出一段 $j$ ，那么剩下的部分可剪可不剪。如果剪，那就是 $j \times dp[i-j]$, 如果不剪，那就是 $j \times (i-j)$ .

```cpp
int dpMethod(int n)
{
    vector<int> dp(n+1,0);
    for (int i=2;i<=n;i++)
    {
        for (int j=1;j<i;j++)
            dp[i] = max(dp[i], max(j*(i-j), j*dp[i-j]));
    }
    return dp[n];
}
```

## 14-II 剪绳子 II

题目：[剑指 Offer 14- II. 剪绳子 II](https://leetcode-cn.com/problems/jian-sheng-zi-ii-lcof/)。

只是 n 的范围变大了，需要取模。

```cpp
class Solution {
public:
    int cuttingRope(int n) {
        if (n<=3) return n-1;
        const int mod = (int)(1e9+7);
        int a = n/3;
        int b = n%3;
        int64_t k = 1;
        while (--a) k = (k*3)%mod;
        if (b==0) return (k*3)%mod;
        else if (b==1) return (k*4) % mod;
        else return (k*6)%mod;
    }
};
```

## 15 二进制中 1 的个数

位操作的经典套路。

```cpp
class Solution {
public:
    int hammingWeight(uint32_t n) {
        int t = 0;
        while (n)
            t++, n&=(n-1);
        return t;
    }
};
```

