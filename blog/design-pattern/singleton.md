## 单例模式

使用 C++ 实现单例模式。

## 版本一：static 函数

```cpp
class Singleton
{
private:
    static Singleton *ptr;
    Singleton() {}
public:
    static Singleton *getInstance()
    { return ptr == nullptr ? ptr = new Singleton() : ptr; }
};
Singleton *Singleton::ptr = nullptr;
int main()
{
    auto p1 = Singleton::getInstance();
    auto p2 = Singleton::getInstance();
    printf("%p %p", p1, p2); // 0x7fb701c059f0 0x7fb701c059f0
}
```

存在的问题：

- 多线程环境下，可能出现 `new` 多次的情况。
- 内存泄漏，因为没有运行析构函数。



## 版本二：静态局部对象

解决了版本一的 2 个问题。

```cpp

class Singleton
{
private:
    Singleton() { cout << "Singleton()" << endl; }
    ~Singleton() { cout << "~Singleton()" << endl; }
public:
    static Singleton *getInstance()
    {
        static Singleton singleton;
        return &singleton;
    }
};
int main()
{
    auto p1 = Singleton::getInstance();
    auto p2 = Singleton::getInstance();
    printf("%p\n%p\n", p1, p2);
}
// Singleton()
// 0x107b440e0
// 0x107b440e0
// ~Singleton()
```

但是还是有缺点：

-  `static` 变量是存放在程序的 DATA 段的，假如这个对象很大，会拖慢进程的启动速度。
- 不能按需分配，我们希望需要用到的时候才分配这个单例对象。



## 版本三：饿汉模式

在版本一的基础上，对指针加锁。

```cpp
class Singleton
{
private:
    Singleton() { cout << "Singleton()\n"; }
    ~Singleton() { cout << "~Singleton()\n"; }
    static Singleton *ptr;
    static pthread_mutex_t mutex;
public:
    static Singleton *getInstance()
    {
        pthread_mutex_lock(&mutex);
        if (ptr == nullptr) ptr = new Singleton();
        pthread_mutex_unlock(&mutex);
        return ptr;
    }
};
Singleton *Singleton::ptr = nullptr;
pthread_mutex_t Singleton::mutex = PTHREAD_MUTEX_INITIALIZER;
```

在内部实现一个额外的类 `GarbageCollect` ，解决内存泄漏问题。

```cpp
class Singleton
{
private:
    Singleton() { cout << "Singleton()\n"; }
    ~Singleton() { cout << "~Singleton()\n"; }
    static Singleton *ptr;
    static pthread_mutex_t mutex;
    class GarbageCollect
    {
    public:
        GarbageCollect() {}
        ~GarbageCollect()
        {
            if (Singleton::ptr != nullptr)
                delete Singleton::ptr;
        }
    };
    static GarbageCollect gc;

public:
    static Singleton *getInstance()
    {
        pthread_mutex_lock(&mutex);
        if (ptr == nullptr)
            ptr = new Singleton();
        pthread_mutex_unlock(&mutex);
        return ptr;
    }
};

Singleton *Singleton::ptr = nullptr;
pthread_mutex_t Singleton::mutex = PTHREAD_MUTEX_INITIALIZER;
Singleton::GarbageCollect Singleton::gc;
```

测试回收的有效性（多线程太简单就不测了）：

```cpp
int main()
{
    auto p1 = Singleton::getInstance();
    auto p2 = Singleton::getInstance();
    printf("%p\n%p\n", p1, p2);
}
// Singleton()
// 0x7f9b165059d0
// 0x7f9b165059d0
// ~Singleton()
```

