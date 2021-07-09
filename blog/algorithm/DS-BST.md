## 二叉搜索树

Github Repo：https://github.com/sinkinben/DataStructure.git

二叉搜索树（Binary Search Tree）定义：

> 左子树的 Key 值不大于根的 Key 值，右子树的 Key 值不小于根的 Key 值。如果限制 Key 值是唯一的，那么就有：`left.key < root.key < right.key` 。

性质：中序遍历所得序列呈升序（非降序）。

本文实现 API：

+ selfCheck：检查当前的 BST 是否满足其性质。
+ search：根据关键字 val 查找某一节点。
+ insert：插入节点。
+ remove：删除节点
+ update：更新某一节点（实质是先删后加）。
+ maxval：BST 中的最大值。
+ minval：BST 中的最小值。
+ successor：查找某一节点的后继（即中序遍历中的下一个）。
+ precedessor：查找某一节点的先驱（即中序遍历的前一个）。

### selfCheck

本函数用于 Debug，因此不考虑复杂度。

先获取整个中序序列，检查是否为升序。

```cpp
void selfCheck()
{
    auto v = this->inorder();
    int n = v.size();
    for (int i = 1; i < n; i++)
        assert(v[i] > v[i - 1]);
}
```

### search

根据 BST 的性质，如果插入的 val 小于当前 x.val，则向左子树寻找，否则向右子树寻找。

```cpp
// search val by non-recursion
TreeNode<T> *search(T val)
{
    auto p = this->root;
    while (p != nullptr && p->val != val)
    {
        if (val < p->val)
            p = p->left;
        else
            p = p->right;
    }
    return p;
}
```

递归形式：

```cpp
// search val by recursion
TreeNode<T> *innerSearch(TreeNode<T> *p, T val)
{
    if (p == nullptr || p->val == val)
        return p;
    if (val < p->val)
        return innerSearch(p->left, val);
    else
        return innerSearch(p->right, val);
}
```



### insert

根据 BST 的性质，如果插入的 val 小于当前 x.val，则向左子树寻找插入位置，否则向右子树寻找。利用指针 y 记住 当前查找节点 x 的 parent，以方便后面的插入。

```cpp
// insert a new node into the BST, whose value is 'val'
// each node in BST is unique
// return nullptr means 'val' is already in BST
TreeNode<T> *insert(T val)
{
    if (this->root == nullptr)
        return this->root = new TreeNode<T>(val);
    auto x = this->root;
    auto y = x->parent; // nullptr actually
    while (x != nullptr)
    {
        y = x;
        if (val < x->val)
            x = x->left;
        else if (val > x->val)
            x = x->right;
        else
            return nullptr;
    }
    auto node = new TreeNode<T>(val);
    node->parent = y;
    return val < y->val ? (y->left = node) : (y->right = node);
}
```

### remove

首先实现一个移植函数 `transplant` ，该函数的作用是把子树 `u` 替换为子树 `v` 。即：

```
    p                       p
   / \                     / \
  u   pr      ==>         v   pr
 / \                     / \
ul  ur                  vl  vr
```

代码：

```cpp
// remove helper
// replace subtree-u with subtree-v
void transplant(TreeNode<T> *u, TreeNode<T> *v)
{
    if (u == nullptr)
        return;
    // u is the root
    if (u->parent == nullptr)
        this->root = v;
    else if (u == u->parent->left)
        u->parent->left = v;
    else
        u->parent->right = v;
    if (v != nullptr)
        v->parent = u->parent;
}
```

根据删除节点 `node` 需要分情况讨论：

<img src="https://gitee.com/sinkinben/pic-go/raw/master/img/20200921203852.png" alt="image-20200921203659172" style="width: 67%;" />

图中的 z 就是下面所说的 node 。

1. 没有左子树：直接用 `node` 的右子树替换 `node` ，右子树可以为  NULL 。如上图 (a) 。
2. 只有左子树：用 `node` 的左子树替换 `node`。如上图 (b) 。
3. 左右子树均有，那么用右子树的最小值 `y` （可以确定 y 没有左子树）替换 `node`，这时需要根据 `y` 是否为 `node` 的右孩子进行讨论：
   + y 是 node 的右孩子：如图 (c)，直接 y 替换 node 。
   + y 不是 node 的右孩子：如图 (d)，先使用 x 替换 y ，然后让 node 的右子树 r 成为 y 的右子树，**最后使用变化后的 y 替换 node （该步骤与上一步相同）** 。

`remove` 代码：

```cpp
// remove node from the BST, whose value is 'val'
bool remove(T val)
{
    auto node = search(val);
    if (node == nullptr)
        return false;
    bool l = (node->left != nullptr);
    bool r = (node->right != nullptr);
    // case '!l' include (!l && !r) and (!l && r)
    if (!l)
        transplant(node, node->right);
    else if (l && !r)
        transplant(node, node->left);
    else
    {
        auto y = minval(node->right);
        if (y->parent != node)
        {
            transplant(y, y->right);
            y->right = node->right;
            y->right->parent = y;
        }
        transplant(node, y);
        y->left = node->left;
        y->left->parent = y;
    }

    delete node;
    return true;
}
```

### update

先删除后添加。

```cpp
// update oldval to newval if oldval is in the BST, otherwise do nothing
TreeNode<T> *update(T oldval, T newval)
{
    if (!remove(oldval))
        return nullptr;
    return insert(newval);
}
```

### maxval/minval

maxval 是位于 BST 最右端的孩子，minval 是最左端的孩子。

```cpp
// get max val in the BST
TreeNode<T> *maxval(TreeNode<T> *p)
{
    if (p == nullptr)
        return nullptr;
    while (p->right != nullptr)
        p = p->right;
    return p;
}
TreeNode<T> *maxval() { return maxval(this->root); }

// get min val in the BST
TreeNode<T> *minval(TreeNode<T> *p)
{
    if (p == nullptr)
        return nullptr;
    while (p->left != nullptr)
        p = p->left;
    return p;
}
TreeNode<T> *minval() { return minval(this->root); }

```

### successor

查找节点 `x` 的后继节点。

如果 `x` 有右子树，那么 `minval(x->right)` 即为所求。

如果没有，那么 x 的后继是**最接近 x 的祖先，并且该祖先的左孩子也是 x 的祖先**。

如图：

```
    19
   /  \
  7    20
 / \
6   8
     \
      9
       \
        10
       /
    ...
```

上述二叉树的中序遍历为：`6, 7, 8, 9, ..., 10, 19, 20` . 节点 10 的后继是 19 .

```cpp
/* get the node's successor, whose value is 'val'
     * if the right subtree of a node x in T is empty and x has a successor y, 
     * then y is the lowest ancestor of x whose left child is also an ancestor of x. 
     * (Recall that every node is its own ancestor.)
     */
TreeNode<T> *successor(T val)
{
    auto x = search(val);
    if (x == nullptr)
        return nullptr;
    if (x->right != nullptr)
        return minval(x->right);
    auto y = x->parent;
    while (y != nullptr && y->right == x)
        x = y, y = y->parent;
    return y;
}
```

### precedessor

与 successor 同理。

```cpp
// get the node's predecessor, whose value is 'val'
TreeNode<T> *predecessor(T val)
{
    auto x = search(val);
    if (x == nullptr)
        return nullptr;
    if (x->left != nullptr)
        return maxval(x->left);
    auto y = x->parent;
    while (y != nullptr && y->left == x)
        x = y, y = y->parent;
    return y;
}
```

