## [leetcode] 剑指 Offer 专题（三）

《剑指 Offer》专题第三部，后面要去做各种各样课程的大作业了，可能不能集中精力刷，要 🐦 一段时间。

## 30 包含 min 函数的栈

题目：[剑指 Offer 30. 包含min函数的栈](https://leetcode-cn.com/problems/bao-han-minhan-shu-de-zhan-lcof/)。

题目要求所有操作均在 $O(1)$ 内完成，这种 IQ 题我是直接看书上的解法了。

首先一个 `data` 栈跟普通的栈一样，用于保存数据。下面来看如何利用 `minbuf` 实现在 $O(1)$ 时间内完成 `min` 操作。

在 `data` 栈压入新元素 `x` 时，`minbuf` 总是压入 `data` 中最小的值。所以，**两个栈的元素个数总是完全一致的。**

`minbuf` 栈的意义是：保证 `minbuf` 的栈顶是 `data` 所有元素中的最小值。

一图胜千言。

<img src="https://gitee.com/sinkinben/pic-go/raw/master/img/20201017160500.png" style="width:80%;" />

```cpp
class MinStack {
public:
    stack<int> data;
    stack<int> minbuf;
    int minval = 0x7fffffff;
    /** initialize your data structure here. */
    MinStack() {}
    void push(int x) {
        minval  = std::min(minval, x);
        minbuf.push(minval);
        data.push(x);
    }
    void pop() {
        int x = data.top();
        data.pop();
        minbuf.pop();
        if (x == minval)
            minval = minbuf.empty() ? 0x7fffffff : minbuf.top();
    }    
    int top() { return data.top(); }
    int min() { return minval; }
};
```

## 31 栈的压入、弹出队列

题目：[剑指 Offer 31. 栈的压入、弹出序列](https://leetcode-cn.com/problems/zhan-de-ya-ru-dan-chu-xu-lie-lcof/)。

**书本解法：对 popped 序列扫描**

使用一个栈去模拟。

扫描整个 `popped` 序列，`popIdx` 指向当前需要出栈的元素 `x` ，因此在 `pushed` 找到 `x` ，`x` 本身及其之前的元素都要入栈 `s` 。如果在扫描过程中，栈顶元素与 `x` 不一致，说明该出栈序列是无效的。

+ 例子 1

<img src="https://gitee.com/sinkinben/pic-go/raw/master/img/20201017170244.png" style="width:80%;" />

+ 例子 2

<img src="https://gitee.com/sinkinben/pic-go/raw/master/img/20201017170346.png" style="width:80%;" />

```cpp
class Solution {
public:
    bool validateStackSequences(vector<int>& pushed, vector<int>& popped) {
        if (popped.empty()) return pushed.empty();
        int pushlen = pushed.size();
        int poplen = popped.size();
        stack<int> s;
        int pushIdx = 0, popIdx = 0;
        while (popIdx < poplen)
        {
            while (s.empty() || popped[popIdx] != s.top())
            {
                if (pushIdx >= pushlen) break;
                s.push(pushed[pushIdx++]);
            }
            if (s.top() != popped[popIdx])
                break;
            s.pop(), popIdx++;
        }
        return s.empty() && popIdx>=poplen;
    }
};
```

但这种解法居然只超过 5%。

---

**评论区解法：对 pushed 序列扫描**

`popIdx` 指向需要出栈对元素 `x`。

对每个 `pushed` 元素扫描，每次入栈。然后，判断 `x` 是否与栈顶相等，若是则出栈。

```cpp
class Solution {
public:
    bool validateStackSequences(vector<int> &pushed, vector<int> &popped)
    {
        stack<int> s;
        int popIdx = 0;
        for (int x : pushed)
        {
            s.push(x);
            while (!s.empty() && s.top() == popped[popIdx])
                s.pop(), popIdx++;
        }
        return s.empty() && popIdx == (int)popped.size();
    }
};
```

对比这 2 种解法，个人感觉：

+ 解法 1 是假定 `popped` 是正确的，判断 `pushed` 是否匹配。
+ 解法 2 是假定 `pushed` 是正确的，判断 `popped` 是否匹配。

## 32-I 从上到下打印二叉树

题目：[剑指 Offer 32 - I. 从上到下打印二叉树](https://leetcode-cn.com/problems/cong-shang-dao-xia-da-yin-er-cha-shu-lcof/)。

水题，层次遍历。

```cpp
class Solution {
public:
    vector<int> levelOrder(TreeNode* root) {
        vector<int> result;
        if (root == NULL) return  result;
        queue<TreeNode*> q;
        q.push(root);
        while (!q.empty())
        {
            auto p = q.front();
            q.pop();
            result.push_back(p->val);
            if (p->left) q.push(p->left);
            if (p->right) q.push(p->right);
        }
        return result;
    }
};
```

## 32-II 从上到下打印二叉树 II

题目：[剑指 Offer 32 - II. 从上到下打印二叉树 II](https://leetcode-cn.com/problems/cong-shang-dao-xia-da-yin-er-cha-shu-ii-lcof/)。

 再用一个队列 `buf` 记录下一层的节点。

```cpp
class Solution {
public:
    vector<vector<int>> levelOrder(TreeNode* root) {
        vector<vector<int>> res;
        if (root == nullptr) return res;
        queue<TreeNode*> q;
        q.push(root);
        while (!q.empty())
        {
            queue<TreeNode*> buf;
            vector<int> v;
            while (!q.empty())
            {
                auto p = q.front();
                q.pop();
                v.push_back(p->val);
                if (p->left) buf.push(p->left);
                if (p->right) buf.push(p->right);
            }
            res.push_back(v);
            q = buf;
        }
        return res;
    }
};
```

## 32-III 从上到下打印二叉树 III

题目：[剑指 Offer 32 - III. 从上到下打印二叉树 III](https://leetcode-cn.com/problems/cong-shang-dao-xia-da-yin-er-cha-shu-iii-lcof/)。

如果是偶数层，那么调用一次 `reverse` 进行反转。

```cpp
class Solution {
public:
    vector<vector<int>> levelOrder(TreeNode* root) {
        vector<vector<int>> ans;
        if (root == NULL) return ans;
        queue<TreeNode*> q;
        q.push(root);
        bool needRev = false;
        while (!q.empty())
        {
            queue<TreeNode*> next;
            vector<int> v;
            while (!q.empty())
            {
                auto p = q.front();
                q.pop();
                v.push_back(p->val);
                if (p->left) next.push(p->left);
                if (p->right) next.push(p->right);
            }
            q = next;
            if (needRev) reverse(v.begin(), v.end());
            ans.push_back(v);
            needRev = !needRev;
        }
        return ans;
    }
};
```

现在可以考虑优化一下，不调用 `reverse` 反转，是否可行？使用双端队列即可。

```cpp
vector<vector<int>> method2(TreeNode *root)
{
    vector<vector<int>> ans;
    if (root == nullptr)
        return ans;
    deque<TreeNode *> q;
    q.push_back(root);
    while (!q.empty())
    {
        vector<int> cur;
        deque<TreeNode *> next;
        while (!q.empty())
        {
            TreeNode *p = nullptr;
            // 打印奇数层，注意 if-else 的 2 个分支的不同之处
            if (ans.size() % 2 == 0)
            {
                p = q.front();
                q.pop_front();
                if (p->left) next.push_back(p->left);
                if (p->right) next.push_back(p->right);
            }
            // 打印偶数层
            else
            {
                p = q.back();
                q.pop_back();
                if (p->right) next.push_front(p->right);
                if (p->left) next.push_front(p->left);
            }
            cur.push_back(p->val);
        }
        if (!cur.empty())
            ans.push_back(cur);
        q = next;
    }
    return ans;
}
```

## 33 二叉搜索树的后序遍历序列

题目：[剑指 Offer 33. 二叉搜索树的后序遍历序列](https://leetcode-cn.com/problems/er-cha-sou-suo-shu-de-hou-xu-bian-li-xu-lie-lcof/)。

### 递归解法

BST 的特性是：左子树 < 根 < 右子树。而后序遍历是「左右根」的顺序，因此，在后序序列中（最后一个元素是根节点），必然存在一个 `idx` 使得，`[0, idx-1]` 是左子树，`[idx, len-2]` 是右子树。

我们需要对这 2 个区间检查：

+ 左子树都小于根
+ 右子树都大于根

不满足上述条件，即返回 `false` 。

最后，对左右子树进行同样的操作。

因为这里采用了临时变量构建左右子树，空间效率不高，只超过了 5% 的用户。

```cpp
class Solution {
public:
    bool verifyPostorder(vector<int>& postorder) 
    {
        int len = postorder.size();
        if (len == 0) return true;
        int root = postorder.back();

        int idx = 0;
        while (postorder[idx] < root) idx++;

        // [0, idx-1] is left subtree
        // [idx, len-2] is right subtree

        vector<int> ltree, rtree;
        for (int i=0; i<idx; i++) ltree.push_back(postorder[i]);
        for (int i=idx; i<len-1; i++) rtree.push_back(postorder[i]);

        // check rtree
        for (int x: rtree)
            if (x < root)
                return false;
        return verifyPostorder(ltree) && verifyPostorder(rtree);

    }
};
```

优化方法：新建一个函数 `help(const vector<int> &post, int l, int r)` 通过引用和 2 个参数（表示区间）进行递归调用就行了。

```cpp
class Solution {
public:
    bool verifyPostorder(vector<int>& postorder) 
    {
        int len = postorder.size();
        if (len == 0) return true;
        return helper(postorder, 0, len-1);
    }

    bool helper(const vector<int> &postorder, int l, int r)
    {
        if (l >= r) return true;
        int root = postorder[r];
        int idx = l;
        while (postorder[idx] < root) idx++;
        for (int i=idx; i<r; i++)
            if (postorder[i] < root)
                return false;
        return helper(postorder, l, idx-1) && helper(postorder, idx, r-1);
    }
};
```

### 单调栈解法

来源于评论区 [@失火的夏天](https://leetcode-cn.com/u/burning-summer/)，以及[题解](https://leetcode-cn.com/problems/er-cha-sou-suo-shu-de-hou-xu-bian-li-xu-lie-lcof/solution/mian-shi-ti-33-er-cha-sou-suo-shu-de-hou-xu-bian-6/) .

先把此处的 BST 加入一个虚拟根节点：

```
    ROOT
  /
BST
```

BST中，`left -> root -> right` 是呈升序的。

我们把 `postorder` 逆向看，即 `root -> right -> left`，显然，这么看的话 `root->right` 这一区间是严格升序的。那么，在某个降下来的地方，就是意味着该位置及其后面的元素属于左子树。这个左子树的根是谁呢？是该位置前面大于它的，但最小的那一个。

比如 `post = [4, 8, 6, 12, 16, 14, 10]` ，其逆序列为：`[10, 14, 16, 12, 6, 8, 4]` 。

树形结构：

```
        ROOT
      /
     10
   /    \
  6      14
 / \    /  \
4   8  12  16
```

首先，`[10, 14, 16]` 呈升序，属于右子树。

当扫描 `12` 时，该位置时下降的，所以 12 属于左子树，且它的根是前面大于它，离它最近的 14 。14 的根是 10 。

🤒️ 人晕了，这个方法确实不容易想到，太嗯了些。

```cpp
class Solution {
public:
    bool verifyPostorder(vector<int>& postorder) {
        int len = postorder.size();
        // 默认整个 BST 位于虚拟根节点的左子树
        int root = 0x7fffffff;
        stack<int> s;
        for (int i=len-1; i>=0; i--)
        {
            int x = postorder[i];
            // 任意节点都不能大于当前的根
            if (x > root) return false;
            // 来到下降位置，需要找出前面大于 x 但离 x 最近的元素，设为当前的根
            while (!s.empty() && s.top() > x)
                root = s.top(), s.pop();
            s.push(x);
        }
        return true;
    }
};
```

## 34 二叉树中和为某一值的路径

题目：[剑指 Offer 34. 二叉树中和为某一值的路径](https://leetcode-cn.com/problems/er-cha-shu-zhong-he-wei-mou-yi-zhi-de-lu-jing-lcof/)。

回溯法，穷举每一条路径。因为传递参数的方式是 pass by value，因此空间效率不高。

```cpp
class Solution {
public:
    int target = 0;
    vector<vector<int>> result;
    vector<vector<int>> pathSum(TreeNode* root, int sum) 
    {
        target = sum;
        vector<int> path;
        if (root) helper(root, 0, path);
        return result;
    }

    void helper(TreeNode *t, int val, vector<int> path)
    {
        val += t->val;
        path.push_back(t->val);
        if (!t->left && !t->right && val == target)
            result.push_back(path);
        if (t->left) helper(t->left, val, path);
        if (t->right) helper(t->right, val, path);
    }
};
```

通过 `pass by reference` 优化，只需要加一句 `pop_back`：

```cpp
class Solution {
public:
    int target = 0;
    vector<vector<int>> result;
    vector<vector<int>> pathSum(TreeNode* root, int sum) 
    {
        target = sum;
        vector<int> path;
        if (root) helper(root, 0, path);
        return result;
    }
    void helper(TreeNode *t, int val, vector<int> &path)
    {
        val += t->val;
        path.push_back(t->val);
        if (!t->left && !t->right && val == target)
            result.push_back(path);
        if (t->left) helper(t->left, val, path);
        if (t->right) helper(t->right, val, path);
        // look here
        path.pop_back();
    }
};
```

## 35 复杂链表的复制

题目：[剑指 Offer 35. 复杂链表的复制](https://leetcode-cn.com/problems/fu-za-lian-biao-de-fu-zhi-lcof/)。

### 哈希解法

需要 2 次遍历。

设 $x$ 是原节点，$x’$ 是新复制的节点。我们使用一个 `map` 去记录这样的映射：$map[x] = x'$ .

首先，遍历一遍，用 `next` 指针把新链表连接起来。后面考虑把 `random` 指针修正。

假如 $x.random$ 指向了节点 $y$，那么有：$x'.random = y' = m[y] = m[x.random]$ .

时间和空间复杂度均为 $O(N)$ .

```cpp
class Solution {
public:
    Node* copyRandomList(Node* head) 
    {
        if (head == NULL) return NULL;
        auto newHead = new Node(-1);
        auto cur = newHead, p = head;
        unordered_map<Node*, Node*> m;
        while (p)
        {
            auto t = new Node(p->val);
            m[p] = t;
            cur->next = t, cur = cur->next, p = p->next;
        }
        p = head, cur = newHead->next;
        while (p && cur)
        {
            if (p->random) cur->random = m[p->random];
            cur = cur->next, p = p->next;
        }
        return newHead->next;    
    }
};
```

### 书本解法

需要 3 次遍历。

时间 $O(N)$，但空间为 $O(1)$ 的解法。实现起来麻烦一些。

OS：虽然空间是优化了，但是真正机考做题肯定是「哈希解法」更好 0-0 。所以就懒写具体实现了，面试的时候能够说出这种解法，跟面试官吹吹牛就够了 0w0 。

一图胜千言。

第一步，把复制的 $x'$ 插入到原节点 $x$ 的后面。

<img src="https://gitee.com/sinkinben/pic-go/raw/master/img/20201017204828.png" style="width:80%;" />

第二步，把 $x'$ 的 `random` 指针指向 `x->random->next` .

<img src="https://gitee.com/sinkinben/pic-go/raw/master/img/20201017204928.png" style="width:80%;" />

第三步，根据奇偶性分离出 2 个链表。

<img src="https://gitee.com/sinkinben/pic-go/raw/master/img/20201017205129.png" style="width:80%;" />

### DFS 和 BFS

将这个复杂链表看作是一个有向图，如下图所示（源自 leetcode [讨论区](https://leetcode-cn.com/problems/fu-za-lian-biao-de-fu-zhi-lcof/solution/lian-biao-de-shen-kao-bei-by-z1m/)）。

<img src="https://gitee.com/sinkinben/pic-go/raw/master/img/20201029210110.png" style="width:67%;" />

**DFS 代码**

```cpp
class Solution {
public:
    unordered_map<Node*, Node*> m;
    Node* copyRandomList(Node* head) { return dfs(head); }
    Node *dfs(Node *p)
    {
        if (p == nullptr) return nullptr;
        if (m.count(p) != 0) return m[p];
        auto q = new Node(p->val);
        m[p] = q;
        q->next = dfs(p->next), q->random = dfs(p->random);
        return q;
    }
};
```

**BFS 代码**

```cpp
class Solution {
public:
    Node* copyRandomList(Node* head) 
    {
        if (head == nullptr) return nullptr;
        auto newHead = new Node(head->val);
        queue<Node*> q;
        unordered_map<Node*, Node*> visited = {{nullptr, nullptr}};
        q.push(head);
        visited[head] = newHead;
        while (!q.empty())
        {
            auto p = q.front();
            q.pop();
            if (visited.count(p->next) == 0)
                visited[p->next] = new Node(p->next->val), q.push(p->next);
            if (visited.count(p->random) == 0)
                visited[p->random] = new Node(p->random->val), q.push(p->random);
            visited[p]->next = visited[p->next];
            visited[p]->random = visited[p->random];
        }
        return newHead;
    }
};
```

