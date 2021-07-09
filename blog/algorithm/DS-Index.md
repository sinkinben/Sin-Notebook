## 重读数据结构

最近在重新学习和实现各类数据结构（以前的代码太烂了 orz），后面也许会一直更新。

现在先整理一下进度，顺便把这篇文章作为项目的说明文档。

Github Repo：https://github.com/sinkinben/DataStructure.git

相关文章：

+ 普通二叉树：https://www.cnblogs.com/sinkinben/p/13700645.html
+ 二叉搜索树：https://www.cnblogs.com/sinkinben/p/13708593.html
+ 二叉平衡树：https://www.cnblogs.com/sinkinben/p/13731435.html
+ 红黑树：https://www.cnblogs.com/sinkinben/p/13720765.html

To be continued.

## 已实现的数据结构

按文件夹分类：

+ BinaryTree：普通二叉树、二叉搜索树、二叉平衡树、红黑树。
+ HashTable：Cuckoo哈希表，线性哈希表。其实这是 2019 年 SJTU 夏令营的题目，请参考[这个仓库的 PDF 文档](https://github.com/sinkinben/SJTU-Courses/blob/master/2019SummerCamp/src/main/resources/2019%E8%BD%AF%E4%BB%B6%E5%B7%A5%E7%A8%8B%E4%BC%98%E6%89%8D%E5%A4%8F%E4%BB%A4%E8%90%A5%E6%B5%8B%E8%AF%95%E9%A2%98.pdf)。

## 环境

所使用环境配置和依赖：

+ Windows
+ Git Bash
+ GNU Make 4.2.1 
+ g++ 6.3.0  (需要支持 C++11/17 的版本)

如果你用的是 Linux ，我想也不用教怎么运行了。

考虑到只有大一/大二才会上数据结构这门课，所以如果不懂 bash 和 makefile 也是正常的，如果想知道怎么运行（但自己又尝试失败），欢迎[邮件](mailto:sinkinben@qq.com)联系。

## Feature

如果说这个项目有什么 Feature，唯一值得说的就是使用了模板。

比如 BSTree.hpp，可以像 STL 那样支持 `BSTree<int>, BSTree<char>` ，同时也支持自定义的数据类型（参考 `BSTreeTest.cpp` ），但要求自定义数据类型实现比较运算符和赋值运算符的重载。

实际上，我想到一种设计方案只需要实现 `<` 和 `==` 即可，但是我太懒了，没有改代码。

```cpp
// test generic and expandability
void test3()
{
    class Node
    {
    private:
        int key, val;

    public:
        Node() {}
        Node(int k, int v) : key(k), val(v) {}
        // These operators are necessary.
        bool operator<(const Node &n) const { return key < n.key; }
        bool operator>(const Node &n) const { return key > n.key; }
        bool operator==(const Node &n) const { return key == n.key; }
        bool operator!=(const Node &n) const { return key != n.key; }
        Node &operator=(const Node &n)
        {
            key = n.key, val = n.val;
            return *this;
        }
    };
    BSTree<Node> tree;
    srand(time(nullptr));
    vector<int> pool;
    int n = 10000;
    while (n--)
        pool.push_back(rand());
    for (int x : pool)
        tree.insert(Node(x, x));
    tree.selfCheck();
    for (int x : pool)
    {
        tree.remove(Node(x, x));
        tree.selfCheck();
    }
    cout << "Pass Test Case 3: Generic Expandability." << endl;
}
```



## 二叉树演示

进入目录 BinaryTree，输入 `make xxx` 可以运行对应数据结构的演示程序，`xxx` 可以是：

+ bintree：普通二叉树
+ bstree：二叉搜索树
+ rbtree：红黑树
+ avltree：二叉平衡树

如果可以，学习读一下 makefile ，挺简单的~

### 代码文件关系

+ TreeNode 是所有树公用的节点类。
+ AVLTree、BSTree 继承 BinaryTree，RBTree 没有选择继承是因为它使用了「哨兵」进行特殊处理，是单独实现的。
+ XXXMain.cpp 是各个树的演示程序。

### 普通二叉树

运行命令：

```bash
cd BinaryTree/
make bintree
```

该程序是一个交互式的命令行演示程序，允许输入的命令有：

+ show：显示二叉树的结构。
+ preoder：先序遍历。
+ inorder：中序遍历。
+ postorder：后序遍历。
+ build + `[...]` ：重新建树。
+ q：退出。

Demo 演示：

<img src="https://gitee.com/sinkinben/pic-go/raw/master/img/20200921213230.png" alt="image-20200921213230108" style="width:77%;" />



### 二叉搜索树

运行命令：

```bash
cd BinaryTree/
make bstree
```

允许输入的命令：

+ u oldval newval：update, 把 BST 中的 `old val` 更新为 `new val` 。
+ i + 1 2 3 ... : insert, 依次往 BST 插入后面跟随的数字序列。
+ r + val: remove, 从 BST 中删除 `val` 节点。
+ s + val: successor, 查找后继节点。
+ p + val: precedessor, 查找前驱节点。
+ preorder/inorder/postorder: 三序遍历（同上）。
+ show: 显示二叉树结构。
+ cls: 清屏。Windows 有效，其他 OS 平台大概率失效。

Demo 演示：

<img src="https://gitee.com/sinkinben/pic-go/raw/master/img/20200921213730.png" alt="image-20200921213730211" style="width: 77%;" />

### 红黑树

运行：

```bash
make rbtree
```

支持命令：

+ i + val: insert a value into RB-tree.
+ r + val: remove a val into RB-tree.
+ show: display the structure of RB-tree.
+ cls: clear the screen (only on Windows).

Demo 演示：

<img src="https://gitee.com/sinkinben/pic-go/raw/master/img/20200921214050.png" alt="image-20200921214049862" style="width:77%;" />

### 二叉平衡树

运行：

```bash
make avl
```

支持命令：

+ i + [val, ...] ：依次插入节点。
+ r + val：删除某个节点。
+ show：打印 AVL 树形结构。
+ preorder/inorder/postorder：三序遍历序列。

Demo 演示：

<img src="https://gitee.com/sinkinben/pic-go/raw/master/img/20200926183026.png" alt="image-20200926183026094" style="width:77%;" />