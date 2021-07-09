## 缓存淘汰算法 LRU 和 LFU

LRU (Least Recently Used), 即最近最少使用算法，是一种常见的 Cache 页面置换算法，有利于提高 Cache 命中率。

LRU 的算法思想：对于每个页面，记录该页面自上一次被访问以来所经历的时间 $t$，当淘汰一个页面时，应选择所有页面中其 $t$ 值最大的页面，即内存中最近一段时间内**最长时间未被使用**的页面予以淘汰。

LFU (Least Frequently Used) ：为每个页面配置一个计数器，一旦某页被访问，则将其计数器的值加1，在需要选择一页置换时，则将选择其计数器值最小的页面，即内存中访问次数最少的页面进行淘汰。

其余常见的页面置换算法还有：

- OPT：理想化的置换算法，假设 OS 知道程序后续访问的所有页面的顺序，每次淘汰页面时，OPT 选择的页面将是以后永不使用的，或是在最长（未来）时间内不再被访问的页面。采用 OPT 通常可保证最低的缺页率（最高的 Cache 命中率）。
- FIFO：先进先出
- Clock 置换，又称最近未用算法 (NRU，Not Recently Used) 。
  - 算法思想：为每个页面设置一个**访问位**，再将内存中的页面都通过链接指针链接成一个**环形队列**。当某个页被访问时，其访问位置 1。当需要淘汰一个页面时，只需检查页的访问位。如果是 0，就选择该页换出；如果是 1，暂不换出，将访问位改为 0，继续检查下一个页面。若第一轮扫描中所有的页面都是 1，则将这些页面的访问位一次置为 0 后，再进行第二轮扫描，第二轮扫描中一定会有访问位为 0 的页面。



## LRU

### 例子

给定一个程序的页面访问序列：7 0 1 2 0 3 0 4，假设实际 Cache 只有 3 个页面大小，根据 LRU，画出 Cache 中的页面变化过程。

使用一个栈（大小是 3），栈顶总是最新访问的页面。当栈满时，最新访问页面为 `x`：

-  `x` 不在栈当中，去除**栈底元素**，把 `x` 置入栈顶。
- `x` 在栈当中，把 `x` 移至栈顶，其他页面顺序不变。

