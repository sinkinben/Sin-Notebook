## [APUE-11] 线程



## 线程标识

主要有 2 个相关的 API：

+ `pthread_equal`
+ `pthread_self`

## 线程创建

很常见的API：

+ `pthread_create`

## 线程终止

如果线程调用 `exit` 那么整个进程都会退出。线程退出以下有 3 种方式：

- 从线程 `worker` 函数返回
- 被同一进程中的其他线程取消，即调用 `pthread_cancel(pthread_t thread)` .
- 线程调用 `pthread_exit`

`pthread_exit` 的函数原型如下：

```c
void pthread_exit(void *retval);
```

该 `retval` 参数可以通过 `pthread_join` 来获取，但需要保证 `retval` 指向的内存区域在线程返回后仍然生效（即不能返回一个在线程栈上的局部变量），可以是一个全局变量或者 `malloc` 申请的变量。

### pthread_cancel

对于 `pthread_cancel(tid)` 而言，该函数仅仅是提出一个请求，并不意味着线程 `tid` 必须要终止。

函数原型：

```c
int pthread_cancel(pthread_t thread);
```

