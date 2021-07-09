## 红黑树

Github Repo：https://github.com/sinkinben/DataStructure.git

本文部分内容参考《算法导论》。

这篇博客也写的很详细：https://www.cnblogs.com/skywang12345/p/3245399.html

红黑树（Red Black Tree, RBTree）的定义：

>一颗红黑树是满足下面性质的二叉搜索树：
>
>1. 每个节点是红色或者黑色的。
>2. 根是黑色的。
>3. 每个叶子节点均为黑色。注：叶子结点是指哨兵节点 NIL，见下图(a) 。
>4. 如果某个节点是红色的，其左右孩子为黑色。
>5. 对于任意节点，从该节点到 NIL 的所有路径上，每个路径包含相同数目的黑色节点。

其他重要性质：

> 1. 红黑树的查找、插入、删除的时间复杂度最坏为 $O(lgn)$ .
> 2. $n$ 个内部节点（非 NIL 节点）的红黑树的高度至多为 $2lg(n+1)$ .

<img src="https://gitee.com/sinkinben/pic-go/raw/master/img/20200923200645.png" style="width:67%;" />

此处实现的 RBTree 采用上图 (b) 的方式，即只使用一个哨兵节点。

懒得写了，RTFSC。

```cpp
template <typename T>
class RBTree
{
private:
    TreeNode<T> *const NIL = new TreeNode<T>();
    const std::string SEPARATOR = ",";
    TreeNode<T> *root;

    void leftRotate(TreeNode<T> *x)
    {
        auto y = x->right;

        // move y.left to x.right
        x->right = y->left;
        if (x->right != NIL)
            x->right->parent = x;

        // move y to x's position
        y->parent = x->parent;
        if (x->parent == NIL)
            this->root = y;
        else if (x->parent->left == x)
            x->parent->left = y;
        else
            x->parent->right = y;

        // let x to be y.left
        y->left = x;
        x->parent = y;
    }

    void rightRoate(TreeNode<T> *y)
    {

        auto x = y->left;

        // move x.right to y.left
        y->left = x->right;
        if (y->left != NIL)
            y->left->parent = y;

        // move x to y's position
        x->parent = y->parent;
        if (y->parent == NIL)
            this->root = x;
        else if (y->parent->left == y)
            y->parent->left = x;
        else
            y->parent->right = x;

        // let y to be x.right
        x->right = y;
        y->parent = x;
    }

    void destroy(TreeNode<T> *subtree)
    {
        if (subtree == NIL)
            return;
        std::stack<TreeNode<T> *> result;
        auto p = subtree;
        std::stack<TreeNode<T> *> s;
        s.push(p);
        while (!s.empty())
        {
            p = s.top(), s.pop();
            result.push(p);
            if (p->left != NIL)
                s.push(p->left);
            if (p->right != NIL)
                s.push(p->right);
        }
        while (!result.empty())
        {
            p = result.top(), result.pop();
            if (p != NIL)
                delete p;
        }
    }

    void removeFixup(TreeNode<T> *x)
    {
        TreeNode<T> *w;
        while (x != root && x->color == RBColor::Black)
        {
            if (x == x->parent->left)
            {
                w = x->parent->right;
                if (w->color == RBColor::Red)
                {
                    w->color = RBColor::Black;
                    x->parent->color = RBColor::Red;
                    leftRotate(x->parent);
                    w = x->parent->right;
                }
                if (w->left->color == RBColor::Black && w->right->color == RBColor::Black)
                {
                    w->color = RBColor::Red;
                    x = x->parent;
                }
                else
                {
                    if (w->right->color == RBColor::Black)
                    {
                        w->left->color = RBColor::Black;
                        w->color = RBColor::Red;
                        rightRoate(w);
                        w = x->parent->right;
                    }
                    w->color = x->parent->color;
                    x->parent->color = RBColor::Black;
                    w->right->color = RBColor::Black;
                    leftRotate(x->parent);
                    x = root;
                }
            }
            else
            {
                // same as above the clause with 'right' and 'left' exchanged
                w = x->parent->left;
                if (w->color == RBColor::Red)
                {
                    w->color = RBColor::Black;
                    x->parent->color = RBColor::Red;
                    rightRoate(x->parent);
                    w = x->parent->left;
                }
                if (w->right->color == RBColor::Black && w->left->color == RBColor::Black)
                {
                    w->color = RBColor::Red;
                    x = x->parent;
                }
                else
                {
                    if (w->left->color == RBColor::Black)
                    {
                        w->right->color = RBColor::Black;
                        w->color = RBColor::Red;
                        leftRotate(w);
                        w = x->parent->left;
                    }
                    w->color = x->parent->color;
                    x->parent->color = RBColor::Black;
                    x->left->color = RBColor::Black;
                    rightRoate(x->parent);
                    x = root;
                }
            }
            x->color = RBColor::Black;
        }
    }

    void insertFixup(TreeNode<T> *z)
    {
        while (z->parent->color == RBColor::Red)
        {
            if (z->parent == z->parent->parent->left)
            {
                auto y = z->parent->parent->right;
                if (y->color == RBColor::Red)
                {
                    z->parent->color = RBColor::Black;
                    y->color = RBColor::Black;
                    z->parent->parent->color = RBColor::Red;
                    z = z->parent->parent;
                }
                else
                {
                    if (z == z->parent->right)
                    {
                        z = z->parent;
                        leftRotate(z);
                    }
                    z->parent->color = RBColor::Black;
                    z->parent->parent->color = RBColor::Red;
                    rightRoate(z->parent->parent);
                }
            }
            else
            {
                auto y = z->parent->parent->left;
                if (y->color == RBColor::Red)
                {
                    z->parent->color = RBColor::Black;
                    y->color = RBColor::Black;
                    z->parent->parent->color = RBColor::Red;
                    z = z->parent->parent;
                }
                else
                {
                    if (z == z->parent->left)
                    {
                        z = z->parent;
                        rightRoate(z);
                    }
                    z->parent->color = RBColor::Black;
                    z->parent->parent->color = RBColor::Red;
                    leftRotate(z->parent->parent);
                }
            }
        }
        this->root->color = RBColor::Black;
    }

    void initx(TreeNode<T> *p, int &x)
    {
        if (p == NIL)
            return;
        initx(p->left, x);
        p->setx(x++);
        initx(p->right, x);
    }

    void inity()
    {
        if (root == NIL)
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
                if (p->left != NIL)
                    next.push(p->left);
                if (p->right != NIL)
                    next.push(p->right);
            }
            q = next;
            level++;
        }
    }

    void transplant(TreeNode<T> *u, TreeNode<T> *v)
    {
        // if (u == NIL)
        //     return;
        if (u->parent == NIL)
            root = v;
        else if (u == u->parent->left)
            u->parent->left = v;
        else
            u->parent->right = v;
        v->parent = u->parent;
    }

public:
    RBTree()
    {
        NIL->color = RBColor::Black;
        this->root = NIL;
    }
    ~RBTree()
    {
        destroy(root);
        delete NIL;
    }

    TreeNode<T> *insert(T val)
    {
        auto y = NIL, x = this->root;
        while (x != NIL)
        {
            y = x;
            if (val < x->val)
                x = x->left;
            else if (val > x->val)
                x = x->right;
            else
                return NIL;
        }
        auto z = new TreeNode<T>(val, y, NIL, NIL, RBColor::Red);
        if (y == NIL)
            this->root = z;
        else if (val < y->val)
            y->left = z;
        else
            y->right = z;
        insertFixup(z);
        return z;
    }

    void remove(T val)
    {
        TreeNode<T> *x, *y, *z;
        z = search(val);
        y = z;
        RBColor ycolor = y->color;
        if (z->left == NIL)
        {
            x = z->right;
            transplant(z, z->right);
        }
        else if (z->right == NIL)
        {
            x = z->left;
            transplant(z, z->left);
        }
        else
        {
            y = minval(z->right);
            ycolor = y->color;
            x = y->right;
            if (y->parent == z)
                x->parent = y;
            else
            {
                transplant(y, y->right);
                y->right = z->right;
                y->right->parent = y;
            }
            transplant(z, y);
            y->left = z->left;
            y->left->parent = y;
            y->color = z->color;
        }
        if (ycolor == RBColor::Black)
            removeFixup(x);
    }

    TreeNode<T> *search(T val)
    {
        auto p = root;
        while (p != NIL && val != p->val)
            p = (val < p->val) ? p->left : p->right;
        return p;
    }

    TreeNode<T> *minval(TreeNode<T> *substree)
    {
        auto p = substree;
        while (p->left != NIL)
            p = p->left;
        return p;
    }
    TreeNode<T> *minval() { return minval(root); }

    std::vector<T> inorder()
    {
        std::vector<T> v;
        if (root == NIL)
            return v;
        std::stack<TreeNode<T> *> s;
        auto p = root;
        while (!s.empty() || p != NIL)
        {
            if (p != NIL)
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

    TreeNode<T> *getRoot() { return root; }

    TreeNode<T> *getNil() { return NIL; }

    void buildTree(std::string str)
    {
        if (str.front() == '[' & str.back() == ']')
            str = str.substr(1, str.length() - 2);
        std::vector<std::string> list;
        size_t l = 0, r = str.find(SEPARATOR, l);
        while (r != std::string::npos)
        {
            list.emplace_back(str.substr(l, r - l));
            l = r + SEPARATOR.length();
            r = str.find(SEPARATOR, l);
        }
        if (l < str.length())
            list.emplace_back(str.substr(l));
        for (std::string &x : list)
            insert(std::stoi(x));
    }

    void draw(int widthzoom = 3)
    {
        if (root == NIL)
            return;

        // print function with color
        auto printNode = [](RBColor color, T val) {
            if (color == RBColor::Red)
                std::cout << "\033[41m" << val << "\033[0m";
            else if (color == RBColor::Black)
                std::cout << "\033[40m" << val << "\033[0m";
            else
                assert(0);
        };
        // print n chars
        auto printChars = [](char c, int n) {for(;n>0;n--) std::cout << c; };

        typedef std::tuple<int, int, char> Tuple;

        int x = 0;
        initx(root, x);
        inity();

        std::queue<TreeNode<T> *> q;
        int y, val;
        std::string sval;
        RBColor color;

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
                color = p->color;
                x = p->getx() * widthzoom, y = p->gety(), val = p->val, sval = std::to_string(val);
                printChars(' ', x - idx), printNode(color, val);
                idx = (x + sval.length());
                if (p->left != NIL)
                {
                    next.push(p->left);
                    int a = widthzoom * p->left->getx();
                    int b = x + sval.length() / 2 - 1;
                    tuples.push_back(Tuple(a, b, '_'));
                }
                if (p->left != NIL || p->right != NIL)
                {
                    int t = x + sval.length() / 2;
                    tuples.push_back(Tuple(t, t, '|'));
                }
                if (p->right != NIL)
                {
                    next.push(p->right);
                    int a = x + sval.length() / 2 + 1;
                    int b = widthzoom * p->right->getx() + std::to_string(p->right->val).length() - 1;
                    tuples.push_back(Tuple(a, b, '_'));
                }
            }
            std::cout << '\n';
            idx = 0;
            for (auto &t : tuples)
            {
                int a = std::get<0>(t), b = std::get<1>(t);
                char c = std::get<2>(t);
                for (; idx < a; idx++)
                    printChars(' ', 1);
                printChars(c, b - a + 1);
                idx = b + 1;
            }
            std::cout << '\n';
            q = next;
        }
    }
};
```

