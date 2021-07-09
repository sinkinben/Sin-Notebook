## 面向对象的二叉树的实现

最近在重新实现各种树的数据结构（BST、AVL等等），其中最基本的就是二叉树的实现。

Github Repo：https://github.com/sinkinben/DataStructure.git

本次的实现包括：

+ 根据序列化字符串建树
+ 非递归三序遍历
+ 在终端图形化显示

## 建树

节点定义：

```cpp
template <typename T>
class TreeNode
{
private:
    // (x, y) is used to draw the binary tree in the console
    // which is the node's coordinate on the screen
    int x, y;

public:
    T val;
    TreeNode<T> *left, *right, *parent;
    // 'color' variable is due to the implementation of the red and black tree.
    RBColor color;
    // 'factor variable is due to the implementation of the AVLTree.
    AVLFactor factor;

public:
    TreeNode() { left = right = parent = this; }
    TreeNode(T _val)
    {
        val = _val;
        left = right = parent = nullptr;
        x = y = -1;
    }

    TreeNode(T _val, TreeNode<T> *_parent)
    {
        val = _val;
        parent = _parent;
        left = right = nullptr;
    }

    // This construct function is used for RBTree.hpp
    TreeNode(T _val, TreeNode<T> *_parent, TreeNode<T> *_left, TreeNode<T> *_right, RBColor _color)
    {
        val = _val, parent = _parent, left = _left, right = _right;
        color = _color;
    }

    // This construct function is uedf for AVLTree.hpp
    TreeNode(T _val, AVLFactor _factor, TreeNode<T> *_parent)
    {
        val = _val, factor = _factor;
        left = right = nullptr;
        parent = _parent;
    }

    int getx() { return x; }
    int gety() { return y; }
    void setx(int _x) { x = _x; }
    void sety(int _y) { y = _y; }
};
```

