## pthread 互斥量

参考文献：

+ [1] https://computing.llnl.gov/tutorials/pthreads/

## 温故知新

在 OS 中，每个进程都独立地拥有：

+ Process ID, process group ID, user ID, and group ID
+ Environment
+ Working directory
+ Program instructions 
+ Registers
+ Stack
+ Heap
+ File descriptors
+ Signal actions
+ Shared libraries
+ Inter-process communication tools (such as message queues, pipes, semaphores, or shared memory).

因此，使用 `fork` 开启一个新的进程，需要拷贝很多数据，开销较大。

与进程不同，线程需要只需要独立拥有：

+ Stack pointer
+ Registers
+ Scheduling properties (such as policy or priority)
+ Set of pending and blocked signals
+ Thread specific data

需要特别注意的是，文件描述符和堆空间是进程独有的，因此该进程下面的所有线程都共用该进程的堆与文件描述符。

例如下面的代码：

```c
#include <stdlib.h>
#include <pthread.h>
#include <stdio.h>
#include <string.h>
void *worker1(void *arg)
{
    char *p = malloc(25);
    memcpy(p, "heap data from worker1", 23);
    return p;
};
void *worker2(void *arg)
{
    pthread_t tid1 = *(pthread_t *)arg;
    char *ptr = NULL;
    pthread_join(tid1, (void **)&ptr);
    printf("In worker2: ");
    if (ptr) puts(ptr);
    return NULL;
}
int main()
{
    pthread_t tid1, tid2;
    pthread_create(&tid1, NULL, worker1, NULL);
    pthread_create(&tid2, NULL, worker2, &tid1);
    pthread_join(tid2, NULL);
}
```

## 线程同步

在描述这个概念之前，先看一段代码：

```c
#include <stdlib.h>
#include <pthread.h>
#include <stdio.h>
#include <string.h>
const int N = 1e4;
int value = 0;
void *worker1(void *arg)
{
    int i = 1;
    for (; i <= N / 2; i++) value = value + i;
    return NULL;
};
void *worker2(void *arg)
{
    int i = N / 2 + 1;
    for (; i <= N; i++) value = value + i;
    return NULL;
}
int main()
{
    pthread_t tid1, tid2;
    pthread_create(&tid1, NULL, worker1, NULL);
    pthread_create(&tid2, NULL, worker2, NULL);
    pthread_join(tid1, NULL);
    pthread_join(tid2, NULL);
    printf("%d\n", value);
    printf("SUM(1, %d) should be %d .\n", N, N * (N + 1) / 2);
}
```

显然，我们想通过 2 个线程实现 `SUM(1, N)` 这个功能，但是编译多次你会发现，`value` 的值并不准确，有时候能输出正确答案 `500500`，有时候却不能。

这是因为 `work1` 和 `work2` 是并发执行的，假设一开始，2 个线程同时计算 `value + i`，`work1` 和 `work2` 分别得到 `1` 和 `5001`，但是写入 `value` 变量是有先后顺序的。假设 `work1` 先写入，`work2` 后写入，那么对于这 2 次累加，`value` 的最终结果是 `5001` ，而不是 `5002` 。

从这个例子可以看出，线程与进程类似，同样需要同步 (Synchronization) ，对于临界资源，每次只允许一个线程访问。

## 互斥量 mutex

互斥量，也叫互斥锁。mutex, 即 Mutual exclusion , 意为相互排斥，主要用于实现线程同步中的写保护操作。

### pthread_mutex_init

初始化一个互斥量 `pthread_mutex_t mutex` .

函数原型：

```c
int pthread_mutex_init(pthread_mutex_t *mutex, const pthread_mutexattr_t *attr);
```

参数：

+ `mutex` 是即将要被初始化的互斥量
+ `attr` 是互斥量的属性，与 `pthread_attr_t` 类似

与之类似的，还有 `pthread_mutex_destroy` 函数。

使用方法：

```c
pthread_mutex_t mutex;
pthread_mutex_init(&mutex, NULL);
pthread_mutex_destroy(&mutex);
```

### pthread_mutex_lock

