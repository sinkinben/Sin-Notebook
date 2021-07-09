## C++ 面试题

整理日常遇到的 C++ 问题。

## new/delete 和 malloc/free 的区别

关于 `new[]` 可参考：https://www.cnblogs.com/sinkinben/p/14363115.html

`new/delete` 会调用类的构造/析构函数，底层实际调用的是 `malloc/free` ，但 `malloc/free` 不会调用构造/析构函数。



## 指针数组和数组指针

```cpp
char *strs[4] = {"1", "2", "3", nullptr};  //指针数组，每个元素都是指针
char (*pstr)[4]; //数组指针，是一个指针，指向一个 char[4] 数组
```

详细例子如下：

```cpp
int main()
{
    char s[4] = "123";
    char *strs[4] = {"1", "2", "3", nullptr}; //指针数组，每个元素都是指针
    char(*pstr)[4];                           //数组指针，是一个指针，指向一个 char 数组
    pstr = &s;
    cout << s << endl;
    cout << pstr << endl;
}
```



## 深拷贝与浅拷贝

```cpp
class Test
{
    int *p;  // p points to a memory
    Test(): P(new int[10]) {}
};
int main()
{
    // shallow copy
    Test t1;
    Test t2 = t1;
}
```

浅拷贝：没有自定义拷贝构造函数，按位拷贝，直接把 `t1.p` 的地址赋值给 `t2.p`。

深拷贝：自定义一个拷贝构造函数，申请一段新内存，把 `t1.p` 的数据，复制到 `t2.p` 。



## 构造函数

- 可以不写，编译器自动补上默认的
- 有参数/无参数的构造函数：`Test(), Test(int x, ...)`
- 拷贝构造函数（一般用于实现深拷贝）：`Test(const Test &t)`
- 移动构造函数（一般用于实现浅拷贝，转移实例对象的状态和所有权）：`Test(Test &&t)` . C++ 11 后为了实现 `move` 语义而加入的。 

构造与析构的顺序可以参考：https://www.cnblogs.com/sinkinben/p/13773825.html



## struct 和 class

- 成员变量/函数的默认权限：struct 是 public 的，class 是 private 。
- 默认的继承访问权限：struct 是 public 的，class 是 private 的。



## 指针和引用

编译得到的机器码是类似的，都是获取变量的地址方式实现。引用只是C++对指针操作的一个“语法糖”，在底层实现时C++编译器实现这两种操作的方法完全相同。

|          指针          |         引用         |
| :--------------------: | :------------------: |
| 可以不初始化，可以为空 | 必须初始化，不能为空 |
|   可以更换指向的目标   |  不能更换指向的目标  |



## 虚函数与纯虚函数

See Blog：https://www.cnblogs.com/sinkinben/p/13775706.html



## 重载、重写与重定义

- 重载 (Overload) ：函数名相同，参数列表不同。
- 重写 (Override) : 类 `A` 定义了一个**虚函数** `func`，其子类 `B` 把函数 `func` 重新实现（aka, 覆盖）。
- 重定义 (Refine) : 类 `A` 定义了一个**普通函数** `func`，其子类 `B` 把函数 `func` 重新实现。

那么 Override 和 Refine 的区别在哪？答案是动态绑定。

比如下面的 Override 的例子：

```cpp
class A { public: virtual void say() { cout << "fuck" << endl; } };
class B : public A { void say() { cout << "you" << endl; } };
int main()
{
    A *a = new B();
    a->say(); // you
}
```

由于虚函数动态绑定的特性，`a` 指针调用的是 `B` 的 `say` 。

再看 Refine 的例子：

```cpp
class A { public: void say() { cout << "fuck" << endl; } };
class B : public A { void say() { cout << "you" << endl; } };
int main()
{
    A *a = new B();
    a->say(); // fuck
}
```

这就是区别.jpg

## 构造函数为什么不能被声明为虚函数

跟虚函数表有关。指向虚函数表的指针需要在构造之后才被创建。

尽管虚函数表 vtable 是在编译阶段就已经建立的，但指向虚函数表的指针 `vptr` 是在运行阶段实例化对象时才产生的。 如果类含有虚函数，编译器会在构造函数中添加代码来创建 `vptr`。 如果构造函数是虚的，那么它需要 `vptr` 来访问 vtable，可这个时候 `vptr` 还没产生。 因此，构造函数不可以为虚函数。

之所以使用虚函数，是因为需要在信息不全的情况下进行多态运行。而构造函数是用来初始化实例的，实例的类型必须是明确的。 因此，构造函数没有必要被声明为虚函数。

## 简述 C++ 中内存对齐的使用场景

就是所谓的字节对齐，ICS 课程的内容。

```cpp
struct node 
{
    char ch;
    int value;
};
// 常见的是 4 字节对齐，因此 sizeof(node) == 8
```

不同平台上编译器的 `pragma pack` 默认值不同。而我们可以通过预编译命令 `#pragma pack(n), n= 1,2,4,8,16` 来改变对齐系数。

使用场景：

- 平台移植：不是所有的硬件平台都能访问任意地址上的任意数据的；某些硬件平台只能在某些地址处取某些特定类型的数据，否则抛出硬件异常。 
- 提高性能：数据结构 (尤其是栈) 应该尽可能地在自然边界上对齐。原因在于，为了访问未对齐的内存，处理器需要作两次内存访问；而对齐的内存访问仅需要一次访问。

## 类模板和模板类

类模版：`vector<typename>, string<>` 这一类东西，基于关键字 `template` 来实现。

模版类：类模板实例化后的一个产物。

这里的一个重要知识点是：模版类是编译期产生的。还是通过一个例子来说明:

```cpp
template <typename T>
class ivector
{
public:
    T *data;
    ivector(int size = 10) : data(new T[size]) {}
};
int main()
{
    ivector<int> v1;
    ivector<char> v2;
}
```

 上面编译过后，也就是说在运行时，其实会产生 2 个类（如下），这就是模版的作用。

```cpp
/* First instantiated from: insights.cpp:14 */
#ifdef INSIGHTS_USE_TEMPLATE
template<>
class ivector<int>
{
public: 
  int * data;
  inline ivector(int size)
  : data{new int[static_cast<unsigned long>(size)]} {}
};
#endif

/* First instantiated from: insights.cpp:15 */
#ifdef INSIGHTS_USE_TEMPLATE
template<>
class ivector<char>
{
public: 
  char * data;
  inline ivector(int size)
  : data{new char[static_cast<unsigned long>(size)]} {}
};
#endif

int main()
{
  ivector<int> v1 = ivector<int>(10);
  ivector<char> v2 = ivector<char>(10);
}
```



## C++ 的内存分布

一张经典的图片（就是不知道在哪本教材里面截的）。

<img src="https://gitee.com/sinkinben/pic-go/raw/master/img/20210203201428.png" style="width:70%;" />

好像这张图更好看一些。

<img src="https://gitee.com/sinkinben/pic-go/raw/master/img/20210307105919.png" />



## malloc/free 的实现

https://blog.csdn.net/yeditaba/article/details/53443792



## C++ 编译的过程

预处理、编译、汇编、链接。

<img src="https://gitee.com/sinkinben/pic-go/raw/master/img/20210203223819.png" style="" />



## C++ 右值引用和移动语义

- 解决的问题：某些临时对象（即所谓的「右值」）的拷贝开销很大
- `move` 把一个左值转换为右值。
- `forward` 函数传参时保持原有的引用特性，即 `x` 为左值，那么 `forward(x)` 依然是一个左值，当 `x` 为右值时同理。