## 0401节点间通路

题目：[面试题 04.01. 节点间通路](https://leetcode-cn.com/problems/route-between-nodes-lcci/) 。

DFS 水题。我在想，如果是无向图，是不是可以用并查集？

```cpp
class Solution {
public:
    vector<vector<int>> g;
    vector<bool> visited;
    bool findWhetherExistsPath(int n, vector<vector<int>>& graph, int start, int target) {
        g.resize(n, vector<int>());
        visited.resize(n, 0);
        for (auto &v: graph)
            g[v[0]].emplace_back(v[1]);
        return dfs(n, start, target);
    }

    bool dfs(int n, int cur, int target)
    {
        visited[cur] = true;
        if (cur == target) return true;
        for(int x: g[cur])
        {
            if (!visited[x] && dfs(n, x, target))
                return true;
        }
        return false;
    }
};
```

## 0402 最小高度树

题目：[面试题 04.02. 最小高度树](https://leetcode-cn.com/problems/minimum-height-tree-lcci/).

递归。

```cpp
class Solution {
public:
    TreeNode* sortedArrayToBST(vector<int>& nums) {
        return createBST(nums, 0, nums.size()-1);
    }

    TreeNode* createBST(const vector<int> &v, int l, int r)
    {
        if (l > r) return nullptr;
        int m = l + (r - l) / 2;
        auto root = new TreeNode(v[m]);
        root->left = createBST(v, l, m-1);
        root->right = createBST(v, m+1, r);
        return root; 
    }
};
```

## 0403 特定深度节点链表

题目：[面试题 04.03. 特定深度节点链表](https://leetcode-cn.com/problems/list-of-depth-lcci/) 。

层次遍历。由于 `queue` 是不支持迭代的，因此使用 `deque` .

```cpp
class Solution {
public:
    vector<ListNode*> listOfDepth(TreeNode* tree) {
        if (tree == nullptr) return {};
        deque<TreeNode*> q;
        q.push_back(tree);
        vector<ListNode*> result;
        result.push_back(createlist(q));
        while (!q.empty())
        {
            deque<TreeNode*> next;
            while (!q.empty())
            {
                auto node = q.front();
                q.pop_front();
                if (node->left) next.push_back(node->left);
                if (node->right) next.push_back(node->right);
            }
            auto list = createlist(next);
            if (list) result.push_back(list);
            q = next;
        }
        return result;
    }
    ListNode* createlist(const deque<TreeNode*> &q)
    {
        auto *head = new ListNode(-1);
        auto p = head;
        for (auto x: q)
            p->next = new ListNode(x->val), p = p->next;
        return head->next;
    }
};
```

## 0404 检查平衡性

题目：[面试题 04.04. 检查平衡性](https://leetcode-cn.com/problems/check-balance-lcci/) 。

后序遍历。通过一个 `map` 来记录节点的高度，结合后序遍历，减少高度的重复计算。计算各个节点的高度遍历了一次树，求解是否平衡又需要一次后序遍历，因此这里一共遍历了 2 次。时间与空间复杂度均为 $O(N)$ .

```cpp
class Solution {
public:
    unordered_map<TreeNode*, int> table = {{nullptr, 0}};
    bool result = true;
    bool isBalanced(TreeNode* root) 
    {
        postorder(root);
        return result;
    }
    int height(TreeNode *ptr)
    {
        if (table.count(ptr)) return table[ptr];
        return table[ptr] = 1 + max(height(ptr->left), height(ptr->right));
    }
    void postorder(TreeNode *ptr)
    {
        if (!result || ptr == nullptr) return;
        postorder(ptr->left), postorder(ptr->right);
        int lh = height(ptr->left), rh = height(ptr->right);
        result = result && abs(lh - rh) <= 1;
    }
};
```

只用一次后序遍历的解法。

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
        if (isBalanced(p->left, ldepth) &&  isBalanced(p->right, rdepth) && abs(ldepth - rdepth) <= 1)
        {
            depth = 1 + max(ldepth, rdepth);
            return true;
        }
        return false;
    }
};
```



## 0405 合法二叉搜索树

题目：[面试题 04.05. 合法二叉搜索树](https://leetcode-cn.com/problems/legal-binary-search-tree-lcci/)

中序递归遍历，检查当前节点是否严格大于前一个节点。

当然也可以使用 `vector` 缓存 N 个节点的数据。

```cpp
class Solution {
public:
    int preval = INT_MIN;
    bool first = true;
    bool isValidBST(TreeNode* root) {
        return inorder(root);
    }
    bool inorder(TreeNode *p)
    {
        if (p == nullptr) return true;
        bool l = inorder(p->left);
        if (first)  preval = p->val, first=false;
        else if (p->val <= preval) return false;
        preval = p->val;
        bool r = inorder(p->right);
        return l&&r;
    }
};
```

非递归：

```cpp
class Solution {
public:
    bool isValidBST(TreeNode* root) {
        if (root == nullptr) return 1;
        stack<TreeNode*> s;
        auto ptr = root;
        int preval = 0x80000000;
        bool first = 1;
        while (!s.empty() || ptr != nullptr)
        {
            if (ptr) s.push(ptr), ptr = ptr->left;
            else
            {
                ptr = s.top(), s.pop();
                if (!first)
                {
                    if (preval >= ptr->val) return false;
                    preval = ptr->val;
                }
                else
                    preval = ptr->val, first = 0;
                ptr = ptr->right;
            }
        }
        return true;

    }
};
```

## 0406 后继者

题目：[面试题 04.06. 后继者](https://leetcode-cn.com/problems/successor-lcci/) 。

中序遍历解法：

```cpp
class Solution {
public:
    TreeNode* inorderSuccessor(TreeNode* root, TreeNode* p) 
    {
        if (root == nullptr || p == nullptr) return nullptr;
        stack<TreeNode*> s;
        auto ptr = root;
        int preval, first = 1;
        while (!s.empty() || ptr)
        {
            if (ptr) s.push(ptr), ptr = ptr->left;
            else
            {
                ptr = s.top(), s.pop();
                if (!first)
                {
                    if (preval == p->val) return ptr;
                    preval = ptr->val;
                }
                else
                    preval = ptr->val, first = 0;
                ptr = ptr->right;
            }
        }
        return nullptr;
    }
};
```

二分查找：

```cpp
class Solution {
public:
    TreeNode* inorderSuccessor(TreeNode* root, TreeNode* p) 
    {
        auto cur = root;
        TreeNode* result = nullptr;
        while (cur)
        {
            if (cur->val <= p->val) cur = cur->right;
            else result = cur, cur = cur->left;
        }
        return result;
    }
};
```

## 0407 最近公共祖先

题目：[面试题 04.08. 首个共同祖先](https://leetcode-cn.com/problems/first-common-ancestor-lcci/) 

剑指 Offer 原题：https://www.cnblogs.com/sinkinben/p/13936418.html

```cpp
class Solution {
public:
    TreeNode* lowestCommonAncestor(TreeNode* root, TreeNode* p, TreeNode* q) 
    {
        return lca(root, p, q);
    }
    TreeNode* lca(TreeNode *r, TreeNode *p, TreeNode *q)
    {
        if (r == nullptr || r == p || r == q) return r;
        auto left = lca(r->left, p, q), right = lca(r->right, p, q);
        if (left && right) return r;
        return left ? left : right;
    }
};
```