如下图所示，图源自[知乎](https://zhuanlan.zhihu.com/p/34133067)。

<img src="https://gitee.com/sinkinben/pic-go/raw/master/img/20210120200109.jpg" style="width:60%;" />

如果简单使用一个数组来模拟上述过程，访问一次页面的平均时间复杂度为 $O(n)$ 。

###  实现

在这里，LRU 存放的数据是一个键值对 `(key, val)` 。

Leetcode 题目：[146. LRU 缓存机制](https://leetcode-cn.com/problems/lru-cache/)。

要求 `get` 和 `put` 操作都在 $O(1)$ 时间内完成。

方法：双向链表+哈希。双向链表头尾各自带有一个 `dummy` 节点（可以简化插入、删除操作的代码）。

哈希表与链表的关系如图所示（图来源自 [Leetcode 讨论区](https://leetcode-cn.com/problems/lru-cache/solution/jian-dan-shi-li-xiang-xi-jiang-jie-lru-s-exsd/)）。

<img src="https://gitee.com/sinkinben/pic-go/raw/master/img/20210121163618.jpg" style="width:67%;" />

需要保证 `get` 方法在 $O(1)$ 内完成，因此哈希表只能使用 `unordered_map` 而不是 `map` 。

```cpp
struct Node
{
    int key, value;
    Node *next, *prev;
    Node(int k, int v) : key(k), value(v), prev(nullptr), next(nullptr) {}
};
class List
{
public:
    Node *head, *tail;
    int length;
    List() : length(0)
    {
        head = new Node(-1, -1), tail = new Node(-1, -1);
        head->next = tail, tail->prev = head;
    }
    void pushFront(Node *node)
    {
        auto next = head->next;
        head->next = node, node->next = next;
        next->prev = node, node->prev = head;
        length++;
    }
    void pushBack(Node *node)
    {
        auto prev = tail->prev;
        prev->next = node, node->next = tail;
        tail->prev = node, node->prev = prev;
        length++;
    }
    bool empty() { return length == 0 && head->next == tail && tail->prev == head; }
    Node *popFront() { return remove(head->next); }
    Node *popBack() { return remove(tail->prev); }
    Node *remove(Node *node)
    {
        if (node == nullptr || empty()) return nullptr;
        auto prev = node->prev, next = node->next;
        prev->next = next, next->prev = prev;
        node->next = node->prev = nullptr;
        length--;
        return node;
    }
};
class LRUCache
{
public:
    unordered_map<int, Node *> table;
    List list;
    int capacity;
    LRUCache(int capacity) { this->capacity = capacity; }

    int get(int key)
    {
        if (table.count(key) == 0) return -1;
        else
        {
            auto node = list.remove(table[key]);
            list.pushFront(node);
            return node->value;
        }
    }

    void put(int key, int value)
    {
        if (table.count(key) == 0)
        {
            auto node = new Node(key, value);
            table[key] = node;
            if (list.length == capacity)
            {
                auto p = list.popBack();
                table.erase(p->key);
                delete p;
            }
            list.pushFront(node);
        }
        else
        {
            table[key]->value = value;
            list.pushFront(list.remove(table[key]));
        }
    }
};

```

下面尝试使用 STL 中的 `list` 完成。

在 `list` 中，调用 `push, emplace, pop` 等操作，不会引起其他节点的迭代器失效。

```cpp
struct node
{
    int key, value;
    node(int k, int v) : key(k), value(v) {}
};
class LRUCache
{
public:
    unordered_map<int, list<node>::iterator> table;
    list<node> data;
    int capacity, length;
    LRUCache(int capacity) : length(0) { this->capacity = capacity; }

    int get(int key)
    {
        if (table.count(key) == 0) return -1;
        else
        {
            auto itor = table[key];
            int key = itor->key, val = itor->value;
            data.erase(itor);
            data.emplace_front(key, val);
            table[key] = data.begin();
            return val;
        }
    }

    void put(int key, int value)
    {
        if (table.count(key) == 0)
        {
            if (length == capacity)
            {
                auto itor = data.rbegin();
                table.erase(itor->key);
                data.pop_back();
                length--;
            }
            data.emplace_front(key, value);
            table[key] = data.begin();
            length++;
        }
        else
        {
            auto itor = table[key];
            data.erase(itor);
            data.emplace_front(key, value);
            table[key] = data.begin();
        }
    }
};
```



## LFU

Leetcode 题目：[460. LFU 缓存](https://leetcode-cn.com/problems/lfu-cache/)。

LFU (Least Frequently Used) 的主要思想是为每个缓存项一个计数器，一旦某个缓存项被访问，则将其计数器的值加 1，在需要淘汰缓存项时，则将选择其计数器值最小的，即内存中访问次数最少的缓存进行淘汰。

### 堆

按照这一描述，首先想到的是可以通过哈希表 + 优先队列（小顶堆），**堆中的数据以缓存项的计数器作为键值排序**。

对于 `get` 方法而言，通过哈希表找到 `key` 对应元素的位置，计数器加 1，重新调整堆，时间复杂度为 $O(\log{n})$ .

对于 `put` 方法而言，如果缓存中存在该元素，计数器加一，重新调整堆，时间复杂度为 $O(\log{n})$ ；如果不存在该元素并且缓存已满，那么直接把堆顶元素替换为新的元素 `<key,value>`（因为新元素的计数器为 1 ，必然也是最小的），时间复杂度为 $O(1)$。因此，`put` 方法总的时间复杂度为 $O(\log{n})$ .

这一方法基于堆实现，无法保证当 2 个元素的计数器相同时，被淘汰的是「较旧」的元素。



### 哈希/链表

现考虑 `get` 和 `put` 均为 $O(1)$ 的解法。

考虑基于「哈希+十字链表」实现，如下图所示（出处见水印）。

链表节点定义为：

```cpp
struct Node
{
    int key, value, counter;
    Node *prev, *next, *another;
};
```

<img src="https://gitee.com/sinkinben/pic-go/raw/master/img/20210121174737.png" style="width:65%" />

`hashmap` 用于记录 `key -> Node*` 的映射关系，辅助 `get` 方法在 $O(1)$ 时间内完成。

对于访问次数相同的节点，用链表连接起来（上图的横向链表），链表的头部记录访问次数，随后连接缓存数据的节点，数据节点有一个额外的指针 `another` 指向第一个节点（即记录访问次数的节点），然后把所有链表的头部也连接起来（上图的纵向链表），形成十字链表。

对于 `get` 方法，通过 `p = hashmap[key]` 找到指向数据的指针，然后把 `p` 移动到 `count+1` 的链表尾部（如果链表 `count+1` 不存在则插入一个）。

考虑 `put` 方法的最坏情况，如果 `<key, val>` 不在缓存当中， 并且缓存已满，那么就从十字链表的第一个链表（`count` 值最小的链表）**删除尾部节点**，并在**头部插入新节点**（这么做是为了按照从新到旧存储每个数据，尾部就是「最旧」的节点，可以做到计数相同的情况下，淘汰旧元素）。

但这种「十字链表」结构实现起来过于复杂（代码肯定不简洁），所以我们把「十字链表」转换为一个哈希表 `hash<int, List>` ，如下图所示。

<img src="https://gitee.com/sinkinben/pic-go/raw/master/img/20210121185255.jpg" style="width:80%;" />

代码实现：

```cpp
struct Node
{
    int key, value, counter;
    Node *next, *prev;
    Node(int k, int v) : key(k), value(v), counter(1), prev(nullptr), next(nullptr) {}
};
class List
{
public:
    Node *head, *tail;
    int length;
    List() : length(0)
    {
        head = new Node(-1, -1), tail = new Node(-1, -1);
        head->next = tail, tail->prev = head;
    }
    void pushFront(Node *node)
    {
        auto next = head->next;
        head->next = node, node->next = next;
        next->prev = node, node->prev = head;
        length++;
    }
    void pushBack(Node *node)
    {
        auto prev = tail->prev;
        prev->next = node, node->next = tail;
        tail->prev = node, node->prev = prev;
        length++;
    }
    bool empty() { return length == 0 && head->next == tail && tail->prev == head; }
    Node *popFront() { return remove(head->next); }
    Node *popBack() { return remove(tail->prev); }
    Node *remove(Node *node)
    {
        if (node == nullptr || empty()) return nullptr;
        auto prev = node->prev, next = node->next;
        prev->next = next, next->prev = prev;
        node->next = node->prev = nullptr;
        length--;
        return node;
    }
};
class LFUCache
{
public:
    unordered_map<int, Node *> table;
    unordered_map<int, List> data;
    int capacity, total, minCounter;
    LFUCache(int capacity)
    {
        this->capacity = capacity;
        this->total = 0;
        this->minCounter = 1;
    }

    int get(int key)
    {
        if (table.count(key) == 0) return -1;
        else
        {
            auto node = table[key];
            int counter = node->counter;
            node->counter++;
            data[counter].remove(node);
            data[counter + 1].pushFront(node);
            if (data[counter].length == 0 && counter == minCounter)
                minCounter = counter + 1;
            return node->value;
        }
    }

    void put(int key, int value)
    {
        if (capacity == 0) return;
        if (table.count(key) == 0)
        {
            auto node = new Node(key, value);
            if (total == capacity)
            {
                auto p = data[minCounter].popBack();
                table.erase(p->key);
                total--;
                delete p;
            }
            minCounter = 1;
            table[key] = node;
            data[node->counter].pushFront(node);
            total++;
        }
        else
        {
            auto node = table[key];
            int counter = node->counter;
            node->value = value, node->counter++;
            data[counter].remove(node);
            data[counter + 1].pushFront(node);
            if (data[counter].length == 0 && counter == minCounter)
                minCounter = counter + 1;
        }
    }
};
```

基于 STL 的 `list` 实现。

（最近老是因为手残而 de 一些毫无意义的 bug，真的服了自己，这里就因为把 `node` 的构造函数初始化 `counter(c)` 写成为 `counter(1)`，白白纠结了 1 个多小时）

```cpp
struct node
{
    int key, value, counter;
    node(int k, int v, int c = 1) : key(k), value(v), counter(c) {}
};
class LFUCache
{
public:
    unordered_map<int, list<node>::iterator> table;
    unordered_map<int, list<node>> data;
    int capacity, size, mincounter;
    LFUCache(int capacity) : size(0), mincounter(1)
    { this->capacity = capacity; }

    int get(int key)
    {
        if (table.count(key))
        {
            auto itor = table[key];
            int k = itor->key, v = itor->value, c = itor->counter;
            data[c].erase(itor);
            data[c + 1].emplace_front(k, v, c + 1);
            table[key] = data[c + 1].begin();
            if (data[c].size() == 0 && mincounter == c)
                mincounter = c + 1;
            return v;
        }
        return -1;
    }

    void put(int key, int value)
    {
        if (capacity == 0) return;
        if (table.count(key) == 0)
        {
            if (size == capacity)
            {
                auto node = data[mincounter].back();
                table.erase(node.key);
                data[mincounter].pop_back();
                size--;
            }
            mincounter = 1;
            data[1].emplace_front(key, value);
            table[key] = data[1].begin();
            size++;
        }
        else
        {
            auto itor = table[key];
            int counter = itor->counter;
            data[counter].erase(itor);
            data[counter + 1].emplace_front(key, value, counter + 1);
            table[key] = data[counter + 1].begin();
            if (data[counter].size() == 0 && mincounter == counter)
                mincounter = counter + 1;
        }
    }
};
```

