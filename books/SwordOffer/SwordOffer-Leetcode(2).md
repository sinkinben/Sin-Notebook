## [leetcode] 剑指 Offer 专题（二）

《剑指 Offer》专题第二部 。

## 16 数值的整数次方

题目：[剑指 Offer 16. 数值的整数次方](https://leetcode-cn.com/problems/shu-zhi-de-zheng-shu-ci-fang-lcof/)。

递归快速幂。如果 `n` 是负数，那么先求正数次幂的（取相反数），结果取倒数。因为涉及相反数，因此需要特殊考虑 `n` 是否为 `-2147483648` .

```cpp
class Solution {
public:
    double myPow(double x, int n) {
        if (n  == 0) return 1;
        else if (n == 1) return x;
        else
        {
            bool sign = (n < 0);
            bool ismin = (n == (int)0x80000000);
            if (ismin) n++;
            if (sign) n = -n;
            double k = myPow(x, n >> 1);
            k *= k;
            if (n & 1) k *= x;
            if (ismin) k *= x;
            return sign ? (1/k):k;
        }
    }
};
```

## 17 打印从 1 到最大的 n 位数

题目：[剑指 Offer 17. 打印从1到最大的n位数](https://leetcode-cn.com/problems/da-yin-cong-1dao-zui-da-de-nwei-shu-lcof/)。

Python 的话只需要一行：

```python
return list(range(1, pow(10, n)))
```

C++ 无脑解法：

```cpp
class Solution {
public:
    vector<int> printNumbers(int n) {
        int limit = pow(10, n);
        vector<int> v;
        for (int i=1; i<limit; i++)
            v.push_back(i);
        return v;
    }
};
```

但是如果面试的时候，`n` 给出很大呢，即 $10^n$ 超出了一个 `int` 的表示范围，该怎么处理？

通过字符串模拟。

```cpp
class Solution
{
public:
    vector<int> printNumbers(int n)
    {
        // 从左到右存放低位->高位
        char str[n + 1];
        memset(str, '0', n);
        str[n] = '\0';
        while (increment(str, n))
            print(str, n);
        return vector<int>();
    }

    void print(char str[], int n)
    {
        int i = n - 1;
        while (i >= 0 && str[i] == '0')
            i--;
        assert(i != -1);
        while (i >= 0)
            cout << str[i--];
        cout << endl;
    }
	// 模拟加法，溢出 n 位时返回 false
    bool increment(char str[], int size)
    {
        int cur = str[0] - '0' + 1;
        if (cur <= 9)
        {
            str[0] = cur + '0';
            return true;
        }
        else
        {
            int inbit = 1;
            int i = 0;
            str[i] = '0';
            while (inbit)
            {
                i++;
                cur = str[i] - '0' + inbit;
                inbit = (cur >= 10);
                str[i] = (inbit ? ('0') : (cur + '0'));
                if (i == size - 1 && inbit)
                    return false;
            }
            return true;
        }
    }
};
```

## 18 链表删除节点

水题。

```cpp
class Solution {
public:
    ListNode* deleteNode(ListNode* head, int val) {
        if (head == NULL) return NULL;
        if (head->val == val) return head->next;
        auto pre = head, p = head->next;
        while (p != NULL)
        {
            if (p->val == val)
            { pre->next = p->next; break; }
            pre = p, p = p->next;
        }
        return head;
    }
};
```

## 移除重复节点

题目：[面试题 02.01. 移除重复节点](https://leetcode-cn.com/problems/remove-duplicate-node-lcci/)。

跟书上要求不一致，这里是未排序的链表。使用集合记录是否第一次出现即可。

```cpp
class Solution {
public:
    ListNode* removeDuplicateNodes(ListNode* head) {
        if (head == nullptr) return nullptr;
        set<int> s;
        s.insert(head->val);
        auto pre = head, p = head->next;
        while (p != nullptr)
        {
            if (s.count(p->val) == 0)
            {
                s.insert(p->val);
                pre = p, p = p->next;
            }
            else
            {
                pre->next = p->next;
                p = p->next;
            }
        } 
        return head;
    }
};
```

## 19 正则表达式匹配

🙅‍♂️ 题目：[剑指 Offer 19. 正则表达式匹配](https://leetcode-cn.com/problems/zheng-ze-biao-da-shi-pi-pei-lcof/)。

使用C++库函数只需要一行代码：

```cpp
return regex_match(s, regex(p));
```

但是面试总不能作弊，对吧 😅 。

评论区的解法：

```c
bool isMatch(char* s, char* p){
    if(*p == '\0')
    {
        if(*s == '\0') return true;
        else return false;
    } //递归出口

    if(*(p+1) == '*')
    {
        if(isMatch(s, p+2)) return true;
        if(*s != '\0' && (*s == *p || *p == '.') && isMatch(s+1, p)) return true;
    }//判断‘*’符号是否能匹配上

    if(*s != '\0' && (*s == *p || *p == '.') && isMatch(s+1, p+1)) return true; //判断正常字符和‘.’的逐一匹配
    else return false;
}
```

## 20 表示数值的字符串

🙅‍♂️ 题目：[剑指 Offer 20. 表示数值的字符串](https://leetcode-cn.com/problems/biao-shi-shu-zhi-de-zi-fu-chuan-lcof/)。

都 TM 是自动机，人晕了 🤒️。

做的时候给我整吐了 🤮 ，测试用例千奇百怪，题目要求也不说清楚。`46.e3` 这也能算数字？

💬 OS：19 和 20 题都是**确定有限自动机**类型的题目，有空会专门补上针对这 2 题的题解。真正参加搬砖面试机考，都是属于直接放弃型的，纯恶心人。

来一个书本上的解法：

```cpp
class Solution {
public:
    unordered_set<char> valid = {'e', 'E', '.', '+', '-'};
    bool isNumber(string s) 
    {
        // 允许头尾有空格，预处理去除之
        int i=0, j=s.length()-1;
        while (i<=j && s[i] == ' ') i++;
        while (j>=i && s[j] == ' ') j--;
        s = s.substr(i, j-i+1);
        
        int len = s.length();
        if (len == 0) return false;
        int idx = 0;
        bool result = scanInteger(s, idx);
        if (idx < len)
        {
            if (s[idx] =='.')
            {
                idx++;
                result = scanUnsignedInteger(s, idx) || result;
                // 为什么是 || 呢？
                // 因为 
                // ".123" == "0.123"
                // "123." == "123.0"
                // "123.4"
                // 这三种都算符合要求的数字
            }
            if (s[idx] == 'e' || s[idx] == 'E')
            {
                idx++;
                result = result && scanInteger(s, idx);
                // 为什么是 && 呢？因为 e/E 后面必须是一个带符号的整数（题目要求又不说，简直傻到家了）
            }
        }
        return result && idx == len;
    }
    inline bool isDigital(char c) {return '0'<=c && c<='9';}
    inline bool isValid(char c) {return valid.count(c) != 0;}
    bool scanUnsignedInteger(const string &s, int &idx)
    {
        int before = idx;
        int len = s.length();
        while (idx < len && isDigital(s[idx])) idx++;
        // 判断中间是否为不合法字符，例如 "3me" ，面向测试用例编程
        if (idx < len && !isValid(s[idx])) return false;
        return idx > before;
    }
    bool scanInteger(const string &s, int &idx)
    {
        if (idx >= (int)s.length()) return false;
        if (s[idx] == '+' || s[idx] == '-') idx++;
        return scanUnsignedInteger(s, idx);
    }
};
```



## 21  调整数组顺序使奇数位于偶数前面

题目：[剑指 Offer 21. 调整数组顺序使奇数位于偶数前面](https://leetcode-cn.com/problems/diao-zheng-shu-zu-shun-xu-shi-qi-shu-wei-yu-ou-shu-qian-mian-lcof/)。

头尾各一个指针，头指针往后找第一个偶数，尾指针往前找第一个奇数，交换之。重复操作直到两指针相遇。

```cpp
class Solution {
public:
    vector<int> exchange(vector<int>& nums) {
        int len = nums.size();
        int i = 0, j = len - 1;
        while (i < j)
        {
            while (i<len && (nums[i]%2 != 0)) i++;
            while (j>=0 && (nums[j]%2 == 0)) j--;
            if (i>j) break;
            else swap(nums[i], nums[j]);
        }
        return nums;
    }
};
```

进一步扩展，对上述方法改进，使得具有可扩展性，例如满足：

+ 所有负数在前
+ 所有被 k 整除的数字在前
+ ...

显然就是改 `while` 循环的判断条件，我们可以通过 `lambda` 表达式来实现这种可扩展性。

```cpp
class Solution
{
public:
    vector<int> exchange(vector<int> &nums)
    { return helper(nums, [](int x) { return x % 2 == 0; }); }
    vector<int> helper(vector<int> &nums, function<bool(int x)> fcmp)
    {
        int len = nums.size();
        int i = 0, j = len - 1;
        while (i < j)
        {
            while (i < len && !fcmp(nums[i])) i++;
            while (j >= 0 && fcmp(nums[j])) j--;
            if (i > j) break;
            else swap(nums[i], nums[j]);
        }
        return nums;
    }
};
```

## 22 链表中倒数第 k 个节点

题目：[剑指 Offer 22. 链表中倒数第k个节点](https://leetcode-cn.com/problems/lian-biao-zhong-dao-shu-di-kge-jie-dian-lcof/)。

双指针。让一个指针先走 k 步，最后第二个指针从头开始陪跑，第一个指针到尾时，第二个指针就是倒数第 k 。

边界情况：如果 k 大于链表节点数，那么返回原链表（测试用例好像没有这种情况）。3

```cpp
class Solution {
public:
    ListNode* getKthFromEnd(ListNode* head, int k) {
        auto p = head;
        while (p != NULL && k)
            p = p->next, k--;
        if (p == NULL) return head;
        auto q = head;
        while(p != NULL)
            q = q->next, p = p->next;
        return q;
    }
};
```

## 23 链表中环的入口节点

本题包含 2 个知识点：

+ 如何判断链表有环
+ 怎么找到环的入口

第一个问题是：[141. 环形链表](https://leetcode-cn.com/problems/linked-list-cycle/) 。

第二个问题是：[142. 环形链表 II](https://leetcode-cn.com/problems/linked-list-cycle-ii/) 。

### 环形链表

快慢指针。快指针每次走 2 步，慢指针每次走 1 步。

```cpp
class Solution {
public:
    bool hasCycle(ListNode *head) {
        if (head == nullptr) return false;
        // 初始化必须为不同节点
        auto p = head, q = head->next;
        while (p && q)
        {
            if (p == q) return true;
            p = p->next, q = q->next;
            if (q) q = q->next;
        }
        return false;
    }
};
```

### 环形链表 II

先判断是否有环。

根据上面的「快慢指针」法，找到快慢指针相遇的节点 `keyNode` ，该节点必然是环中的一个节点。

然后计算环的节点个数 `k` 。最后，`p` 先走 `k` 步，`q` 从头开始，同样的速度前进，那么相遇的节点就是答案。

```cpp
class Solution {
public:
    ListNode *detectCycle(ListNode *head) {
        const auto keyNode = meetingNode(head);
        if (keyNode == nullptr) return nullptr;
        // 计算环的长度 k
        int k = 1;
        auto p = keyNode->next;
        while (p != keyNode)
            p = p->next, k++;
        // 先走 k 步
        p = head;
        for (int i=0; i<k; i++) p = p->next;
        // 找到相遇节点
        auto q = head;
        while (p != q) p = p->next, q = q->next;
        return p;
    }
    ListNode* meetingNode(ListNode *head)
    {
        if (head == nullptr) return nullptr;
        auto p = head, q = head->next;
        while (p && q)
        {
            if (p == q) return p;
            p = p->next, q = q->next;
            if (q) q = q->next;
        }
        return nullptr;
    }
};
```



## 24 反转链表

题目：[剑指 Offer 24. 反转链表](https://leetcode-cn.com/problems/fan-zhuan-lian-biao-lcof/)。

双指针。

```cpp
class Solution {
public:
    ListNode* reverseList(ListNode* head) {
        if (head == nullptr) return nullptr;
        auto pre = head, p = head->next;
        while (p != nullptr)
        {
            auto next = p->next;
            p->next = pre;
            pre = p, p = next;
        }
        head->next = nullptr; // very important
        return pre;
    }
};
```

## 25 合并两个排序链表

题目：[剑指 Offer 25. 合并两个排序的链表](https://leetcode-cn.com/problems/he-bing-liang-ge-pai-xu-de-lian-biao-lcof/)。

双指针法（类似于归并排序）。

新建一个全新链表：

```cpp
class Solution {
public:
    ListNode* mergeTwoLists(ListNode* l1, ListNode* l2) {
        const auto head = new ListNode(-1);
        auto ptr = head;
        auto p = l1, q = l2;
        while (p && q)
        {
            if (p->val <= q->val)
                ptr->next = new ListNode(p->val), p = p->next;
            else
                ptr->next = new ListNode(q->val), q = q->next;
            ptr = ptr->next; 
        }
        while (p)
            ptr->next = new ListNode(p->val), ptr = ptr->next, p = p->next;
        while (q)
            ptr->next = new ListNode(q->val), ptr = ptr->next, q = q->next;
        return head->next;
        
    }
};
```

在原地合并：

```cpp
class Solution {
public:
    ListNode* mergeTwoLists(ListNode* l1, ListNode* l2) {
        auto head = new ListNode(-1);
        auto cur = head;
        auto p = l1, q = l2;
        while (p && q)
        {
            if (p->val <= q->val) cur->next = p, p = p->next;
            else  cur->next = q, q = q->next;
            cur = cur->next;
        }
        while (p) cur->next = p, p = p->next, cur = cur->next;
        while (q) cur->next = q, q = q->next, cur = cur->next;
        return head->next;
    }
};
```

## 26 树的子结构

题目：[剑指 Offer 26. 树的子结构](https://leetcode-cn.com/problems/shu-de-zi-jie-gou-lcof/)。

遍历 A 的每一个节点 `p`，如果 `p->val == B->val` 说明 `p` 可能是包含 B 的子树。我们通过 `has` 函数来判断 `t1` 是否包含 `t2` .

```cpp
class Solution {
public:
    bool isSubStructure(TreeNode* A, TreeNode* B) 
    {
        if (!A || !B) return false;
        queue<TreeNode*> q;
        q.push(A);
        while (!q.empty())
        {
            auto p = q.front();
            q.pop();
            if (p->val == B->val && has(p, B))
                return true;
            if (p->left) q.push(p->left);
            if (p->right) q.push(p->right);
        }
        return false;
    }
    bool has(TreeNode *t1, TreeNode *t2)
    {
        // t2 遍历完成，说明 t1 包含 t2
        if (t2 == nullptr) return true;
        // 此时 t2 不空，但 t1 为空，说明 t1 不包含 t2
        if (t1 == nullptr) return false;
        if (t1->val != t2->val) return false;
        return has(t1->left, t2->left) && has(t1->right, t2->right);
    }
};
```

## 27 二叉树的镜像

题目：[剑指 Offer 27. 二叉树的镜像](https://leetcode-cn.com/problems/er-cha-shu-de-jing-xiang-lcof/)。

对于每个节点，交换它的左右指针即可。这里使用层次遍历。

```cpp
class Solution {
public:
    TreeNode* mirrorTree(TreeNode* root) 
    {
        if (root == nullptr) return nullptr;
        queue<TreeNode*> q;
        q.push(root);
        while (!q.empty())
        {
            auto p = q.front();
            q.pop();
            swap(p->left, p->right);
            if (p->left) q.push(p->left);
            if (p->right) q.push(p->right);
        }
        return root;
    }
};
```

## 28 对称二叉树

题目：[剑指 Offer 28. 对称的二叉树](https://leetcode-cn.com/problems/dui-cheng-de-er-cha-shu-lcof/)。

递归实现。 `t1` 的左（右）子树与 `t2` 的右（左）子树是相同的。

```cpp
class Solution {
public:
    bool isSymmetric(TreeNode* root) {
        if (root == nullptr) return true;
        return judge(root->left, root->right);
    }

    bool judge(TreeNode *t1, TreeNode *t2)
    {
        bool a = (t1 != nullptr), b = (t2 != nullptr);
        // 一空一不空
        if (a ^ b) return false;
        // 均为空
        if (!a && !b) return true;
        // 均不为空
        if (t1->val != t2->val) return false;
        return judge(t1->left, t2->right) && judge(t1->right, t2->left);
    }
};
```

## 29 顺时针打印矩阵

题目：[剑指 Offer 29. 顺时针打印矩阵](https://leetcode-cn.com/problems/shun-shi-zhen-da-yin-ju-zhen-lcof/)。

使用 `(i, j)` 表示当前位置，`l, r, u, p` 分别表示四个方向的边界（循环过程中逐渐向中心靠拢）。每次循环打印一圈。

```cpp
class Solution {
public:
    vector<int> spiralOrder(vector<vector<int>>& matrix) {
        if (matrix.size() == 0 || matrix[0].size() == 0)
            return vector<int>();
        int rows = matrix.size(), cols = matrix[0].size();
        // boundary of left, right, up, down
        int l = 0, r = cols-1, u = 0, d = rows-1;
        // current position
        int i=0, j=0;
        vector<int> v;
        while (true)
        {
            // ➡️
            while (j<=r) v.push_back(matrix[i][j]), j++;
            if (v.size() == rows*cols) break;
            j--, i++, u++;
            // ⬇️
            while (i<=d) v.push_back(matrix[i][j]), i++;
            if (v.size() == rows*cols) break;
            i--, j--, r--;
            // ⬅️
            while (j>=l) v.push_back(matrix[i][j]), j--;
            if (v.size() == rows*cols) break;
            j++, i--, d--;
            // ⬆️
            while (i>=u) v.push_back(matrix[i][j]), i--;
            if (v.size() == rows*cols) break;
            i++, j++, l++;
        }
        return v;
    }
};
```

