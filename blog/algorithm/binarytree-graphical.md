## 二叉树图形化显示

在刷 OJ 二叉树题目的时候，文字描述的输入都是 `[1, null, 2]` 这种形式，但输入参数却是 `TreeNode *root`，很不直观，一旦节点数目很多，很难想象输入的二叉树是什么样子的。leetcode 上提供了一个很好的二叉树图形显示，现在自己动手实现一遍，也方便在其他地方使用。

### 第零步：前言

用 C++ 实现。假定输入格式是 `[6,2,8,0,4,7,9,null,null,3,5]` 这种形式的**字符串**（因为 C++ 没有像 Python 那样的 `list`）。

上面的二叉树图形如下：

```
      6
   /     \
  2       8
 / \     / \
0   4   7   9
   / \
  3   5
```

二叉树节点定义如下：

```cpp
struct TreeNode
{
    int val;
    int x, y;
    TreeNode *left, *right;
    TreeNode(int v) : val(v), left(nullptr), right(nullptr), x(-1), y(-1) {}
};
```

其中，`(x, y)` 表示该节点在屏幕上的坐标。

### 第一步：建树

给定字符串 `string s = "[6,2,8,0,4,7,9,null,null,3,5]"`，首先我们将其转换为一个 `vector<string> v` ，其中 `v` 的元素为 `"6", "2", ..., "null", ..., "5"` 。

预处理字符串的操作如下：

```cpp
// discard the '[]'
s = s.substr(1, s.size() - 2);
// change s into ["1", ...,"2"]
auto v = split(s, ",");
```

`split` 函数如下：

```cpp
static vector<string> split(string &s, const string &token)
{
    replaceAll(s, token, " ");
    vector<string> result;
    stringstream ss(s);
    string buf;
    while (ss >> buf)
        result.push_back(buf);
    return result;
}
```

`replaceAll` 函数的作用是把所有的 `oldChars` 替换为 `newChars` ：

```cpp
static void replaceAll(string &s, const string &oldChars, const string &newChars)
{
    int pos = s.find(oldChars);
    while (pos != string::npos)
    {
        s.replace(pos, oldChars.length(), newChars);
        pos = s.find(oldChars);
    }
}
```



那么，这个 `v` 就是二叉树的数组形式（根节点是 `v[0]` ），其具有以下性质：

> `v[i].left` 为 `v[2 * i + 1]`，`v[i].right` 为 `v[2 * i + 2]` 。

 建树是常见的递归操作：

```cpp
// create binary tree
TreeNode *root = nullptr;
innerCreate(v, 0, root);
```

`innerCreate`的具体实现如下：

```cpp
static void innerCreate(vector<string> &v, size_t idx, TreeNode *&p)
{
    if (idx >= v.size() || v[idx] == "null")
        return;
    p = new TreeNode(stoi(v[idx]));
    innerCreate(v, 2 * idx + 1, p->left);
    innerCreate(v, 2 * idx + 2, p->right);
}
```

### 第二步：定义坐标

定义坐标系：Console 中左上角为原点，向右为 X 轴正方向，向下为 Y 轴正方向。

很自然地，我们就把节点所在的**层数**作为节点的 Y 轴坐标。

二叉树还有一个有趣的性质：中序遍历是二叉树的「从左往右」的遍历。所以我们把节点**中序遍历所在的位置**作为节点的 X 轴坐标。

层次遍历初始化所有节点的 Y 坐标：

```cpp
static void initY(TreeNode *root)
{
    if (root == nullptr)
        return;

    typedef pair<TreeNode *, int> Node;

    root->y = 1;

    queue<Node> q;
    q.push(Node(root, root->y));
    while (!q.empty())
    {
        auto p = q.front();
        q.pop();
        if (p.first->left != nullptr)
        {
            p.first->left->y = p.second + 1;
            q.push(Node(p.first->left, p.second + 1));
        }
        if (p.first->right != nullptr)
        {
            p.first->right->y = p.second + 1;
            q.push(Node(p.first->right, p.second + 1));
        }
    }
}
```

中序遍历初始化所有节点的 X 坐标：

```cpp
static void initX(TreeNode *p, int &x)
{
    if (p == nullptr)
        return;
    initX(p->left, x);
    p->x = x++;
    initX(p->right, x);
}
```

