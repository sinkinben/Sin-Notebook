## [CPP] STL 简介

STL 即标准模板库（Standard Template Library），是 C++ 标准库的一部分，里面包含了一些模板化的通用的数据结构和算法。STL 基于模版的实现，因此能够支持自定义的数据结构。

STL 中一共有 6 大组件：

- 容器 (container)
- 迭代器 (iterator)
- 算法 (algorithm)
- 分配器 (allocator)
- 仿函数 (functor)
- 容器适配器 (container adapter)

参考资料：

+ [1] https://oi-wiki.org/lang/csl/container/
+ [2] https://en.cppreference.com/w/cpp/container
+ [3] http://c.biancheng.net/view/409.html

## 仿函数

仿函数 (Functor) 的本质就是**在结构体中重载 `()` 运算符**。

例如：

```cpp
struct Print
{ void operator()(const char *s) const { cout << s << endl; } };
int main()
{
    Print p;
    p("hello");
}
```

这一概念将会在 `priority_queue` 中使用（在智能指针的 `unique_ptr` 自定义 deleter 也会用到）。

## 容器

容器 (Container) 在 STL 中又分为序列式容器 (Sequence Containers) ，关联式容器 (Associative Containers) 和无序容器 (Unorderde Containers) .

<img src="https://gitee.com/sinkinben/pic-go/raw/master/img/20210121202108.png" style="width:80%; border-radius: 0px" />

### 序列式容器

常见的序列式容器包括有：`vector, string, array, deque, list, forward_list` .

**vector/string**

底层实现：`vector` 是**内存连续、自动扩容的数组**，实质还是定长数组。一般通过 3 个迭代器 `first, last, end` 实现，`first` 指向第一个有效元素，`last` 指向最后一个有效元素，`end` 指向申请内存空间的末尾。

特点：

