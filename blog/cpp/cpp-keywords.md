## C++ Keywords

本文是关于 C++ 的学习笔记，主要内容是 C++ 中关键字的剖析。

## sizeof

### sizeof 在数组中的坑

众所周知，`sizeof` 用于获取某个变量/数组的字节数。下面是一个关于 `sizeof` 的坑：

```cpp
void test2()
{
    auto func = [](char str[]) {
        cout << sizeof(str) << endl;
    };
    char str[] = "12";
    cout << sizeof(str) << endl;
    func(str);
}
// 64位机器上输出：3 8
```

这第一次遇见属实有些纳闷，我原本想应该是输出数组的字节数才对，等我使用 `g++ -std=c++17 ` 编译，结果有一个 `warning` : 

```
warning: sizeof on array function parameter will return size of 'char *' instead of 'char []'
    auto func = [](char str[]) { cout << sizeof(str) << endl; };
```

应该都读得懂英文吧（摊手），也就是说，`char str[]` 被编译器转换为 `char *` 了。

### sizeof 和类

来自《剑指 Offer 》一书。

1. 定义一个空类 `A` ，没有成员变量和函数，`sizeof(A)` 是多少？是 0 吗，为什么？

答：至少为 1. 因为空类型的实例不包含任何信息，本要求 `sizeof(A)` 是 0 的，但是当我们声明这个类型的实例时，它必须在内存中占据一定空间（或者说必须要分配一个内存地址），否则无法使用这个实例。因此空类型的大小至少为 1 ，具体由编译器来决定。

2. 如果加入构造与析构函数呢？

答：还是 1 . 调用函数只需要知道这个函数的地址即可，这些地址只和**类型**相关，而与**类的实例**无关。编译器不会因为成员函数在类的实例中添加额外信息。

3. 如果析构函数标记为虚函数呢？

答：32 位机器上是 4，64 位机器上是 8。编译器如果发现一个类中有虚函数，就会为该类型生成一个**虚函数表**，并在每一个实例中添加一个**指向虚函数表的指针**。

## static

老生常谈的 DATA 段和 BSS 段（合称「全局静态存储区」）：

1. DATA 段（全局初始化区）存放初始化的全局变量和静态变量。
2. BSS 段（全局未初始化区）存放未初始化的全局变量和静态变量，在程序执行之前会被系统自动清 0 。

static 的作用：

- 修饰变量
  - 修饰局部变量：在修饰变量的时候，static 修饰的静态局部变量只执行初始化一次，而且延长了局部变量的生命周期，直到程序运行结束以后才释放。
  - 修饰全局变量：static 修饰全局变量的时候，这个全局变量只能在本文件中访问，不能在其它文件中访问，即便是 extern 外部声明也不可以。

- 修饰函数：static 修饰一个函数，则这个函数的只能在本文件中调用，不能被其他文件调用。

- 修饰类的成员变量：多个对象可以共享这一变量

- 修饰类的成员函数：不需要具体的实例对象也能调用，通过域操作符 `ClassName::funcName();` .

  



## const

作用：

- 修饰基本类型变量。如 `const int x = 1;` ，不能对 `x` 进行赋值操作。
- 修饰类变量。参考下面「const 与 类」。
- 修饰函数
  - 修饰返回值 `const int func() {}` ： 没有意义。
  - 修饰函数体 `int func() const {}` :  当且仅当 `func` 是类的成员函数才能这样使用，这样的 `const` 表示不允许 `func` 修改任何成员变量。

### const 与指针

const 与指针的组合有 4 情况：

```cpp
const int* s;        // 修饰 int*, 指向的内存不可变，s 的指向可变
int const *s;        // 同上，奇葩写法
int* const s;        // 修饰 s, 指向的内存可变，s 的指向不可变
const int* const s;  // 指向的内存不可变，s 的指向不可变
```

const 的修饰对象是「向右就近原则」。不能理解的话，还可以这么说：

+ 如果 `*` 位于 const 的右侧，那么指针指向的内存不可变。
+ 如果 `*` 位于 const 的左侧，那么指针变量本身不可变。

函数参数出现这几种情况的时候同理。一种常见的用法就是保证传入的参数不被改变，增强代码安全性。

```cpp
void f1(const int x)  {...}  // x 的值不可变
void f2(int* const p) {...}  // p 的指向不可变，与上同理
```

对于引用 `&` 来说（因为引用本身不可被赋值），就要简单一些：

```cpp
void f1(const vector<int> &v) {...}  // v 不可变
void f2(vector<int> const &v) {...}  // 同上，v 不可变
void f3(vector<int>& const v) {...}  // 错误函数定义
```



**代码样例**

```cpp
#include <iostream>
#include <vector>
using namespace std;
void f1(const vector<int> &v)
{
    // v.push_back(1);  // error
}
void f2(vector<int> const &v)
{
    // v.push_back(1);  // error
}
// void f3(vector<int>& const v) {} // error

int main()
{
    int k = 1;
    const int *p1 = &k;
    // *p1 = 2;  // error
    p1 = nullptr;// ok

    int const *p2 = &k;
    // *p2 = 2;  // error
    p2 = nullptr;// ok

    int *const p3 = &k;
    *p3 = 2;     // ok
    // p3 = nullptr;// error
    cout << k << endl;

    const int *const p4 = &k;
    // *p4 = 3;  // error
    // p4 = nullptr; // error
}
```

