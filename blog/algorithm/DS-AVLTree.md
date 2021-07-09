## 二叉平衡树

Github Repo：https://github.com/sinkinben/DataStructure.git

图解可以看这篇文章：https://www.cnblogs.com/skywang12345/p/3576969.html

二叉搜索树 BST 在插入序列为升序序列时，退化为单向链表，增删改查的复杂度变为 $O(n)$ ，因此对其改进，提出二叉平衡树。

二叉平衡树，简称 AVL （以三位提出者的首字母连接起来作为命名），在二叉搜索树的基础上，加入以下约束条件：**每个节点的左右子树高度之差小于 2 。**

例如，这些子树都是不平衡的：

```
1                 a
 \               /
  2             b
   \          /
    3       c
```

再例如：

```
        x1
      /    \
    x2      x3
   /  \
  x4   x5
 /
x6
```

因此，在 AVL 的插入/删除过程中，需要对某个局部的子树进行「旋转」，以保持 AVL 本身**平衡**的性质。因此，AVL 的查询时间复杂度始终为 $O(lgn)$ ，不会像 BST 那样退化为 $O(n)$ 。

但是，由于 AVL 的插入/删除带来了额外的时间与空间开销（本文均采用递归实现），而 BST 始终为 $O(lgn)$，因此，在频繁插入/删除元素的场景下，AVL 是不适宜的。为了克服 AVL 这种弊端，同时保留其查询的优势，后来又有人提出了 [红黑树](https://www.cnblogs.com/sinkinben/p/13720765.html) 。

本文实现的 API ：

+ draw：在终端输出 AVL 的结构。
+ insert：插入节点。
+ remove：删除节点。
+ leftRotate：左旋处理。
+ rightRotate：右旋处理。
+ leftBalance：左子树超高时，左平衡处理。
+ rightBalance：右子树超高时，右平衡处理。

## 代码实现

RTFSC.

```cpp
#include "BinaryTree.hpp"
#include <cassert>
#include <iostream>
#ifndef AVLTREE_H
#define AVLTREE_H
template <typename T>
class AVLTree : public BinaryTree<T>
{
private:
    void selfCheck()
    {
        auto v = this->inorder();
        int len = v.size();
        for (int i = 1; i < len; i++)
            assert(v[i] > v[i - 1]);
    }

public:
    void rightRotate(TreeNode<T> *p)
    {
        if (p == nullptr)
            return;
        auto lc = p->left;

        assert(lc != nullptr);

        p->left = lc->right;
        if (p->left)
            p->left->parent = p;

        if (p->parent == nullptr)
            this->root = lc;
        else if (p->parent->left == p)
            p->parent->left = lc;
        else
            p->parent->right = lc;

        lc->parent = p->parent;

        lc->right = p, p->parent = lc;

        // p = lc;
    }

    void leftRotate(TreeNode<T> *p)
    {
        // std::cout << "00";
        if (p == nullptr)
            return;
        auto rc = p->right;

        assert(rc != nullptr);

        p->right = rc->left;
        if (p->right)
            p->right->parent = p;

        if (p->parent == nullptr)
            this->root = rc;
        else if (p->parent->left == p)
            p->parent->left = rc;
        else
        {
            // std::cout << "11";
            p->parent->right = rc;
        }

        rc->parent = p->parent;

        rc->left = p, p->parent = rc;

        // p = rc;
    }
    bool insert(int val)
    {
        bool taller;
        return insert(this->root, val, taller, nullptr);
        // std::cout << "111";
    }
    bool insert(TreeNode<T> *&subtree, T val, bool &taller, TreeNode<T> *parent)
    {
        if (subtree == nullptr)
        {
            subtree = new TreeNode<T>(val, AVLFactor::EH, parent);
            taller = true;
        }
        else if (val == subtree->val)
        {
            taller = false;
            return false;
        }
        else if (val < subtree->val)
        {
            if (!insert(subtree->left, val, taller, subtree))
                return false;
            if (taller)
            {
                switch (subtree->factor)
                {
                case AVLFactor::LH:
                    leftBalance(subtree);
                    taller = false;
                    break;
                case AVLFactor::EH:
                    subtree->factor = AVLFactor::LH;
                    taller = true;
                    break;
                case AVLFactor::RH:
                    subtree->factor = AVLFactor::EH;
                    taller = false;
                    break;
                default:
                    assert(0);
                    break;
                }
            }
        }
        else if (val > subtree->val)
        {
            if (!insert(subtree->right, val, taller, subtree))
                return false;
            if (taller)
            {
                switch (subtree->factor)
                {
                case AVLFactor::LH:
                    subtree->factor = AVLFactor::EH;
                    taller = false;
                    break;
                case AVLFactor::EH:
                    subtree->factor = AVLFactor::RH;
                    taller = true;
                    break;
                case AVLFactor::RH:
                    rightBalance(subtree);
                    taller = false;
                    break;
                default:
                    break;
                }
            }
        }
        else
        {
            assert(0);
        }
        return true;
    }

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

    bool remove(T val)
    {
        bool shorter = false;
        return remove(this->root, val, shorter);
    }

    bool remove(TreeNode<T> *&subtree, T val, bool &shorter)
    {
        // val is not in AVL
        if (subtree == nullptr)
        {
            return false;
        }
        else if (subtree->val == val)
        {
            auto p = subtree;
            bool hasLeft = p->left != nullptr;
            bool hasRight = p->right != nullptr;
            if (!hasLeft && hasRight)
            {
                subtree->right->parent = subtree->parent;
                subtree = subtree->right;
                delete p;
                shorter = true;
                return true;
            }
            if (hasLeft && !hasRight)
            {
                subtree->left->parent = subtree->parent;
                subtree = subtree->left;
                delete p;
                shorter = true;
                return true;
            }
            if (!hasLeft && !hasRight)
            {
                if (p == this->root)
                {
                    delete this->root;
                    this->root = nullptr;
                    return true;
                }
                if (p == p->parent->left)
                    p->parent->left = nullptr;
                else
                    p->parent->right = nullptr;
                delete p;
                shorter = true;
                return true;
            }
            if (hasLeft && hasRight)
            {
                // 把前驱的值搬上来，然后递归删除这个前驱
                auto lmax = maxval(p->left);
                subtree->val = lmax->val;
                if (remove(subtree->left, lmax->val, shorter))
                {
                    if (shorter)
                    {
                        switch (subtree->factor)
                        {
                        case AVLFactor::LH:
                            subtree->factor = AVLFactor::EH;
                            shorter = true;
                            break;
                        case AVLFactor::EH:
                            subtree->factor = AVLFactor::RH;
                            shorter = false;
                            break;
                        case AVLFactor::RH:
                            shorter = (subtree->right->factor == AVLFactor::EH) ? false : true;
                            rightBalance(subtree);
                            break;
                        default:
                            assert(0);
                            break;
                        }
                    }
                    return true;
                }
            }
            return false;
        }
        else if (val < subtree->val)
        {
            if (!remove(subtree->left, val, shorter))
                return false;
            if (shorter)
            {
                switch (subtree->factor)
                {
                case AVLFactor::LH:
                    subtree->factor = AVLFactor::EH;
                    shorter = true;
                    break;
                case AVLFactor::EH:
                    subtree->factor = AVLFactor::RH;
                    shorter = false;
                    break;
                case AVLFactor::RH:
                    shorter = (subtree->right->factor == AVLFactor::EH) ? false : true;
                    rightBalance(subtree);
                    break;
                default:
                    assert(0);
                    break;
                }
            }
            return true;
        }
        else if (val > subtree->val)
        {
            if (!remove(subtree->right, val, shorter))
                return false;
            if (shorter)
            {
                switch (subtree->factor)
                {
                case AVLFactor::LH:
                    shorter = (subtree->left->factor == AVLFactor::EH) ? (false) : (true);
                    leftBalance(subtree);
                    break;
                case AVLFactor::EH:
                    subtree->factor = AVLFactor::LH;
                    shorter = false;
                    break;
                case AVLFactor::RH:
                    subtree->factor = AVLFactor::EH;
                    shorter = true;
                    break;
                default:
                    assert(0);
                }
            }
            return true;
        }
        return false;
    }

    TreeNode<T> *maxval(TreeNode<T> *subtree)
    {
        if (subtree == nullptr)
            return nullptr;
        auto p = subtree;
        while (p->right != nullptr)
            p = p->right;
        return p;
    }
    TreeNode<T> *maxval() { return maxval(this->root); }

    void leftBalance(TreeNode<T> *p)
    {
        // lc means left child
        // lcr means leftChild->right
        TreeNode<T> *l = p->left, *lr = nullptr;
        switch (l->factor)
        {
        case AVLFactor::LH:
            p->factor = AVLFactor::EH;
            l->factor = AVLFactor::EH;
            rightRotate(p);
            break;
        case AVLFactor::RH:
            // here lcr must be not nullptr
            lr = l->right;
            switch (lr->factor)
            {
            case AVLFactor::LH:
                l->factor = AVLFactor::EH;
                lr->factor = AVLFactor::EH;
                p->factor = AVLFactor::RH;
                break;
            case AVLFactor::EH:
                l->factor = AVLFactor::EH;
                lr->factor = AVLFactor::EH;
                p->factor = AVLFactor::EH;
                break;
            case AVLFactor::RH:
                l->factor = AVLFactor::LH;
                lr->factor = AVLFactor::EH;
                p->factor = AVLFactor::EH;
                break;
            default:
                break;
            }
            leftRotate(p->left);
            rightRotate(p);
            break;
        // only used in remove operation
        case AVLFactor::EH:
            p->factor = AVLFactor::LH;
            p->left->factor = AVLFactor::RH;
            rightRotate(p);
            break;
        default:
            break;
        }
    }

    void rightBalance(TreeNode<T> *p)
    {
        TreeNode<T> *r, *rl;
        r = p->right;
        switch (r->factor)
        {
        case AVLFactor::RH:
            r->factor = p->factor = AVLFactor::EH;
            leftRotate(p);
            break;
        case AVLFactor::LH:
            rl = r->left;
            switch (rl->factor)
            {
            case AVLFactor::LH:
                p->factor = AVLFactor::EH;
                r->factor = AVLFactor::RH;
                break;
            case AVLFactor::EH:
                p->factor = r->factor = AVLFactor::EH;
                break;
            case AVLFactor::RH:
                p->factor = AVLFactor::EH;
                r->factor = AVLFactor::LH;
                break;
            default:
                break;
            }
            rl->factor = AVLFactor::EH;
            rightRotate(p->right);
            leftRotate(p);
            break;
        // only used in remove operation
        case AVLFactor::EH:
            p->factor = AVLFactor::RH;
            p->right->factor = AVLFactor::LH;
            leftRotate(p);
            break;
        default:
            break;
        }
    }
};

#endif
```