请参考文章[二叉树的序列化与反序列化](https://www.cnblogs.com/sinkinben/p/13668996.html) ，其中「反序列化」过程就是建树。

## 非递归三序遍历

See the fucking source code below.

```cpp
std::vector<T> inorder()
{
    std::vector<T> v;
    if (root == nullptr)
        return v;
    std::stack<TreeNode<T> *> s;
    auto p = root;
    while (!s.empty() || p != nullptr)
    {
        if (p != nullptr)
        {
            s.push(p);
            p = p->left;
        }
        else
        {
            p = s.top(), s.pop();
            v.push_back(p->val);
            p = p->right;
        }
    }
    return v;
}

std::vector<T> preorder()
{
    std::vector<T> v;
    if (root == nullptr)
        return v;
    std::stack<TreeNode<T> *> s;
    auto p = root;
    s.push(p);
    while (!s.empty())
    {
        p = s.top(), s.pop();
        v.emplace_back(p->val);
        if (p->right != nullptr)
            s.push(p->right);
        if (p->left != nullptr)
            s.push(p->left);
    }
    return v;
}

std::vector<T> postorder()
{
    std::vector<T> v;
    if (root == nullptr)
        return v;
    auto p = root;
    std::stack<TreeNode<T> *> s;
    s.push(p);
    while (!s.empty())
    {
        p = s.top(), s.pop();
        v.push_back(p->val);
        if (p->left != nullptr)
            s.push(p->left);
        if (p->right != nullptr)
            s.push(p->right);
    }
    std::reverse(v.begin(), v.end());
    return v;
}
```



## 图形化显示

定义坐标：左上角为 `(0,0)` 往右是 X 坐标，往下是 Y 坐标。

二叉树节点坐标：

+ X 坐标：节点在中序遍历序列中的位置（参考 `initX` 函数）。
+ Y 坐标：节点所在的二叉树的层次（参考 `initY` 函数）。

绘制过程参考 `draw` 函数。

## 完整代码

```cpp
#ifndef BINARYTREE_H
#define BINARYTREE_H
template <typename T>
class BinaryTree
{
protected:
    const std::string NIL = "null";
    const std::string SEPARATOR = ",";
    TreeNode<T> *root;

private:
    TreeNode<T> *generateNode(const std::string &s, TreeNode<T> *parent)
    {
        return s == NIL ? nullptr : new TreeNode<T>(std::stoi(s), parent);
    }
    TreeNode<T> *innerBuild(const std::vector<std::string> &v)
    {
        if (v.size() == 0)
            return nullptr;
        int idx = 0, len = v.size();
        TreeNode<T> *root = new TreeNode<T>(stoi(v[idx++]));
        assert(root != nullptr);
        std::queue<TreeNode<T> *> q;
        q.push(root);
        while (!q.empty())
        {
            auto node = q.front();
            q.pop();
            if (idx < len)
                node->left = generateNode(v[idx++], node);
            if (idx < len)
                node->right = generateNode(v[idx++], node);
            if (node->left != nullptr)
                q.push(node->left);
            if (node->right != nullptr)
                q.push(node->right);
        }
        return root;
    }

    void destroy(TreeNode<T> *subtree)
    {
        if (subtree == nullptr)
            return;
        std::stack<TreeNode<T> *> result;
        auto p = subtree;
        std::stack<TreeNode<T> *> s;
        s.push(p);
        while (!s.empty())
        {
            p = s.top(), s.pop();
            result.push(p);
            if (p->left != nullptr)
                s.push(p->left);
            if (p->right != nullptr)
                s.push(p->right);
        }
        while (!result.empty())
        {
            p = result.top(), result.pop();
            if (p != nullptr)
                delete p;
        }
    }

    void initCoordinate()
    {
        int x = 0;
        initXCoordinate(this->root, x);
        initYCoordinate();
    }
    void initXCoordinate(TreeNode<T> *currentNode, int &x)
    {
        if (currentNode == nullptr)
            return;
        initXCoordinate(currentNode->left, x);
        currentNode->setx(x++);
        initXCoordinate(currentNode->right, x);
    }
    void initYCoordinate()
    {
        if (this->root == nullptr)
            return;
        int level = 1;
        std::queue<TreeNode<T> *> q;
        q.push(root);
        while (!q.empty())
        {
            std::queue<TreeNode<T> *> next;
            while (!q.empty())
            {
                auto p = q.front();
                q.pop();
                p->sety(level);
                if (p->left != nullptr)
                    next.push(p->left);
                if (p->right != nullptr)
                    next.push(p->right);
            }
            q = next;
            level++;
        }
    }

public:
    BinaryTree()
    {
        root = nullptr;
    }

    ~BinaryTree()
    {
        destroy(this->root);
        root = nullptr;
    }

    TreeNode<T> *getRoot() { return this->root; }

    // 用于通过序列化字符串创建树，节点值 val 需要使用 int 表示
    void buildTree(std::string str)
    {
        if (str.front() == '[' && str.back() == ']')
            str = str.substr(1, str.length() - 2);

        // replace "," with " "
        auto pos = str.find(",");
        while (pos != std::string::npos)
        {
            str.replace(pos, 1, " ");
            pos = str.find(",");
        }

        // make str into vector
        std::stringstream ss(str);
        std::string buf;
        std::vector<std::string> result;
        while (ss >> buf)
            result.emplace_back(buf);

        this->root = innerBuild(result);
    }

    void draw(int widthZoom = 3)
    {
        if (root == nullptr)
            return;

        initCoordinate();

        auto printChars = [](char c, int n) { for (; n > 0; n--) std::cout << c; };

        typedef std::tuple<int, int, char> Tuple;
        std::queue<TreeNode<T> *> q;
        std::string sval;
        int x, y, val;

        q.push(root);
        while (!q.empty())
        {
            std::queue<TreeNode<T> *> next;
            std::vector<Tuple> tuples;
            int idx = 0;
            while (!q.empty())
            {
                auto p = q.front();
                q.pop();
                bool l = (p->left != nullptr), r = (p->right != nullptr);
                x = p->getx() * widthZoom, y = p->gety(), val = p->val, sval = std::to_string(val);
                printChars(' ', x - idx), std::cout << val;
                idx = x + sval.length();
                if (l)
                {
                    next.push(p->left);
                    int a = widthZoom * p->left->getx();
                    int b = x + sval.length() / 2 - 1;
                    tuples.emplace_back(Tuple(a, b, '_'));
                }
                if (l || r)
                {
                    int a = x + sval.length() / 2;
                    tuples.emplace_back(Tuple(a, a, '|'));
                }
                if (r)
                {
                    next.push(p->right);
                    int a = x + sval.length() / 2 + 1;
                    int b = widthZoom * p->right->getx() + std::to_string(p->right->val).length() - 1;
                    tuples.push_back(Tuple(a, b, '_'));
                }
            }
            std::cout << '\n';
            idx = 0;
            if (!tuples.empty())
            {
                for (auto &t : tuples)
                {
                    int a = std::get<0>(t), b = std::get<1>(t);
                    char c = std::get<2>(t);
                    for (; idx < a; idx++)
                        std::cout << ' ';
                    printChars(c, b - a + 1);
                    idx = b + 1;
                }
            }
            std::cout << '\n';
            q = next;
        }
    }

    void destroy()
    {
        this->destroy(this->root);
        this->root = nullptr;
    }
};
#endif
```

