## 剑指 Offer 面试题

本文是《剑指 Offer》一书中出现的面试题整理。

TODO List:

+ 题解：DFA求解，正则表达式（19）和表示数字的字符串（20）
+ 

## 第二章：面试基础知识

### 拷贝构造函数的参数

下面代码存在什么问题？

```cpp
class A
{
public:
    int val;
    A() { val = 0; };
    A(A a) { val = a.val; }
};
int main()
{
    A a1;
    A a2 = a1;
}
```

答：编译出错或者栈溢出。`A a2 = a1;` 这一行代码会调用拷贝构造函数，但由于拷贝构造函数的参数是**值传递**，实参向形参传递过程中会继续调用拷贝构造函数，因此会栈溢出。把参数改为 `const A &a` 即可修正错误。

 ### 重载赋值运算符 = 

给定下面类型，要求实现重载赋值运算符的函数。

```cpp
class MyString
{
public:
    MyString();
    MyString(const MyString &mystr);
    ~MyString();
private:
    char *data;
};
```

需要考虑下面 4 个问题：

+ 返回类型是否为**引用**。只有声明为引用，并且返回 `*this` 才能允许连续赋值（连续赋值是指 `a = b= c;` 这种形式的语句），否则不能通过编译。
+ 传入参数类型是否为常量引用 `const MyString &`。如果不是引用，需要多一次拷贝构造函数调用。
+ 是否释放原来实例已有的内存。如果没有，会造成内存泄漏。
+ 传入参数与当前实例（即 `*this` ）是否为同一实例。如果是，那么在释放 `*this` 已有内存的时候，传入参数的内存也被释放，赋值内容丢失。

初级代码如下：

```cpp
const MyString &MyString::operator=(const MyString &mystr)
{
    if (this == &mystr)
        return *this;
    if (data != nullptr)
        delete[] data;
    data = new char[strlen(mystr.data) + 1];
    strcpy(data, mystr.data);
    return *this;
}
```

但存在一个缺点，如果 `new` 申请内存失败（内存不足），那么 `data` 是一个空指针，有 2 个不良后果：

+ 在 `strcpy` 执行出错，程序崩溃。

+ **赋值失败**，并且原来的内容也被 `delete` 了。此外，当前实例不是一个有效状态，违背了异常安全性（Exception Safety）原则。

2 种解决办法：

1. 先申请后释放。如果 `new` 成功才执行 `delete` ，否则返回或抛出异常。
2. 创建一个临时实例 `temp`，利用析构函数去释放当前实例的内存。如下代码：

```cpp
const MyString &MyString::operator=(const MyString &mystr)
{
    if (this != &mystr)
    {
        // 这里执行拷贝构造函数
        MyString temp(mystr);
        char *p = temp.data;
        temp.data = data;
        data = p;
        // 跳出 if 时，会自动执行 temp 的析构函数
    }
    return *this;
}
```

如果在拷贝构造函数中使用 `new` 失败，那么会抛出 `bad_alloc` 之类的异常，但在这里**当前实例的状态没有任何变动**。