- 随机访问：重载 `[]` 运算符
- 动态扩容：插入新元素前，如果 `size == capacity` 时，那么扩容为当前容量的 2 倍，并拷贝原来的数据
- 支持 `==, !=, <, <=, >, >=` 比较运算
  - C++20 前，通过上述 6 个重载运算符实现；C++20中，统一「封装」为一个 `<=>` 运算符 (aka, [three-way comparsion](https://en.cppreference.com/w/cpp/language/operator_comparison#Three-way_comparison) )。
  - 不难理解，时间复杂度为 $O(n)$ 。

PS：`string` 的底层实现与 `vector` 是类似的，同样是内存连续、自动扩容的数组（但扩容策略不同）。

-----

**array (C++11)**

底层实现：`array` 是**内存连续的** 、 **固定长度**的数组，其本质是对原生数组的直接封装。

特点（主要是与 `vector` 比较）：

- 支持 6 种比较运算符，支持 `[]` 随机访问
- 丢弃自动扩容，以获得跟原生数组一样的性能
- 不支持 `pop_front/back, erase, insert` 这些操作。 
- 长度在编译期确定。`vector` 的初始化方式为函数参数（如 `vector<int> v(10, -1)`，长度可动态确定），但 `array` 的长度需要在编译期确定，如 `array<int, 10> a = {1, 2, 3}` .

需要注意的是，`array` 的 `swap` 方法复杂度是 $\Theta(n)$ ，而其他 STL 容器的 `swap` 是 $O(1)$，因为只需要交换一下指针。

----

**deque**

又称为“双端队列”。

底层实现：**多个不连续的缓冲区，而缓冲区中的内存是连续的**。而每个缓冲区还会记录首指针和尾指针，用来标记有效数据的区间。当一个缓冲区填满之后便会在之前或者之后分配新的缓冲区来存储更多的数据。

特点：

- 支持 `[]` 随机访问
- 线性复杂度的插入和删除，以及常数复杂度的随机访问。

-----

**list**

底层实现：双向链表。

特点：

- 不支持 `[]` 随机访问
- 常数复杂度的插入和删除

**forwar_list (C++11)**

底层实现：单向链表。

特点：

- 相比 `list` 减少了空间开销
- 不支持 `[]` 随机访问
- 不支持反向迭代 `rbegin(), rend()`



### 关联式容器

关联式容器包括：`set/multiset`，`map/multimap`。`multi` 表示键值可重复插入容器。

底层实现：红黑树。

特点：

- 内部自排序，搜索、移除和插入拥有对数复杂度。
- 对于任意关联式容器，使用迭代器遍历容器的时间复杂度均为 $O(n)$ 。

自定义比较方式：

- 如果是自定义数据类型，重载运算符 `<` 
- 如果是 `int` 等内置类型，通过仿函数

```cpp
struct cmp { bool operator()(int a, int b) { return a > b; } };
set<int, cmp> s;
```



### 无序容器

无序容器 (Unorderde Containers) 包括：`unordered_set/unordered_multiset`,`unordered_map/unordered_multimap` .

底层实现：哈希表。在标准库实现里，每个元素的散列值是将值对**一个质数**取模得到的，

特点：

- 内部元素无序
- 在最坏情况下，对无序关联式容器进行插入、删除、查找等操作的时间复杂度会**与容器大小成线性关系** 。这一情况往往在容器内出现**大量哈希冲突**时产生。

在实际应用场景下，假设我们已知键值的具体分布情况，为了避免大量的哈希冲突，我们可以自定义哈希函数（还是通过仿函数的形式）。

```cpp
struct my_hash { size_t operator()(int x) const { return x; } };
unordered_map<int, int, my_hash> my_map;
unordered_map<pair<int, int>, int, my_hash> my_pair_map;
```



### 小结

四种操作**的平均时间复杂度**比较：

- 增：在指定位置插入元素
- 删：删除指定位置的元素
- 改：修改指定位置的元素
- 查：查找某一元素

|                         Containers                         |                    底层结构                    |      增      |      删      |      改      |      查      |
| :--------------------------------------------------------: | :--------------------------------------------: | :----------: | :----------: | :----------: | :----------: |
|                       `vector/deque`                       | vector: 动态连续内存<br />deque: 连续内存+链表 |    $O(n)$    |    $O(n)$    |    $O(1)$    |    $O(n)$    |
|                           `list`                           |                    双向链表                    |    $O(1)$    |    $O(1)$    |    $O(1)$    |    $O(n)$    |
|                       `forward_list`                       |                    单向链表                    |    $O(1)$    |    $O(n)$    |    $O(1)$    |    $O(n)$    |
|                          `array`                           |                    连续内存                    |    不支持    |    不支持    |    $O(1)$    |    $O(n)$    |
|                `set/map/multiset/multimap`                 |                     红黑树                     | $O(\log{n})$ | $O(\log{n})$ | $O(\log{n})$ | $O(\log{n})$ |
| `unordered_{set,multiset}`<br />`unordered_{map,multimap}` |                     哈希表                     |    $O(1)$    |    $O(1)$    |    $O(1)$    |    $O(1)$    |



## 容器适配器

容器适配器 (Container Adapter) 其实并不是容器（个人理解是对容器的一种封装），它们不具有容器的某些特点（如：有迭代器、有 `clear()` 函数……）。

常见的适配器：`stack`，`queue`，`priority_queue`。

对于适配器而言，可以指定某一容器作为其底层的数据结构。

**stack**

- 默认容器：`deque`
- 不支持随机访问，不支持迭代器
- `top, pop, push, size, empty` 操作的时间复杂度均为 $O(1)$ 。

指定容器作为底层数据结构：

```cpp
stack<TypeName, Container> s;  // 使用 Container 作为底层容器
```



**queue**

- 默认容器：`deque`
- 不支持随机访问，不支持迭代器
- `front, push, pop, size, empty` 操作的时间复杂度均为 $O(1)$ 。

指定容器：

```cpp
queue<int, vector<int>> q;
```



**priority_queue**

又称为 “优先队列” 。

- 默认容器：`vector`
- $O(1)$：`top, empty, size`
- $O(\log{n})$ : `push, pop`

模版参数解析：

```cpp
priority_queue<T, Container = vector<T>, Compare = less<T>> q;
// 通过 Container 指定底层容器，默认为 vector
// 通过 Compare 自定义比较函数，默认为 less，元素优先级大的在堆顶，即大顶堆
priority_queue<int, vector<int>, greater<int>> q;
// 传入 greater<int> 那么将构造一个小顶堆
// 类似的，还有 greater_equal, less_equal
```

## 迭代器

迭代器 (Iterator) 实际上也是 GOF 中的一种设计模式：**提供一种方法顺序访问一个聚合对象中各个元素，而又不需暴露该对象的内部表示。**

迭代器的分类如下图所示。

<img src="https://gitee.com/sinkinben/pic-go/raw/master/img/20210122145356.png" style="width:50%;" />

### 各容器的迭代器

STL 中各容器/适配器对应使用的迭代器如下表所示。

|         Container         |    Iterator    |
| :-----------------------: | :------------: |
|           array           | 随机访问迭代器 |
|          vector           | 随机访问迭代器 |
|           deque           | 随机访问迭代器 |
|           list            |   双向迭代器   |
|      set / multiset       |   双向迭代器   |
|      map / multimap       |   双向迭代器   |
|       forward_list        |   前向迭代器   |
| unordered_{set, multiset} |   前向迭代器   |
| unordered_{map, multimap} |   前向迭代器   |
|           stack           |  不支持迭代器  |
|           queue           |  不支持迭代器  |
|      priority_queue       |  不支持迭代器  |



### 迭代器失效

迭代器失效是因为向容器插入或者删除元素导致容器的空间变化或者说是次序发生了变化，使得原迭代器变得不可用。因此在对 STL 迭代器进行增删操作时，要格外注意迭代器是否失效。

网络上搜索「迭代器失效」，会发现很多这样的例子，在一个 `vector` 中去除所有的 2 和 3，故意用一下迭代器扫描（~~大家都知道可以用下标~~）：

```cpp
int main()
{
    vector<int> v = {2, 3, 4, 6, 7, 8, 9, 3, 2, 2, 2, 2, 3, 3, 3, 4, 5, 6};
    for (auto i = v.begin(); i != v.end(); i++)
    {
        if (*i==2 || *i==3) v.erase(i), i--;
        // correct code should be
        // if (*i==2 || *i==3) i=v.erase(i), i--;
    }
    for (auto i = v.begin(); i != v.end(); i++)
        cout << *i << ' ';
}
```

我很久之前用 Dev C++ （应该是内置了很古老的 MinGW）写代码的时候，印象中也遇到过这种情况，`v.erase(i), i--` 这样的操作是有问题的。 `erase(i)` 会使得 `i` 及其后面的迭代器失效，从而发生段错误。

但现在 MacOS (clang++ 12), Ubuntu16 (g++ 5.4), Windows (mingw 9.2) 上测试，这段代码都没有问题，并且能输出正确结果。编译选项为：

```
g++ test.cpp -std=c++11 -O0
```

实际上也不难理解，因为是连续内存，`i` 指向的内存位置，在 `erase` 之后被其他数据覆盖了（这里的行为就跟我们使用普通数组一样），但该位置仍然在 `vector` 的有效范围之内。在上述代码中，当 `i = v.begin()` 时，执行 `erase` 后，对 `i` 进行自减操作，这已经是一种未定义行为。

我猜应该是 C++11 后（或者是后来的编译器更新），对迭代器失效的这个问题进行了优化。

虽然能够正常运行，但我认为最好还是严谨一些，更严格地遵循迭代器的使用规则：`if (*i==2 || *i==3) i=v.erase(i), i--;` . 

以下为各类容器可能会发生迭代器失效的情况：

- 数组型 (`vector, deque`)
  - `insert(i)` 和 `erase(i)` 会发生数据挪动，使得 `i` 后的迭代器失效，建议使用 `i = erase(i)` 获取下一个有效迭代器。
  - 内存重新分配：当 `vector` 自动扩容时，**可能**会申请一块新的内存并拷贝原数据（也有可能是在当前内存的基础上，再扩充一段连续内存），因此所有的迭代器都将失效。
- 链表型 (`list, forward_list`)：`insert(i)` 和 `erase(i)` 操作不影响其他位置的迭代器，`erase(i)` 使得迭代器 `i` 失效，指向数据无效，`i = erase(i)` 可获得下一个有效迭代器，或者使用 `erase(i++)` 也可（在进入 `erase` 操作前已完成自增）。
- 树型 (`set/map`)：与链表型相同。
- 哈希型 (`unodered_{set_map}`)：与链表型相同。

