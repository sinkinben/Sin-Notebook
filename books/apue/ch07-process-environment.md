## [APUE] 进程环境

📓 APUE 一书的第七章学习笔记。

## 进程终止

有 8 种方式可以使得进程终止，5 种为正常方式：

1. Return from `main`
2. Calling `exit()`
3. Calling `_exit` or `_Exit`
4. Return of the last thread from its start routine
5. Calling `pthread_exit` from the last thread

3 种非正常终止方式：

1. Calling `abort`
2. Receipt of a signal
3. Response of the last thread to a cancellation request

### 退出函数

函数原型：

```c
#include <stdlib.h> 
void exit(int status); 
void _Exit(int status); 
#include <unistd.h> 
void _exit(int status);
```

区别：`_exit/_Exit` 会立即进入内核；`exit` 先执行清理处理（对所有打开的 Stream 执行 `fclose` ），后进入内核。

`exit(k)` 相当于在 `main` 函数中 `return k` ，最近一个进程的退出码可以在 Shell 中使用 `echo $?` 获取。

### 回调函数 atexit

按照 ISO C 标准，一个进程最多可以登记 32 个回调函数，这些函数称为终止处理程序 (exit handler)，由 `exit` 来调用。

函数原型：

```c
int atexit(void (*func)(void));
// Returns: 0 if OK, nonzero on error
```

`exit` 调用终止处理函数的顺序与登记顺序相反（`_exit/_Exit` 则不会调用）；如果登记多次，也会被调用多次。

例子：

```c
void test1() { printf("A "); }
void test2() { printf("B "); }
int main()
{
    atexit(test1);
    atexit(test1);
    atexit(test2);
    // 如果调用下面 2 个函数，则不会有输出
    // _exit(0), _Exit(0)
}
// Output: B A A
```

下图展示了一个 C 程序如何启动和终止的过程，也显示了 `exit, _exit, _Exit, atexit` 这 4 个函数的关系。

<img src="https://gitee.com/sinkinben/pic-go/raw/master/img/20210125192440.png" style="width:70%;" />

## 环境表

环境表 (Environment List), 与命令行参数 `argv` 一样，是一个 `char*` 数组。

下列程序可以打印所有的环境参数：

```c
#include <stdio.h>
extern char **environ;
int main()
{
    int i;
    for (i = 0; environ[i] != NULL; i++)
        puts(environ[i]);
}
```

 输出一大片：

```
TERM=xterm-256color
SHELL=/bin/bash
...
USER=sinkinben
LS_COLORS=rs=0:di=01;...
PATH=...
MAIL=/var/mail/sinkinben
PWD=/home/sinkinben/workspace/apue
HOME=/home/sinkinben
...
_=./a.out
```

## C 存储布局

如下图所示，一个 C 程序由以下部分组成：

- text 段：这是 CPU 执行的机器指令部分。text 段通常是只读（防止意外修改）和可共享的，即使是频繁执行的程序，它们的 text 段在内存中只会存在 1 个副本。（难怪我的 Mac 开机第一次调用 clang++ 会特别慢）
- 初始化数据段（简称数据段）：包括初始化的全局变量和 `static` 修饰的变量。
- 未初始化的数据段（简称 bss 段）：在程序开始执行前，由 `exec` 初始化为 0 或者空指针。（确实，我还以为是随机值）
- 栈：局部变量，函数调用。
- 堆：动态内存分配。

<img src="https://gitee.com/sinkinben/pic-go/raw/master/img/20210125193649.png" style="width:450px" />

这是一种典型的逻辑布局，但不是所有的实现都是如此，具体取决于实际的 OS 和硬件。对于 32 位 Intel x86 架构的 Linux，text 段从 0x80480000 开始，栈从 0xC0000000 开始向低地址增长。

可以通过 `size` 命令获取一个 C 程序的各个段大小，`dec, hex` 分别是前 3 个数字的总和的十进制和十六进制：

```text
$ size /bin/bash 
   text    data     bss     dec     hex filename
 997958   36496   23480 1057934  10248e /bin/bash
```

## 共享库

共享库 (Shared Libraries): 对于一些常用的公共函数库，只需要在所有进程都可引用的存储区保存一份副本。程序第一次调用某个库函数的同时，用**动态链接**的方式将程序与库函数相链接。这就减少了可执行文件的大小，但增加了一些运行时开销。

使用 Shared Libraries 的另外一个优点是：可以动态更新某个库的版本，而不需要重新对使用该库的程序重新链接。