阻塞调用。如果这个互斥锁此时正在被其它线程占用， 那么 `pthread_mutex_lock()` 调用会进入到这个互斥锁量的等待队列中，并会进入阻塞状态， 直到拿到该锁之后才会返回。

函数原型如下：

```c
int pthread_mutex_lock(pthread_mutex_t *mutex);
```

### pthread_mutex_trylock

非阻塞调用。当请求的锁正在被占用的时候， 不会进入阻塞状态，而是立刻返回，并返回一个错误代码 EBUSY，意思是说， 有其它线程正在使用这个锁。

直白点的说法，请求资源，能拿到就拿，拿不到我就继续往下执行。

函数原型：

```c
int pthread_mutex_trylock(pthread_mutex_t *mutex);
```

### pthread_mutex_timedlock

函数原型：

```c
int pthread_mutex_timedlock(pthread_mutex_t *restrict mutex, const struct timespec *restrict abs_timeout);
```

作用：尝试申请 `mutex` ，如果申请失败，那么会阻塞当前线程，直至时间到达 `abs_timeout` 这一时刻。

例子：

```c
#include <pthread.h>
#include <stdio.h>
#include <errno.h>
#include <string.h>
#include <unistd.h>
#include <sys/time.h>
int main(void)
{
    int err;
    struct timespec tout;
    struct tm *tmp;
    char buf[64];
    pthread_mutex_t lock = PTHREAD_MUTEX_INITIALIZER;
    pthread_mutex_lock(&lock);
    printf("mutex is locked\n");

    clock_gettime(CLOCK_REALTIME, &tout);
    tmp = localtime(&tout.tv_sec);
    strftime(buf, sizeof(buf), "%r", tmp);
    printf("current time is %s\n", buf);

    tout.tv_sec += 10; /* 10 seconds from now */
    /* caution: this could lead to deadlock */
    err = pthread_mutex_timedlock(&lock, &tout);
    clock_gettime(CLOCK_REALTIME, &tout);
    tmp = localtime(&tout.tv_sec);
    strftime(buf, sizeof(buf), "%r", tmp);
    printf("the time is now %s\n", buf);
    
    if (err == 0)
        printf("mutex locked again!\n");
    else
        printf("can’t lock mutex again: %s\n", strerror(err));
    exit(0);
}
```



### pthread_mutex_unlock

释放互斥锁。

函数原型如下：

```c
int pthread_mutex_unlock(pthread_mutex_t *mutex);
```

对于这些 API，如果成功，那么返回 0，否则返回错误码 `errno` ，可以通过下列宏定义打印错误信息：

```cpp
#define handler_error_en(en, msg) \
    do                            \
    {                             \
        errno = en;               \
        perror(msg);              \
        exit(EXIT_FAILURE);       \
    } while (0)
```



## 例子：同步累加

对于「线程同步」一节给出的例子，使用互斥量实现同步操作，使得程序能够正确完成累加操作。

```c
#include <stdlib.h>
#include <pthread.h>
#include <stdio.h>
#include <string.h>
const int N = 1e4;
int value = 0;
pthread_mutex_t mutex;
void *worker1(void *arg)
{
    int i = 1;
    int sum = 0;
    for (; i <= N / 2; i++) sum += i;
    // 这样保证 value 仅能由一个线程访问
    pthread_mutex_lock(&mutex);
    value += sum;
    pthread_mutex_unlock(&mutex);
    return NULL;
};
void *worker2(void *arg)
{
    int i = N / 2 + 1;
    int sum = 0;
    for (; i <= N; i++) sum += i;
    pthread_mutex_lock(&mutex);
    value += sum;
    pthread_mutex_unlock(&mutex);
    return NULL;
}
int main()
{
    pthread_t tid1, tid2;
    pthread_mutex_init(&mutex, NULL);
    pthread_create(&tid1, NULL, worker1, NULL);
    pthread_create(&tid2, NULL, worker2, NULL);
    pthread_join(tid1, NULL);
    pthread_join(tid2, NULL);
    printf("%d\n", value);
    printf("SUM(1, %d) should be %d .\n", N, N * (N + 1) / 2);
    pthread_mutex_destroy(&mutex);
}
```