完整的建树操作（包括初始化坐标操作）：

```cpp
static TreeNode *create(string &s)
{
    // discard the '[]'
    s = s.substr(1, s.size() - 2);
    // change s into ["1", ...,"2"]
    auto v = split(s, ',');
    // create binary tree
    TreeNode *root = nullptr;
    innerCreate(v, 0, root);
    // init x and y of tree nodes
    initCoordinate(root);
    return root;
}
static void initCoordinate(TreeNode *root)
{
    int x = 0;
    initX(root, x);
    initY(root);
}
```

### 第三步：定义画布

所谓的画布 `Canvas` 其实就是一个二维数组 `char buffer[HEIGHT][WIDTH]` ，我们把要输出的字符都放到 `buffer` 相应的位置，最后输出 `buffer` 。

`Canvas` 类代码如下：

```cpp
class Canvas
{
public:
    static const int HEIGHT = 10;
    static const int WIDTH = 80;
    static char buffer[HEIGHT][WIDTH + 1];

    // print buffer
    static void draw()
    {
        cout << endl;
        for (int i = 0; i < HEIGHT; i++)
        {
            buffer[i][WIDTH] = '\0';
            cout << buffer[i] << endl;
        }
        cout << endl;
    }

    // put 's' at buffer[r][c]
    static void put(int r, int c, const string &s)
    {
        int len = s.length();
        int idx = 0;
        for (int i = c; (i < WIDTH) && (idx < len); i++)
            buffer[r][i] = s[idx++];
    }
    // put n 'ch' at buffer[r][c]
    static void put(int r, int c, char ch, int num)
    {
        while (num > 0 && c < WIDTH)
            buffer[r][c++] = ch, num--;
    }

    // clear the buffer
    static void resetBuffer()
    {
        for (int i = 0; i < HEIGHT; i++)
            memset(buffer[i], ' ', WIDTH);
    }
};
// Do not remove this line
char Canvas::buffer[Canvas::HEIGHT][Canvas::WIDTH + 1];
```

调用方法如下：

```cpp
Canvas::resetBuffer();
// call Cancas::put() to put something into buffer
Canvas::put(3, 3, "hello world");
Cancas::draw();
```

### 第五步：绘制二叉树

绘制样式我想到 2 种。

经典型。看着好看，一旦考虑到每个节点 `val` 的长度不一致，节点多的时候，画出来的效果很不好。

```
Tree-1
    1
   / \
  2   4
   \
    3

Tree-2
               1
              / \
  111111111111   22222222222222222 
 /            \
4              5
```

对于前面定义的 X 坐标，中序遍历的序列当中，X 坐标都是连续的，即从 0 到 n 变化。这样画出来显然不行，因为 `node.val` 占据了一定的长度，符号 `/` 和 `\` 也要占据一个宽度，所以采取的办法是 **将横坐标统一乘以定值 `widthZoom`** 。根据输入的实际情况，自己调整 `widthZoom` 的大小（数值都是个位数，`widthZoom` 取 1 即可）。

对于 Y 坐标也是连续的，但显然符号 `/` 和 `\` 要占据一行，所以节点 `node` 在画布中的 Y 轴位置应该为 `2 * node.y` 。

```cpp
static void show2(TreeNode *root)
{
    const int widthZoom = 1;
    Canvas::resetBuffer();
    queue<TreeNode *> q;
    q.push(root);
    int x, y, val;
    while (!q.empty())
    {
        auto p = q.front();
        q.pop();
        x = p->x, y = p->y, val = p->val;
        Canvas::put(2 * y, widthZoom * x, to_string(val));
        if (p->left != nullptr)
        {
            q.push(p->left);
            Canvas::put(2 * y + 1, widthZoom * ((p->left->x + x) / 2), '/', 1);
        }
        if (p->right != nullptr)
        {
            q.push(p->right);
            Canvas::put(2 * y + 1, widthZoom * ((x + p->right->x) / 2) + 1, '\\', 1);
        }
    }
    Canvas::draw();
}
```



不知道叫什么型。节点数少，效果固然不如第一种。但是节点数一多，效果比第一种稍好（但是还是不太满意），应付一般的场景够用。

```
     1
  ___|____________
  2     4444444444
  |______
    33333