使用 `gcc -static` 可指定生成的可执行文件使用静态链接。

```text
$ gcc test.c ; size ./a.out 
   text    data     bss     dec     hex filename
   1099     544       8    1651     673 ./a.out
$ gcc test.c -static ; size ./a.out 
   text    data     bss     dec     hex filename
 823142    7284    6360  836786   cc4b2 ./a.out
```



## 动态内存分配

相关函数：

```c
#include <stdlib.h>
void *malloc(size_t size);
void *calloc(size_t nobj, size_t size); 
void *realloc(void *ptr, size_t newsize);
// All three return: non-null pointer if OK, NULL on error
void free(void *ptr);
```

作用：

- `malloc`: 申请指定字节数的内存，该内存的值不确定。
- `calloc`: 为指定数量，指定长度的对象分配内存，并初始化为 0 。
- `realloc`: 增加或者减少 `ptr` 指向内存区的长度。当增加长度时，可能需要将之前的数据拷贝到另外一个足够大的内存区（也有可能是在原有基础上增加一段连续内存），新增区域的初始值不确定。

## 环境变量

### getenv

```c
#include <stdlib.h>
char *getenv(const char *name);
// Returns: pointer to value associated with name, NULL if not found
```

获取环境变量。例子：

```c
#include <stdio.h>
#include <stdlib.h>
int main()
{
    puts(getenv("JAVA_HOME"));
}
// Output: /usr/local/java/jdk1.7
```

### putenv, setenv, unsetenv

相关 API：

```c
#include <stdlib.h> 
int putenv(char *str);
// Returns: 0 if OK, nonzero on error
int setenv(const char *name, const char *value, int rewrite); 
int unsetenv(const char *name);
// Both return: 0 if OK, −1 on error
```

改变当前进程以及后续产生的子进程的环境变量（实际上是修改进程的环境表）。作用分别如下：

- `putenv`: 把 `name=value` 的环境变量添加到环境表，如果 `name` 已存在，则删除原来的定义。
- `setenv`: 将 `name` 设置为 `value` 。`rewrite = 0/1` 表示是否覆盖已有的 `name` （如果有的话）。 
- `unsetenv`: 删除 `name` ，如果不存在则什么都不做。

## getrlimit, setrlimit

每个进程都有一组资源限制，可以通过 `getrlimit, setrlimit` 进行查询和修改：

```c
#include <sys/resource.h>
int getrlimit(int resource, struct rlimit *rlptr);
int setrlimit(int resource, const struct rlimit *rlptr);
// Both return: 0 if OK, −1 on error
```

`resource` 是形如 `RLIMIT_CPU` 的一组宏定义。

结构体 `rlimit` 的定义如下：

```c
 struct rlimit {
     rlim_t  rlim_cur;  /* soft limit: current limit */
     rlim_t  rlim_max;  /* hard limit: maximum value for rlim_cur */
};
```

修改操作需要遵循以下规则：

- `cur <= max`
- 可以降低 `rlim_max` ，但必须大于等于 `rlim_cur`. 
- 只有超级用户进程可以提高 `rlim_max` 。

修改资源限制会影响当前进程和它的子进程，所以 Shell 中一般会内置 `ulimit` 命令来修改当前 Shell 的资源限制。



## setjmp 和 longjmp

C 语言中，`goto` 语句是不能跨越函数的，而通过 `setjmp` 和 `longjmp` 则可以实现。

```c
#include <setjmp.h>
int setjmp(jmp_buf env);
// Returns: 0 if called directly, nonzero if returning from a call to longjmp 
void longjmp(jmp_buf env, int val);
```

`env` 参数用于保存跳转**目标点**的上下文的变量，通过 `jmp_buf` 这一特殊数据类型定义。

`val` 是 `longjmp` 跳转到 `setjmp` 的目标点时到返回值。

例子：

```c
#include "apue.h"
#include <setjmp.h>
jmp_buf jmpbuf;
void f3() { longjmp(jmpbuf, 1); }
void f2() { f3(); }
void f1() { f2(); }
int main()
{
    switch (setjmp(jmpbuf))
    {
    case 0:
        puts("0");
        f1();
        break;
    case 1:
        puts("1");
        break;
    default:
        puts("default");
        break;
    }
}
// Output: 0 1
```

如果 `f3` 调用的是 `longjmp(jmpbuf, 233)` ，那么输出 `0 default` .



