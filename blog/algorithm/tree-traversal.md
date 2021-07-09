## 树的遍历

Leetcode 题目总结：

- [144. 二叉树的前序遍历](https://leetcode-cn.com/problems/binary-tree-preorder-traversal/)
- [94. 二叉树的中序遍历](https://leetcode-cn.com/problems/binary-tree-inorder-traversal/)
- [145. 二叉树的后序遍历](https://leetcode-cn.com/problems/binary-tree-postorder-traversal/)
- [102. 二叉树的层序遍历](https://leetcode-cn.com/problems/binary-tree-level-order-traversal/)
- [589. N叉树的前序遍历](https://leetcode-cn.com/problems/n-ary-tree-preorder-traversal/)
- [590. N叉树的后序遍历](https://leetcode-cn.com/problems/n-ary-tree-postorder-traversal/)
- [429. N 叉树的层序遍历](https://leetcode-cn.com/problems/n-ary-tree-level-order-traversal/)

感觉这是我写的无敌简洁的一版代码了，同时把难搞的后序遍历套上了前序的模版，思路更加清晰。

## 二叉树

二叉树的三序遍历。

**前序遍历**

先序：`root->left->right`，因此 `left` 需要放在栈顶。

```cpp
class Solution {
public:
    vector<int> preorderTraversal(TreeNode* root) 
    {
        vector<int> res;
        if (root == nullptr) return res;
        stack<TreeNode*> s;
        s.push(root);
        while (!s.empty())
        {
            auto p = s.top();
            s.pop();
            res.push_back(p->val);
            if (p->right) s.push(p->right);
            if (p->left) s.push(p->left);
        }
        return res;
    }
};
```

**中序遍历**

极致压缩代码行数。需要注意的是：中序遍历，进入循环前不需要 `s.push(p)` 。

```cpp
class Solution {
public:
    vector<int> inorderTraversal(TreeNode* root) {
        vector<int> res;
        auto p = root;
        stack<TreeNode*> s;
        while (!s.empty() || p)
        {
            if (p) s.push(p), p = p->left;
            else p = s.top(), s.pop(), res.push_back(p->val), p = p->right;
        }
        return res;
    }
};
```

**后序遍历**

逆向考虑顺序：`root, right, left` ，这样可以套上先序遍历的模版（最后倒转 `vector`）。

```cpp
class Solution {
public:
    vector<int> postorderTraversal(TreeNode* root) 
    {
        // postorder: left, right, root
        // first: root, right, left, which is similar to preorder
        // then: reverse the vector
        vector<int> res;
        if (root == nullptr) return res;
        auto p = root;
        stack<TreeNode*> s;
        s.push(p);
        while (!s.empty())
        {
            p = s.top(), s.pop();
            res.push_back(p->val);
            if (p->left) s.push(p->left);
            if (p->right) s.push(p->right);
        }
        reverse(res.begin(), res.end());
        return res;
    }
};
```

**层次遍历**

```cpp
class Solution {
public:
    vector<vector<int>> levelOrder(TreeNode* root) {
        vector<vector<int>> res;
        if (root == nullptr) return res;
        auto p = root;
        queue<TreeNode*> q;
        q.push(p);
        while (!q.empty())
        {
            vector<int> cur;
            queue<TreeNode*> next;
            while (!q.empty())
            {
                p = q.front(), q.pop();
                cur.push_back(p->val);
                if (p->left) next.push(p->left);
                if (p->right) next.push(p->right);
            }
            q = next;
            res.push_back(cur);
        }
        return res;
    }
};
```



## 树

**先序**

最左孩子放在栈顶。

```cpp
class Solution {
public:
    vector<int> preorder(Node* root) {
        vector<int> res;
        if (root == nullptr) return res;
        auto p = root;
        stack<Node*> s;
        s.push(p);
        while (!s.empty())
        {
            p = s.top(), s.pop();
            res.push_back(p->val);
            auto &children = p->children;
            for (auto x = children.rbegin(); x!=children.rend(); x++)
                s.push(*x);
        }
        return res;
    }
};
```

**后序**

```cpp
class Solution {
public:
    vector<int> postorder(Node* root) 
    {
        // postorder: child[l, ..., r] -> root
        // first: root -> child[r, ..., l], which is similar to preorder
        // then: reverse the vector
        vector<int> res;
        if (root == nullptr) return res;
        auto p = root;
        stack<Node*> s;
        s.push(p);
        while (!s.empty())
        {
            p = s.top(), s.pop();
            res.push_back(p->val);
            for (auto x: p->children) s.push(x);
        } 
        reverse(res.begin(), res.end());
        return res;
    }
};
```

**层序**

```cpp
class Solution {
public:
    vector<vector<int>> levelOrder(Node* root) {
        vector<vector<int>> res;
        if (root == nullptr) return res;
        auto p = root;
        queue<Node*> q;
        q.push(p);
        while (!q.empty())
        {
            vector<int> cur;
            queue<Node*> next;
            while (!q.empty())
            {
                p = q.front(), q.pop();
                cur.push_back(p->val);
                for (auto x: p->children) next.push(x);
            }
            q = next;
            res.push_back(cur);
        }
        return res;
    }
};
```

