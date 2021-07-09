## 字典树

字典树（Trie Tree）：

> 又称单词查找树，是一种树形结构，是一种哈希树的变种。典型应用是用于统计，排序和保存大量的字符串（但不仅限于字符串），所以经常被搜索引擎系统用于文本词频统计。它的优点是：利用字符串的公共前缀来减少查询时间，最大限度地减少无谓的字符串比较，查询效率比哈希树高。

Trie Tree 的性质：

> 根节点不包含字符，除根节点外每一个节点都只包含一个字符； 从根节点到某一节点，路径上经过的字符连接起来，为该节点对应的字符串； 每个节点的所有子节点包含的字符都不相同。

下面是字典树的树形结构图示（图源来自 Leetcode 题解讨论区）：

<img src="https://gitee.com/sinkinben/pic-go/raw/master/img/20200929151020.png" style="width:67%;" />

## 实现

这是 leetcode 题目：[208. 实现 Trie (前缀树)](https://leetcode-cn.com/problems/implement-trie-prefix-tree/)。

**节点**

使用指针数组 `links[26]` 去记录下一个字符，`isEnd` 表示该节点是否为叶子节点，同时也表示某次遍历**是否找到一个完整的单词**。

```cpp
class TrieNode {
private:
    vector<TrieNode*> links;
    bool isEnd;
public:
    TrieNode(){ isEnd = false; links.resize(26, nullptr); }
    bool containsKey(char c) { return links[c - 'a'] != nullptr; }
    void put(char c) { links[c - 'a'] = new TrieNode(); }
    TrieNode* get(char c) { return links[c - 'a']; }
    void setEnd() { isEnd = true; }
    bool getEnd() { return isEnd; }
};
```

**字典树**

```cpp
class Trie {
private:
    TrieNode *root;
public:
    Trie() { root = new TrieNode(); }
    void insert(string word)
    {
        auto cur = root;
        for (char c: word)
        {
            if (!cur->containsKey(c))
                cur->put(c);
            cur = cur->get(c);
        }
        cur->setEnd();
    }

    bool search(string word)
    {
        auto cur = root;
        for (char c: word)
        {
            if (!cur->containsKey(c))
                return false;
            else
                cur = cur->get(c);
        }
        return cur->getEnd();
    }

    bool startsWith(string prefix)
    {
        auto cur = root;
        for (char c: prefix)
        {
            if (!cur->containsKey(c))
                return false;
            else
                cur = cur->get(c);
        }
        return true;
    }
};

```

## 应用

### 单词搜索 II

题目：[212. 单词搜索 II](https://leetcode-cn.com/problems/word-search-ii/)。

**解题思路**

源于[题解](https://leetcode-cn.com/problems/word-search-ii/solution/c-jian-dan-qing-xi-de-trieshu-ti-jie-by-talanto_li/)。

将所有的 `words` 建立字典树，然后对于 `board` 的每一个位置 `(i,j)` 进行 DFS。

**代码实现**

```cpp
struct TrieNode
{
    vector<TrieNode *> links;
    bool isend;
    string word;
    TrieNode() : isend(false), word("") { links.resize(26, nullptr); }
    bool contains(char c) { return links[c - 'a'] != nullptr; }
    void put(char c) { links[c - 'a'] = new TrieNode(); }
    TrieNode *get(char c) { return links[c - 'a']; }
};
class Solution
{
public:
    TrieNode *root = new TrieNode();
    vector<string> result;
    int row, col;
    vector<string> findWords(vector<vector<char>> &board, vector<string> &words)
    {
        if (board.size() == 0 || board[0].size() == 0)
            return result;
        buildTrieTree(words);
        row = board.size();
        col = board[0].size();
        for (int i = 0; i < row; i++)
        {
            for (int j = 0; j < col; j++)
            {
                dfs(board, root, i, j);
            }
        }
        return result;
    }

    void dfs(vector<vector<char>> &board, TrieNode *p, int x, int y)
    {
        char ch = board[x][y];
        if (ch == '.' || !p->contains(ch))
            return;
        p = p->get(ch);
        if (p->isend && p->word != "")
        {
            result.push_back(p->word);
            // 防止重复添加
            p->word = "";
        }

        board[x][y] = '.';
        if (x - 1 >= 0)  dfs(board, p, x - 1, y);
        if (x + 1 < row) dfs(board, p, x + 1, y);
        if (y + 1 < col) dfs(board, p, x, y + 1);
        if (y - 1 >= 0)  dfs(board, p, x, y - 1);
        board[x][y] = ch;
    }

    void buildTrieTree(vector<string> &vs)
    {
        for (auto &x : vs)
        {
            auto cur = root;
            for (char c : x)
            {
                if (!cur->contains(c))
                    cur->put(c);
                cur = cur->get(c);
            }
            cur->isend = true, cur->word = x;
        }
    }
};
```

### 添加与搜索单词

题目：[211. 添加与搜索单词 - 数据结构设计](https://leetcode-cn.com/problems/design-add-and-search-words-data-structure/)。

递归搜索。

```cpp
struct TrieNode
{
    vector<TrieNode *> links;
    bool isend;
    TrieNode() : isend(false) { links.resize(26, nullptr); }
    bool contains(char c) { return (links[c - 'a'] != nullptr); }
    void put(char c) { links[c - 'a'] = new TrieNode(); }
    TrieNode *get(char c) { return links[c - 'a']; }
};
class WordDictionary
{
public:
    TrieNode *root;
    /** Initialize your data structure here. */
    WordDictionary()
    {
        root = new TrieNode();
    }

    /** Adds a word into the data structure. */
    void addWord(string word)
    {
        auto cur = root;
        for (char c : word)
        {
            if (!cur->contains(c))
                cur->put(c);
            cur = cur->get(c);
        }
        cur->isend = true;
    }

    /** Returns if the word is in the data structure. A word could contain the dot character '.' to represent any one letter. */
    bool search(string word)
    {
        return innerSearch(root, word);
    }

    bool innerSearch(TrieNode *p, string word)
    {
        if (p == nullptr)
            return false;
        if (word.length() == 0)
            return p->isend;
        auto cur = p;
        int len = word.length();
        for (int i = 0; i < len; i++)
        {
            char c = word[i];
            if (c == '.')
            {
                for (auto x : cur->links)
                {
                    if (x != nullptr && innerSearch(x, word.substr(i + 1)))
                        return true;
                }
                return false;
            }
            else
            {
                if (!cur->contains(c))
                    return false;
                else
                    // return innerSearch(cur->get(c), word.substr(i + 1));
                    cur = cur->get(c);
            }
        }
        return cur->isend;
    }
};
```