### const 与类

`const ` 修饰一个类变量，比如 `const vector<int> v` ，那么在 `v` 的生命周期内，不能调用 `push_back` 等会改变内部状态的函数。

例如：

```cpp
class Test
{
public:
    int value;
    Test(int v = 0) : value(v) {}
    void func1() const {}
    void func2() {}
};
int main()
{
    const Test t; // 必须提供构造函数
    t.func1();
    t.func2();    // 编译不过
}
```



## volatile

volatile 的主要作用是**避免编译优化**。

volatile 意为不稳定的，该关键字的作用是避免编译器对修饰对象进行编译优化。

被 `volatile` 修饰的变量，在对其进行读写操作时，会引发一些**可观测的副作用**。而这些可观测的副作用，是由**程序之外的因素决定的**。

在程序层面，其具体作用是：volatile 关键字声明的变量，每次访问时都必须从内存中取出值（没有被 volatile 修饰的变量，可能由于编译器的优化，从 CPU 寄存器中取值）

```cpp
volatile uint32_t *port = 0xff800000;
// uint32_t *port = 0xff800000;
void init()
{
    for (int i=0; i<5; i++)
        *port = i;
}
```

如果没有 `volatile` 修饰的话，使用 `g++ -S volatile.cpp -O3` 进行编译，代码就会被优化为：

```assembly
movl	_p, %eax
movl	$4, (%eax)
```

如果有 `volatile` 修饰，同样的编译命令：

```assembly
movl    $0, (%eax)
movl    $1, (%eax)
movl    $2, (%eax)
movl    $3, (%eax)
movl    $4, (%eax)
```

但在某些情况下，端口必须经过某几个值的初始化，编译器给优化掉，那么发生这种事情，大家都不想的，谁能想到缺失一个关键字就有 BUG 呢。

再比如，`port` 是一个输出端口，给外部设备显示一个倒计时，这也是必须要避免编译优化的情况。



## decltype

在泛型编程 (Generic Programming) 中用于推导返回值的类型。

```cpp
template<typename T1, typename T2>
auto multiple(T1 a, T2 b) -> decltype(a*b)
{
    return a*b;
}
```

如果被函数指针的奇怪写法困扰，也能通过 `decltype` 避免这一「困扰」：

```cpp
void close_file(FILE *fp) { std::fclose(fp); }
unique_ptr<FILE, decltype(&close_file)> fp(fopen("demo.txt", "r"), &close_file);
```

## explict

- 修饰构造函数：防止隐式转换

```cpp
class A
{
public:
    int value;
    A(int v) : value(v) { cout << "A" << endl; }
};
class B
{
public:
    int value;
    explicit B(int v) : value(v) { cout << "B" << endl; }
};
int main()
{
    A a = 10;                // A
    cout << a.value << endl; // 10
    B b = 233;               // compile error
}
```



## friend 

友元提供了一种普通函数或者类成员函数访问另一个类中的**私有或保护成员**的机制。也就是说有两种形式的友元：

- 友元函数：普通函数对一个访问某个类中的私有或保护成员。
- 友元类：类 A 中的成员函数访问类 B 中的私有或保护成员。

**友元函数**

```cpp
class A
{
    int value;

public:
    A(int v) : value(v) {}
    friend int get_value(const A &a);
};

int get_value(const A &a) { return a.value; }
int main()
{
    A a(233);
    cout << get_value(a) << endl; // 233
}
```

**友元类**

```cpp
class A
{
private: int value;
public:
    A(int v) : value(v) {}
    friend class B;
};
class B
{ public: static int get_value(const A &a) { return a.value; } };

class C : public B { };
int main()
{
    A a(233);
    cout << B::get_value(a) << endl;
    cout << C::get_value(a) << endl;
}
```

## virtual

主要作用就是声明一个[虚函数或者纯虚函数](https://www.cnblogs.com/sinkinben/p/13775706.html)。

- `static` 函数可以是虚函数吗？
  - 🙅‍♂️ 不可以。因为 `static` 函数不属于任何实例对象，是所有对象共享的。
- 构造函数可以为虚函数吗？
  - 🙅‍♂️ 不可以。构造函数只能用 `inline/explicit` 修饰。
- 析构函数可以为虚函数吗？
  - 🉑️ 可以。请看下面的例子。由此可以看出**析构函数不是动态绑定的**。

```cpp
class Animal { public: ~Animal() { cout << "~Animal" << endl; } };
class Cat : public Animal
{
public:
    void whoami() { cout << "I am cat." << endl; }
    ~Cat() { cout << "~Cat" << endl; }
};
int main()
{
    Animal *ptr = new Cat();
    delete ptr; // ~Animal
}
```

如果把 `Animal` 的析构函数改为 `virtual ~Animal() { ... }` ，`delete ptr` 将会调用 `Cat` 的析构函数，将会输出：

```
~Cat
~Animal
```

 