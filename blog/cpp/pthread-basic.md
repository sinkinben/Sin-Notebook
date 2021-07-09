## pthread 多线程基础

本文主要介绍如何通过 `pthread` 库进行多线程编程，并通过以下例子进行说明。

+ 基于莱布尼兹级数计算 $\pi$ .
+ 多线程归并排序

参考文章：

+ [1] https://computing.llnl.gov/tutorials/pthreads

## API 介绍

### pthread_create

作用：新建一个线程。

函数原型：

```c
int pthread_create(pthread_t *thread, const pthread_attr_t *attr, void *(*start_routine)(void *), void *arg);
```

参数解析：

+ `pthread_t *thread` 用于缓存新线程的 `pid` .
+ `const pthread_attr_t *attr` 制定新线程的 `attr` ，如果为 `NULL`，那么将使用默认的 `attr` 。
+ `start_routine` 是新线程即将进入的执行函数。
+ `arg` 向新线程传递的某些参数，一般封装为结构体传入。

线程的中止可以通过以下方式：

+ 调用 `pthread_exit(void *retval)` , 其中 `retval` 可以通过 `pthread_join` 获得。
+ 在 `start_routine` 函数直接 `return ` .
+ 该线程被取消 (See [pthread_cancel](https://man7.org/linux/man-pages/man3/pthread_cancel.3.html)) .
+ 线程所属的进程调用了 `exit` , 或者该进程的 `main` 函数中执行了 `return` .

### pthread_join

等待某个线程结束。

函数原型：

```c
int pthread_join(pthread_t thread, void **retval);
```

参数解析：

+ `thread` 是某个线程的 `pid` .

+ `retval` 用于获取线程 `start_routine` 的返回值 .

基本用法请看下面的「双线程计算 $\pi$」，该例子同时能够回答为什么 `retval` 是 `void**` 类型而不是 `void *` 类型。

### pthread_attr_t

`pthread_attr_t` 的定义如下：

```c
struct __pthread_attr
{
    struct sched_param __schedparam;
    void *__stackaddr;
    size_t __stacksize;
    size_t __guardsize;
    enum __pthread_detachstate __detachstate;
    enum __pthread_inheritsched __inheritsched;
    enum __pthread_contentionscope __contentionscope;
    int __schedpolicy;
};
```

与之相关的 API，请看：

```bash
man pthread_attr
```

## Examples

### 创建线程

下面是一个简单的多线程例子，用于演示 `pthread_create` 和 `pthread_join` 的基本用法。

该例子创建 4 个线程，通过 `order[i]` 分别标号，线程的工作内容是输出本线程的标号。 

所涵盖的知识点：

+ 如何创建线程
+ 如何向线程传递参数：通过对 `void *arg` 进行强制类型转换实现。
+ `pthread_join` 的作用：如果去掉 `pthread_join` 调用，那么程序很可能是没有输出的。因为在进入各个线程的 `worker` 函数时，`main` 函数已经结束，这时候所有线程都被强制终止。

```c
#include <stdio.h>
#include <pthread.h>
const int N = 4;
void* worker(void *arg)
{
    int *pid = (int *)arg;
    printf("%d ", *pid);
    return NULL;
}
int main()
{
    pthread_t pid[N] = {0};
    const int order[] = {0, 1, 2, 3};
    int i = 0;
    for (; i < N; i++)
        pthread_create(&pid[i], NULL, worker, (void *)&order[i]);
    for (i = 0; i < N; i++)
        pthread_join(pid[i], NULL);
    return 0;
}
```

### 双线程计算 $\pi$

要求：

+ 基于莱布尼兹级数：1 - 1/3 + 1/5 - 1/7 + 1/9 - ... = PI/4
+ 使用主线程 + 辅助线程的方式

涵盖知识点：

+ 如何向不同的线程传递不同参数
+ 如何获取线程的结果

代码实现：

```c
#include <stdio.h>
#include <pthread.h>
#include <stdlib.h>
const int N = 1e8;
typedef struct { int start, end; } param_t;
typedef struct { double value; } result_t;
void *worker(void *arg)
{
    param_t *param = (param_t *)arg;
    result_t *res = (result_t *)malloc(sizeof(result_t));
    int i = param->start;
    for (; i <= param->end; i++)
    {
        if (i % 2) res->value += 1.0 / (2 * i - 1);
        else res->value -= 1.0 / (2 * i - 1);
    }
    return res;
}

double master(void *arg)
{
    double res = 0.0;
    param_t *param = (param_t *)arg;
    int i = param->start;
    for (; i <= param->end; i++)
    {
        if (i % 2) res += 1.0 / (2 * i - 1);
        else res -= 1.0 / (2 * i - 1);
    }
    return res;
}

int main()
{
    pthread_t tid = 0;
    param_t p1 = {1, N / 2}, p2 = {N / 2 + 1, N};
    pthread_create(&tid, NULL, worker, &p2);
    double val = master(&p1);
    result_t *res = NULL;
    pthread_join(tid, (void **)&res);
    printf("PI = %f\n", 4 * (val + res->value));
    free(res);
}
```

### 多线程计算 $\pi$

要求：

+ 适应 N 核心的 CPU
+ 不能使用全局变量，必须通过传递参数与 `join` 获取返回值实现

代码实现：

```c
#include <stdio.h>
#include <pthread.h>
#include <stdlib.h>
const int N = 1e3;
const int NR_CPU = 8;
typedef struct { int start, end; } param_t;
typedef struct { double value; } result_t;
void *worker(void *arg)
{
    param_t *p = (param_t *)arg;
    int i = p->start;
    result_t *res = malloc(sizeof(result_t));
    for (; i < p->end; i++)
    {
        if (i % 2) res->value += 1.0 / (2 * i - 1);
        else res->value -= 1.0 / (2 * i - 1);
    }
    return res;
}

int main()
{
    param_t params[NR_CPU];
    pthread_t pids[NR_CPU] = {0};
    const int step = N / NR_CPU;
    int i = 0;
    for (; i < NR_CPU; i++)
    {
        params[i].start = i * step + 1;
        params[i].end = params[i].start + step;
    }
    params[NR_CPU - 1].end = N;
    for (i = 0; i < NR_CPU; i++) pthread_create(&pids[i], NULL, worker, &params[i]);
    result_t *res = NULL;
    double pi = 0.0;
    for (i = 0; i < NR_CPU; i++)
    {
        pthread_join(pids[i], (void **)&res);
        pi += res->value;
        if (res) free(res), res = NULL;
    }
    pi *= 4;
    printf("PI = %f\n", pi);
    return 0;
}
```

### 多线程归并排序

要求：

+ 把数组分为若干个区间，每个区间单独通过一个线程排序。
+ 最后在主线程，所有区间通过归并完成排序。

```c
#include <assert.h>
#include <stdio.h>
#include <pthread.h>
#include <stdlib.h>
#include <string.h>
const int N = 1e6;
const int NR_CPU = 4;
typedef struct
{
    int *nums;
    int start, end;
} param_t;
int check(const int *nums, int len)
{
    int i = 1;
    for (; i < len; i++)
        if (nums[i] < nums[i - 1])
            return 0;
    return 1;
}
int cmp(const void *a, const void *b) { return (*(int *)a) - (*(int *)b); }
void *worker(void *arg)
{
    param_t *p = (param_t *)arg;
    int *start = p->nums + p->start;
    int n = p->end - p->start;
    qsort(start, n, sizeof(int), cmp);
    return NULL;
}
// merge [start, mid) and [mid, end)
void merge(const int *nums, int start, int mid, int end)
{
    int *p = malloc(sizeof(int) * (end - start));
    const int *p1 = nums + start, *p2 = nums + mid;
    int len1 = mid - start, len2 = end - mid;
    int idx = 0, i = 0, j = 0;
    while (i < len1 && j < len2)
    {
        if (p1[i] < p2[j]) p[idx++] = p1[i++];
        else p[idx++] = p2[j++];
    }
    while (i < len1) p[idx++] = p1[i++];
    while (j < len2) p[idx++] = p2[j++];
    memcpy((void *)(nums + start), (void *)p, sizeof(int) * idx);
}
int main()
{
    srand(time(NULL));
    int nums[N] = {0};
    int i = 0;
    for (; i < N; i++) nums[i] = random() % N;

    param_t params[NR_CPU];
    int step = N / NR_CPU;
    for (i = 0; i < NR_CPU; i++)
    {
        params[i].nums = nums;
        params[i].start = i * step;
        params[i].end = params[i].start + step;
    }
    params[NR_CPU - 1].end = N;
    pthread_t pids[NR_CPU] = {0};
    for (i = 0; i < NR_CPU; i++) pthread_create(&pids[i], NULL, worker, &params[i]);
    for (i = 0; i < NR_CPU; i++) pthread_join(pids[i], NULL);
    while (step < N)
    {
        int start = 0;
        while (start < N)
        {
            int mid = start + step;
            int end = mid + step;
            if (mid > N) mid = N;
            if (end > N) end = N;
            merge(nums, start, mid, end);
            start = end;
        }
        step *= 2;
    }
    assert(check(nums, N));
}
```

