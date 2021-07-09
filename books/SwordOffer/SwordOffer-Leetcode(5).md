## [leetcode] 剑指 Offer 专题（五）

《剑指 Offer》专题第五部。

## 47 礼物的最大价值

🎁 题目：[剑指 Offer 47. 礼物的最大价值](https://leetcode-cn.com/problems/li-wu-de-zui-da-jie-zhi-lcof/)。

动态规划水题。

```cpp
class Solution {
public:
    int maxValue(vector<vector<int>>& grid) 
    {
        if (grid.size() == 0 || grid[0].size() == 0) return 0;
        int rows = grid.size();
        int cols = grid[0].size();
        for (int j=1; j<cols; j++) grid[0][j] += grid[0][j-1];
        for (int i=1; i<rows; i++) grid[i][0] += grid[i-1][0];
        for (int i=1; i<rows; i++)
            for (int j=1; j<cols; j++)
                grid[i][j] += max(grid[i-1][j], grid[i][j-1]);
        return grid.back().back();
    }
};
```

## 48 最长不含重复字符的子字符串

题目：[剑指 Offer 48. 最长不含重复字符的子字符串](https://leetcode-cn.com/problems/zui-chang-bu-han-zhong-fu-zi-fu-de-zi-zi-fu-chuan-lcof/)。

动态规划。

状态定义：$dp[i]$ 为选中 $s[i]$ 时的不含重复字符的，最长子串的长度。

设 `s[j]` 是在 `i` 之前，且离 `i` 最近的，满足 `s[j] == s[i]` 的字符。如果 `[0, ..., i-1]` 没有与 `s[i]` 重复的字符，那么 `j` 为 -1 , 并且设 `d = i-j` .

转移方程：
$$
dp[i] = 
\left\{
\begin{aligned}
&dp[i-1] + 1  &if \quad &j=-1 \\
& d           &if \quad &d \le dp[i-1] \\
&dp[i-1] + 1  &if \quad &d > dp[i-1] 
\end{aligned}
\right.
$$
$j = -1$ 的情况是显然的。

当 $d \le dp[i-1]$ 时，说明重复字符出现在 $dp[i-1]$ 所表示的无重复最长子串当中。例如 `s = "ababca"` , `i = 5` 时,  `d = 3, dp[i-1] = 3` , 这时候的无重复最长子串为 `"bca"` .

当 $d > dp[i-1]$ 时，说明重复字符出现在 $d[i-1]$ 所表示的无重复最长子串之外。例如 `s = "abcbca"` , `i = 5` 是，`d = 5, dp[i-1] = 2`，这时候的无重复最长子串为 `"bca"` .

```cpp
class Solution {
public:
    int lengthOfLongestSubstring(string s) 
    {
        int len = s.length();
        if (len == 0) return 0;
        vector<int> dp(len, 1);
        int maxval = 1;
        for (int i=1; i<len; i++)
        {
            int j = i-1;
            while (j>=0 && s[j] != s[i]) j--;
            int d = i-j;
            if (j == -1 || d > dp[i-1])
                dp[i] = dp[i-1]+1;
            else if (d <= dp[i-1])
                dp[i] = d;
            maxval = max(maxval, dp[i]);
        }
        return maxval;
    }
};
```

空间优化：

```cpp
class Solution {
public:
    int lengthOfLongestSubstring(string s) 
    {
        int len = s.length();
        if (len == 0) return 0;
        int dp = 1, maxval = 1;
        for (int i=1; i<len; i++)
        {
            int j = i-1;
            while (j>=0 && s[j] != s[i]) j--;
            int d = i-j;
            if (j == -1 || d > dp) dp = dp+1;
            else if (d <= dp) dp = d;
            maxval = max(maxval, dp);
        }
        return maxval;
    }
};
```

## 49 丑数

题目：[剑指 Offer 49. 丑数](https://leetcode-cn.com/problems/chou-shu-lcof/)。

### 暴力

从 1 开始不断扫描，知道找到第 n 个丑数。超时。

```cpp
class Solution {
public:
    int nthUglyNumber(int n) 
    {
        int cnt = 1;
        int order = 1;
        while (cnt < n)
        {
            order++;
            cnt += judge(order);
        }
        return order;
    }

    bool judge(int n)
    {
        while (n % 2 == 0) n/=2;
        while (n % 3 == 0) n/=3;
        while (n % 5 == 0) n/=5;
        return n==1;
    }
};
```

### 动态规划

这是书本的解法。

主要思想：第 `i+1` 个丑数 `v[i]` 肯定是从前 `i` 个丑数 `v[0, ..., i-1]` 中的某一个，乘以 `{2,3,5}` 中的一个数所得的。

结合以下 2 篇题解食用更佳。

解析 1 ：[归并的思想](https://leetcode-cn.com/problems/chou-shu-lcof/solution/chou-shu-ii-qing-xi-de-tui-dao-si-lu-by-mrsate/)。

解析 2 ：[偏数学的推导](https://leetcode-cn.com/problems/chou-shu-lcof/solution/mian-shi-ti-49-chou-shu-dong-tai-gui-hua-qing-xi-t/)。推荐看该文章里面的动画解析。

<img src="https://gitee.com/sinkinben/pic-go/raw/master/img/20201023192000.png" style="width:50%;" />

丑数序列由 3 个子序列组成，即 $2a, 3b, 5c$ 组成，其中 $a,b,c = 1,\dots,n$ 。

使用数组记录丑数，当求解第 `i+1` 个丑数 `v[i]` 时，`v[a]*2, v[b]*3, v[c]*5` 依次分别为三个序列中大于 `v[i-1]` 且离 `v[i]` 最近的一个丑数，因此最小的那一个就是第 `i+1` 个丑数 `v[i]`。

```cpp
class Solution {
public:
    int nthUglyNumber(int n) 
    {
        vector<int> v(n, 0);
        v[0] = 1;
        int a = 0, b = 0, c = 0;
        for (int i=1; i<n; i++)
        {
            v[i] = min(v[a]*2, min(v[b]*3, v[c]*5));
            if (v[i] == v[a] * 2) a++;
            if (v[i] == v[b] * 3) b++;
            if (v[i] == v[c] * 5) c++;
        }
        return v.back();
    }
};
```

## 50 第一个只出现一次的字符

题目：[剑指 Offer 50. 第一个只出现一次的字符](https://leetcode-cn.com/problems/di-yi-ge-zhi-chu-xian-yi-ci-de-zi-fu-lcof/)。

哈希计数。

```cpp
class Solution {
public:
    char firstUniqChar(string s) 
    {
        int cnt[26] = {0};
        for (int x: s) cnt[x - 'a']++;
        for (int x: s)
            if (cnt[x - 'a'] == 1)
                return x;
        return ' ';
    }
};
```

### 附加题：字符流中第一个只出现一次的字符

假如有一个无限长的字符流，每次读入一个字符，设计一个类，可以返回已读入字符中，第一个只出现一次的字符。

```cpp
class CharFinder
{
public:
    // table[x] == 0, 'x' is only inserted once
    // table[x] == -1, 'x' is not inserted
    // table[x] == -2, 'x' is inserted more than once
    // flag 用于记录只出现一次的字符的读入顺序
    int flag;
    int table[256];
    CharFinder():flag(0) { memset(table, sizeof(int) * 256, -1); }
    void insert(char x)
    {
        if (table[x] == -1)
            table[x] = flag;
       	else if (table[x] == 0)
            table[x] = -2;
       	flag++;
    }
    // 找到 table 中，大于等于 0 ，且 table[i] 最小的那个 i
    char get()
    {
        int minFlag = 0x7fffffff;
        char x = -1;
        for (int i=0; i<256; i++)
        {
            if (table[i] >= 0 && table[i] < minFLag)
                minFlag = table[i], x = i;
        }
        return x;
    }
};
```



## 51 数组中的逆序对

题目：[剑指 Offer 50. 第一个只出现一次的字符](https://leetcode-cn.com/problems/di-yi-ge-zhi-chu-xian-yi-ci-de-zi-fu-lcof/)。

### 暴力

别问，问就是枚举每一对。当然会超时啦～

```cpp
class Solution {
public:
    int reversePairs(vector<int>& nums) 
    {
        int rev = 0, size = nums.size();
        for (int i=0; i<size; i++)
            for (int j=i+1; j<size; j++)
                rev += nums[i] > nums[j];
        return rev;
    }
};
```

### 归并思想

时间复杂度 $O(n \log n)$，空间复杂度 $O(n)$ .

参考[官方题解](https://leetcode-cn.com/problems/shu-zu-zhong-de-ni-xu-dui-lcof/solution/shu-zu-zhong-de-ni-xu-dui-by-leetcode-solution/)。

和归并排序有什么关系呢？来看一个例子。

```
     l              mid        mid+1           r
     |               |          |              |
L = [8, 12, 16, 22, 100]   R = [9, 26, 55, 64, 91]  buf = []
     |                          |
   lPtr                       rPtr
```

开始，各个变量初始值如上所示。

根据归并排序的规则，这个时候我们把左边的 8 加入了答案，我们发现右边没有数比  8 小，所以 8 对逆序对总数的「贡献」为 0 。

接着我们继续合并，把 9 加入了答案，此时 `lPtr` 指向 12，`rPtr` 指向 26 。

```
                               mid+1
                                |
L = [8, 12, 16, 22, 100]   R = [9, 26, 55, 64, 91]  buf = [8, 9]
        |                          |
       lPtr                       rPtr
==>
L = [8, 12, 16, 22, 100]   R = [9, 26, 55, 64, 91]  buf = [8, 9, 12]
            |                      |
           lPtr                   rPtr
```

此时 `lPtr` 比 `rPtr` 小，把 `lPtr` 对应的数加入 `buf` ，并考虑它对逆序对总数的贡献为 `rPtr` 相对 R 首位置的偏移 1（即右边只有一个数比 12 小，所以只有它和 12 构成逆序对），以此类推，逆序数为 `rPtr - (mid + 1) ` .

我们发现用这种「算贡献」的思想在合并的过程中计算逆序对的数量的时候，只在 `lPtr` 右移的时候计算，是基于这样的事实：当前 `lPtr` 指向的数字比 `rPtr` 小，但是比 R 中 `[mid+1 ... rPtr - 1]` 的其他数字大，`[0 ... rPtr - 1]` 的其他数字本应当排在 `lPtr` 对应数字的左边，但是它排在了右边，所以这里就贡献了 `rPtr - 1 - (mid + 1) + 1 = rPtr - (mid + 1)` 个逆序对。

```cpp
class Solution
{
public:
    int reversePairs(vector<int> &nums)
    {
        int len = nums.size();
        vector<int> buf(len, 0);
        return mergeSort(nums, buf, 0, len - 1);
    }

    int mergeSort(vector<int> &data, vector<int> &buf, int l, int r)
    {
        if (l >= r) return 0;
        int mid = l + (r - l) / 2;
        int leftCount = mergeSort(data, buf, l, mid);
        int rightCount = mergeSort(data, buf, mid + 1, r);
        int p1 = l, p2 = mid + 1, idx = l;
        int count = 0;
        while (p1 <= mid && p2 <= r)
        {
            if (data[p1] <= data[p2])
                buf[idx] = data[p1++], count += (p2 - (mid + 1));
            else
                buf[idx] = data[p2++];
            idx++;
        }
        while (p1 <= mid) buf[idx++] = data[p1++], count += (p2 - (mid + 1));
        while (p2 <= r) buf[idx++] = data[p2++];
        copy(buf.begin() + l, buf.begin() + idx, data.begin() + l);
        return count + leftCount + rightCount;
    }
};
```

## 52 两个链表的第一个公共节点

题目：[剑指 Offer 52. 两个链表的第一个公共节点](https://leetcode-cn.com/problems/liang-ge-lian-biao-de-di-yi-ge-gong-gong-jie-dian-lcof/)。

### 暴力

问就是双循环。能过就行，能过就行，又不是不能过。

```cpp
class Solution {
public:
    ListNode *getIntersectionNode(ListNode *headA, ListNode *headB) 
    {
        for (auto p = headA; p != nullptr; p = p->next)
        {
            for (auto q = headB; q != nullptr; q = q->next)
                if (p == q) return p;
        }
        return nullptr;
    }
};
```

### 栈

时间复杂度 $O(m+n)$，空间复杂度 $O(m+n)$ .

显然，这 2 个交叉的链表只会是呈 `|` 或者 `Y` 这种形状。

先把所有节点分别压入 2 个栈，那么 2 个栈的栈顶会有若干个相同的节点，把它们都弹出来，找到第一个不同的 `s1.top()` ，那么 `s1.top()->next` 就是第一个公共节点。边界情况：公共节点为 `head` ，即 2 个链表是同一个。

```cpp
class Solution {
public:
    ListNode *getIntersectionNode(ListNode *headA, ListNode *headB) 
    {
        if (headA == nullptr || headB == nullptr) return nullptr;
        stack<ListNode*> s1, s2;
        for (auto p = headA; p!=nullptr; p = p->next) s1.push(p);
        for (auto p = headB; p!=nullptr; p = p->next) s2.push(p);
        while (!s1.empty() && !s2.empty() && s1.top() == s2.top())
            s1.pop(), s2.pop();
        if (s1.empty()) return headA;
        else return s1.top()->next;
    }
};
```

### 长度之差

假设 2 个链表的长度之差为 `k` ，那么先在长链表上走 `k` 步，然后长短链表同时走，每次一步，必然会在某一节点相遇。

下面代码会进行预处理，使得 `headA` 总是长链表。

时间复杂度 $O(n+m)$ 。

```cpp
class Solution {
public:
    ListNode *getIntersectionNode(ListNode *headA, ListNode *headB) 
    {
        if (headA == nullptr || headB == nullptr) return nullptr;
        int alen = 0, blen = 0;
        for (auto p = headA; p!=nullptr; p=p->next) alen++;
        for (auto p = headB; p!=nullptr; p=p->next) blen++;
        if (alen < blen) swap(headA, headB), swap(alen, blen);
        auto p = headA;
        int k = alen - blen;
        for (int i=0; i<k; i++) p = p->next;
        auto q = headB;
        while (p != q) p = p->next, q = q->next;
        return p;
    }
};
```

## 53-I 在排序数组中查找数字 I

题目：[剑指 Offer 53 - I. 在排序数组中查找数字 I](https://leetcode-cn.com/problems/zai-pai-xu-shu-zu-zhong-cha-zhao-shu-zi-lcof/)。

### 双指针

```cpp
class Solution {
public:
    int search(vector<int>& nums, int target) 
    {
        int n = nums.size();
        int i=0, j=n-1;
        while (j>=i && nums[j]!=target) j--;
        while (i<=j && nums[i]!=target) i++;
        return (i<=j)?(j-i+1):0;
    }
};
```

### 二分查找

书本的解法，时间复杂度可降为 $O(\log n)$ .

好无语的解法，空间换时间。

```cpp
class Solution {
public:
    int search(vector<int>& nums, int target) 
    {
        int len = nums.size();
        if (len == 0) return 0;
        int first = getFirstK(nums, 0, len-1, target);
        int last = getLastK(nums, 0, len-1, target);
        return (first!=-1 && last!=-1) ? last-first+1 : 0;
    }

    int getFirstK(const vector<int> &v, int l, int r, const int k)
    {
        if (l > r) return -1;
        if (l == r) return v[l] == k ? l : -1;
        int m = l + (r-l)/2;
        if (v[m] == k)
        {
            if (m == 0 || (m > 0 && v[m-1] != k)) return m;
            else r = m-1;
        }
        else if (v[m] < k) l = m+1;
        else r = m-1;
        return getFirstK(v, l, r, k);
    }
    int getLastK(const vector<int> &v, int l, int r, const int k)
    {
        if (l > r) return -1;
        if (l == r) return v[l] == k ? l : -1;
        int m = l + (r-l)/2;
        int len = v.size();
        if (v[m] == k)
        {
            if ((m == len-1) || (m < len-1 && v[m+1] != k)) return m;
            else l = m+1;
        }
        else if (v[m] < k) l = m+1;
        else r = m-1;
        return getLastK(v, l, r, k);
    }
};
```

### 进一步优化的二分法

`binsearch` 返回值是 `k` 在有序数组中的插入位置，如果值相等，那么默认插在右边。

`binsearch(target)` 返回的是 `target` 的插入位置，即 `target` 出现的最后位置加一。

`binsearch(target-1)` 返回的是 `target-1` 的插入位置，即 `target` 第一次出现的位置。

特殊情况：如果 `target` 没有出现在数组中，但是 `binsearch` 仍会返回相应的插入位置，与 `target-1` 相同，因此返回 0 ，满足要求。

```cpp
class Solution {
public:
    int search(vector<int>& nums, int target) 
    {
        return binsearch(nums, target) - binsearch(nums, target-1);
    }
    int binsearch(vector<int> &v, int k)
    {
        int len = v.size();
        int l = 0, r = len-1;
        while (l <= r)
        {
            int m = l + (r-l)/2;
            if (v[m] <= k) l=m+1;
            else r=m-1;
        }
        return l;
    }
};
```

## 53-II 0～(n-1) 中缺失的数字

题目：[剑指 Offer 53 - II. 0～n-1中缺失的数字](https://leetcode-cn.com/problems/que-shi-de-shu-zi-lcof/)。

```cpp
class Solution {
public:
    int missingNumber(vector<int>& nums) 
    {
        int len = nums.size(), i = 0;
        for (i=0; i<len; i++)
        {
            if (nums[i] != i)
                return i;
        }
        // 处理 nums = [0] 这一情况，正确的输出是 1 
        return i;
    }
};
```

习惯性来个二分查找吧。

正常情况是 `nums[i] == i`，一旦不想等，说明 `[l, m-1]` 区间缺失数字。

```cpp
class Solution {
public:
    int missingNumber(vector<int>& nums) 
    {
        int len = nums.size();
        int l=0, r=len-1;
        while (l<=r)
        {
            int m = l+(r-l)/2;
            if (nums[m] == m) l=m+1;
            else r=m-1;
        }
        return l;
    }
};
```

## 54 二叉搜索树的第 k 大节点

题目：[剑指 Offer 54. 二叉搜索树的第k大节点](https://leetcode-cn.com/problems/er-cha-sou-suo-shu-de-di-kda-jie-dian-lcof/)。

逆中序遍历。

```cpp
class Solution {
public:
    vector<int> v;
    int kthLargest(TreeNode* root, int k) 
    {
        revInorder(root);
        return v[k-1];
    }

    void revInorder(TreeNode *p)
    {
        if (p == NULL) return;
        revInorder(p->right);
        v.push_back(p->val);
        revInorder(p->left);
    }
};
```

空间优化：

```cpp
class Solution {
public:
    const int intmin = 0x80000000;
    int ans = intmin;
    int kthLargest(TreeNode* root, int k) 
    {
        revInorder(root, k);
        return ans;
    }

    void revInorder(TreeNode *p, int &k)
    {
        if (p == NULL) return;
        if (ans == intmin) revInorder(p->right, k);
        k--;
        if (k == 0)
        {
            ans = p->val;
            return;
        }
        if (ans == intmin) revInorder(p->left, k);
    }
};
```

## 55-I 二叉树的高度

题目：[剑指 Offer 55 - I. 二叉树的深度](https://leetcode-cn.com/problems/er-cha-shu-de-shen-du-lcof/)。

做法有很多，根据二叉树结构不同，效率也会不一样：

+ BFS，也就是层次遍历，慢慢数
+ DFS，就是下面👇的解法，最为简洁

 2 种方法时间复杂度均为 $O(n)$ .

```cpp
class Solution {
public:
    int maxDepth(TreeNode* root) 
    {
        return height(root);
    }
    int height(TreeNode *p)
    {
        if (p == NULL) return 0;
        return 1 + max(height(p->left), height(p->right));
    }
};
```

