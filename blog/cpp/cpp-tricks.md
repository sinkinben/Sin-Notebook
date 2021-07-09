## CPP Tricks

记录一些关于 C/C++ 的奇技淫巧。



## 实现 offsetof

`offsetof(type, member)` 是位于 `stddef.h` 下的一个宏定义，作用是求出 `member` 在结构体 `type` 中的偏移量。

**标准解法**

```c
#define offsetof(type, member) (&(((type *)0)->member))
```

**lambda 函数**

```cpp
#define offset(type, member)  \
([]() { type obj; return (uintptr_t)(&obj.member) - (uintptr_t)(&obj); }())
```



## C 语言实现运行时多态

