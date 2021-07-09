## [CPP] new 和 delete

参考：

+ [1] https://www.cnblogs.com/wangpei0522/p/4476470.html
+ [2] https://stackoverflow.com/questions/8940090/is-it-safe-on-linux-to-mix-new-and-delete

## 现象

某日发现一个奇怪的现象，先看以下代码：

```cpp
int *p = new int[10];
delete p;
```

我知道应该用 `delete[]` ，但实际上对于基本数据类型，在这里使用 `delete/delete[]` 似乎都可以。

继续尝试一下 `class`:

```cpp
class Basic
{
public:
    Basic() { cout << "Basic" << endl; }
    ~Basic() { cout << "~Basic" << endl; }
};
int main()
{
    Basic *b = new Basic[10];
    delete b;
}
```

对于上述代码，能编译成功，`~Basic` 只输出 1 次，随后出现运行时错误。但是假如把析构函数 `~Basic` 去掉，这个运行时错误就会消失。

经过在 Stackoverflow 的搜索，据说这是因为 `new[4]` 包含一个信息头，而 `new` 不包含信息头，**`delete` 需要根据信息头来确定析构函数的调用次数**。 `new []` 申请的内存用 `delete` 来释放对象的提前是：对象的类型是内置类型或者是无自定义的析构函数的类。

现在来探究一下，为什么 `new []` 也能用 `delete` 释放？（有一说一，这个问题比较蠢，因为这样写代码会被骂死吧）

现在来看几个有趣的现象，后续会通过简单的实验验证它们。

1. 如果是 `int` 等基本数据类型：使用 `new []` 申请内存，使用 `delete` 释放，不会内存泄漏。
2. 如果是 `class`：
   - 2.1 实现了析构函数：使用 `delete` 去释放 `new []` 申请的内存，会发生运行时错误。
   - 2.2 没有实现析构函数：可以使用 `delete` 去释放 `new []` 申请的内存，不会内存泄漏。

## 实验

下列实验均在 MacOS 上进行。

下面设计实验对上述的现象验证。如何说明内存不泄漏？且看如下代码：

```cpp
void test1()
{
    // 不会内存泄漏
    int *p = new int[4];
    cout << p << endl;  // 0x7f7fd6c05a10
    delete p;
    int *q = new int[4];
    cout << q << endl;  // 0x7f7fd6c05a10
}
```

 不论运行多少次，上述 `p, q` 的值总是一样的。说明 `delete p` 把 4 个 `int` 的内存都成功回收，因此在下一次申请内存时才会从 `p` 的起始位置开始。该程序验证上述的第 1 个现象。

> 按照我原来的理解，我以为 `delete p` 仅仅会释放 `p` 指向的 4 个字节的内存，然后会造成剩下 12 字节的内存泄漏。

再看看 `class` 时的情况。

```cpp
class A { public: int val; };
void test2()
{
    // 也不会内存泄漏
    auto p = new A[4];
    cout << p << endl;
    delete p;
    auto q = new A[4];
    cout << q << endl;
}
```

 这里 `p, q` 的值也总是相同的，可验证上述的现象 2.2 。

如果加入析构函数：

```cpp
class B
{
public:
    int val;
    B() : val(0) { cout << "B" << endl; }
    ~B() { cout << "~B" << endl; }
};
void test3()
{
    auto p = new B[4];
    delete p;
}
```

上述代码输出 `B` 三次，`~B` 一次，然后程序崩溃，后续会解析原因。现在把 `test3` 改为如下代码：

```cpp
void test3()
{
    cout << sizeof(B) << endl; // 4
    auto p = new B[4];
    cout << p << endl; // 0x7fda10405a18
    auto q = new B[4];
    cout << q << endl; // 0x7fef51405a38
}
```

这里需要注意的是，`B` 的大小是 4 字节，因此 `p` 的内存长度应为 16 字节，`q` 应该从 `0x5a18 + 0x0010 = 0x5a28` 开始才对，但实际上 `q` 从 `0x5a38` 开始，那么为什么中间的 16 字节做什么去了呢？这里的 16 字节就是文章开始提到的 `new []` 加入的信息头。