```

X 和 Y 坐标的处理同上。此处 `widthZoom` 的值最好取大于等于 3 。代码如下：

```cpp
static void show(TreeNode *root)
{
    const int widthZoom = 3;
    Canvas::resetBuffer();
    queue<TreeNode *> q;
    q.push(root);
    int x, y, val;
    string sval;
    while (!q.empty())
    {
        auto p = q.front();
        q.pop();
        bool l = (p->left != nullptr);
        bool r = (p->right != nullptr);
        x = p->x, y = p->y, val = p->val, sval = to_string(p->val);
        Canvas::put(2 * y, widthZoom * x, sval);
        if (l)
        {
            q.push(p->left);
            Canvas::put(2 * y + 1, widthZoom * p->left->x, '_', widthZoom * (x - p->left->x) + sval.length() / 2);
        }
        if (r)
        {
            q.push(p->right);
            Canvas::put(2 * y + 1, widthZoom * x, '_',
                        widthZoom * (p->right->x - x) + to_string(p->right->val).length());
        }
        if (l || r)
            Canvas::put(2 * y + 1, widthZoom * x + sval.length() / 2, "|");
    }
    Canvas::draw();
}
```

### 最终步：效果

+ `s = [6,2,8,0,4,7,9,null,null,3,5]`

  ```
  width zoom: 3
                 6
     ____________|______
     2                 8
  ___|______        ___|___
  0        4        7     9
        ___|___
        3     5
  width zoom: 1
       6
     /   \
   2     8
  /  \  / \
  0  4  7 9
    / \
    3 5
  ```

+ `s = [512,46, 7453,35, 5656,26,null,-1,null,9,null]`

  ```
  width zoom: 3
                 512
        __________|________
        46             7453
     ____|_____     _____|
     35       6     26
  ____|    ___|
  -1       9
  width zoom: 2
          512
        /      \
      46        7453
    /    \    /
    35    6   26
  /     /
  -1    9
  ```

### 完整代码

```cpp
#include <queue>
#include <vector>
#include <string>
#include <cstring>
#include <sstream>
#include <iostream>
using namespace std;
struct TreeNode
{
    int val;
    int x, y;
    TreeNode *left, *right;
    TreeNode(int v) : val(v), left(nullptr), right(nullptr), x(-1), y(-1) {}
};

class Canvas
{
public:
    static const int HEIGHT = 10;
    static const int WIDTH = 80;
    static char buffer[HEIGHT][WIDTH + 1];

    static void draw()
    {
        cout << endl;
        for (int i = 0; i < HEIGHT; i++)
        {
            buffer[i][WIDTH] = '\0';
            cout << buffer[i] << endl;
        }
        cout << endl;
    }

    static void put(int r, int c, const string &s)
    {
        int len = s.length();
        int idx = 0;
        for (int i = c; (i < WIDTH) && (idx < len); i++)
            buffer[r][i] = s[idx++];
    }
    static void put(int r, int c, char ch, int num)
    {
        while (num > 0 && c < WIDTH)
            buffer[r][c++] = ch, num--;
    }

    static void resetBuffer()
    {
        for (int i = 0; i < HEIGHT; i++)
            memset(buffer[i], ' ', WIDTH);
    }
};

char Canvas::buffer[Canvas::HEIGHT][Canvas::WIDTH + 1];

class BinaryTreeGui
{
public:
    static TreeNode *create(string &s)
    {
        // discard the '[]'
        s = s.substr(1, s.size() - 2);

        // change s into ["1", ...,"2"]
        auto v = split(s, ",");

        // create binary tree
        TreeNode *root = nullptr;
        innerCreate(v, 0, root);

        // init x and y of tree nodes
        initCoordinate(root);

        return root;
    }