定义 `print4int(p)` ，它的作用是输出 `p` 前面 16 个字节的内容，每 4 个字节以 `int32_t` 的形式输出。例如 `p = 16` 时，该函数输出 `0 - 15` 这 16 个字节。

```cpp
void print4int(int32_t *p)
{
    p -= 4;
    cout << "memory at " << p << ": ";
    for (int i = 0; i < 4; i++)
        cout << p[i] << ' ';
    cout << endl;
}
```

 下面再看 `test3` ：

```cpp
void test3()
{
    auto p = new B[4];
    cout << p << endl;        // 0x7fe0b5c05a18
    print4int((int32_t *)p);  // memory at 0x7fe0b5c05a08: 0 0 4 0
    auto q = new B[16];
    cout << q << endl;        // 0x7fe0b5c05a38
    print4int((int32_t *)q);  // memory at 0x7fe0b5c05a28: 0 0 16 0 
}
```

修改数组长度，重复测试，可以发现，前面 4 个 `int` 中，第 3 个 `int` 总是与 `new []` 申请的长度一致。（实际上，只有 2 个 `int` 是信息头，多出来的 2 个是由于字节对齐的缘故。）

这时候问题来了，为什么 `new int[4]` 这种情况就没有信息头呢？这是由于编译器的原因，如果编译器检查到，如果是 `base type`，`new[]` 操作就会省去前面的信息头，后面发生 `delete` 释放 `new[]` 的时候，`free` 调用的 `ptr` 和 `malloc` 调用的 `ptr` 也就一致了。这就是为什么 `base type` 可以用 `delete` 来释放 `new []` 。



## 分析

我们都知道，`new/delete` 最终是调用 `malloc/free` 来完成内存的申请和释放的，此外 `new/delete` 会调用构造/析构函数，这是  `malloc/free` 没有做的。

例如 `p = new B[4]` 完成的工作有 2 个，一是申请内存，二是执行 `B` 的构造函数（4次）；同理 `delete[] p` 类似，先执行 `B` 的析构函数（4 次），然后释放内存。

`new []` 申请的内存布局如下图所示，`Bookkeeping` 就是指上述的所谓的「信息头」。

```c
| Bookkeeping | First Object | Second Object |....
^             ^
|             |
|             p2: This is what is returned by new[]
|
p1:this is what is returned by malloc()
```

这能够解释当数据类型为某个类时，为什么用 `delete` 去释放 `new []` 会失效。

在程序当中，我们用 `ptr = new B[4]` 得到的是上述的 `p2` 。如果我们使用 `delete ptr` 去释放，先调用第一个对象 `B[0]` 的析构函数，然后调用 `free(ptr)` ，但是在 C++ 的内存管理当中，记录的是上述 `p1` 的值，`ptr` 不被认为是申请的内存地址，所以 `free(ptr)` 会发生 `pointer being freed was not allocated` 的错误（MacOS上的输出）。

而使用 `delete[] ptr`，则先根据信息头调用若干次析构函数，然后在调用 `free(ptr - X)` ，`X` 是 Bookkeeping 的长度。

既然清楚了这个细节，我们也就能骚操作一下：使用 `free / delete` 来释放 `new B[4]` 。

```cpp
void test4()
{
    auto p = new B[4];
    cout << p << endl;       // 0x7f8f14405a18
    print4int((int32_t *)p); // memory at 0x7f8f14405a08: 0 0 4 0

    uint32_t *q = (uint32_t *)p;
    q -= 2;
    cout << q << endl;       // 0x7f8f14405a10
    delete q;
    // free(q) is also ok
}
// 但这仍旧是不安全的，可能会造成内存泄漏的，因为 4 个 B 对象析构函数没有被合理调用
```

## 结论

To be honest，写代码只要遵循 `new/delete` 和 `new[]/delete[]` 配对使用的原则，然后就没这文章什么事了。

（搞了半天，这篇文章有点像在「无病呻吟」🤡）