    static void show(TreeNode *root)
    {
        const int widthZoom = 3;
        printf("width zoom: %d\n", widthZoom);
        Canvas::resetBuffer();
        queue<TreeNode *> q;
        q.push(root);
        int x, y, val;
        string sval;
        while (!q.empty())
        {
            auto p = q.front();
            q.pop();
            bool l = (p->left != nullptr);
            bool r = (p->right != nullptr);
            x = p->x, y = p->y, val = p->val, sval = to_string(p->val);
            Canvas::put(2 * y, widthZoom * x, sval);
            if (l)
            {
                q.push(p->left);
                Canvas::put(2 * y + 1, widthZoom * p->left->x, '_', widthZoom * (x - p->left->x) + sval.length() / 2);
            }
            if (r)
            {
                q.push(p->right);
                Canvas::put(2 * y + 1, widthZoom * x, '_',
                            widthZoom * (p->right->x - x) + to_string(p->right->val).length());
            }
            if (l || r)
                Canvas::put(2 * y + 1, widthZoom * x + sval.length() / 2, "|");
        }
        Canvas::draw();
    }
    static void show2(TreeNode *root)
    {
        const int widthZoom = 2;
        printf("width zoom: %d\n", widthZoom);
        Canvas::resetBuffer();
        queue<TreeNode *> q;
        q.push(root);
        int x, y, val;
        while (!q.empty())
        {
            auto p = q.front();
            q.pop();
            x = p->x, y = p->y, val = p->val;
            Canvas::put(2 * y, widthZoom * x, to_string(val));
            if (p->left != nullptr)
            {
                q.push(p->left);
                Canvas::put(2 * y + 1, widthZoom * ((p->left->x + x) / 2), '/', 1);
            }
            if (p->right != nullptr)
            {
                q.push(p->right);
                Canvas::put(2 * y + 1, widthZoom * ((x + p->right->x) / 2) + 1, '\\', 1);
            }
        }
        Canvas::draw();
    }

    static void destroy(TreeNode *root)
    {
        if (root == nullptr)
            return;
        destroy(root->left);
        destroy(root->right);
        delete root;
        root = nullptr;
    }

private:
    static void innerCreate(vector<string> &v, size_t idx, TreeNode *&p)
    {
        if (idx >= v.size() || v[idx] == "null")
            return;
        p = new TreeNode(stoi(v[idx]));
        innerCreate(v, 2 * idx + 1, p->left);
        innerCreate(v, 2 * idx + 2, p->right);
    }

    static void replaceAll(string &s, const string &oldChars, const string &newChars)
    {
        int pos = s.find(oldChars);
        while (pos != string::npos)
        {
            s.replace(pos, oldChars.length(), newChars);
            pos = s.find(oldChars);
        }
    }

    static vector<string> split(string &s, const string &token)
    {
        replaceAll(s, token, " ");
        vector<string> result;
        stringstream ss(s);
        string buf;
        while (ss >> buf)
            result.push_back(buf);
        return result;
    }

    static void initX(TreeNode *p, int &x)
    {
        if (p == nullptr)
            return;
        initX(p->left, x);
        p->x = x++;
        initX(p->right, x);
    }
    static void initY(TreeNode *root)
    {
        if (root == nullptr)
            return;

        typedef pair<TreeNode *, int> Node;

        root->y = 1;

        queue<Node> q;
        q.push(Node(root, root->y));
        while (!q.empty())
        {
            auto p = q.front();
            q.pop();
            if (p.first->left != nullptr)
            {
                p.first->left->y = p.second + 1;
                q.push(Node(p.first->left, p.second + 1));
            }
            if (p.first->right != nullptr)
            {
                p.first->right->y = p.second + 1;
                q.push(Node(p.first->right, p.second + 1));
            }
        }
    }

    static void initCoordinate(TreeNode *root)
    {
        int x = 0;
        initX(root, x);
        initY(root);
    }

    // print info of tree nodes
    static void inorder(TreeNode *p)
    {
        if (p == nullptr)
            return;
        inorder(p->left);
        printf("val=%d, x=%d, y=%d\n", p->val, p->x, p->y);
        inorder(p->right);
    }
};

int main(int argc, char *argv[])
{
    string s = "[512,46, 7453,35, 6,26,null,-1,null,9,null]";
    // string s = "[6,2,8,0,4,7,9,null,null,3,5]";
    // string s(argv[1]);
    auto root = BinaryTreeGui::create(s);
    BinaryTreeGui::show(root);
    BinaryTreeGui::show2(root);
    BinaryTreeGui::destroy(root);
}
```

