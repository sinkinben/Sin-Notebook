## pthread 信号量

本文介绍 `semaphore.h` 的相关 API ：

+ 与 Named Semaphore 相关：`sem_open, sem_close, sem_unlink` .
+ 与 Unnamed Semaphore 相关：`sem_init, sem_destroy ` .

## API

### sem_init

函数原型：

```c
int sem_init(sem_t *sem, int pshared, unsigned int value);
```

作用：

> `sem_init()`  initializes  the  unnamed  semaphore  at the address pointed to by sem.  The `value` argument specifies the initial value for the semaphore.
>
> The `pshared` argument indicates whether this semaphore is to be shared between the threads of a process, or between processes.
>
> If  `pshared`  has the value `0`, then the semaphore is shared **between the threads** of a process, and should be located at some address that is visible to all threads (e.g., a global variable, or a variable allocated dynamically on the heap).
>
> If  `pshared`  is  nonzero,  then  the  semaphore is shared **between processes**, and should be located in a region of shared memory (see `shm_open(3), mmap(2), and shmget(2)`).  (Since a child created  by  `fork(2)` inherits  its parent's memory mappings, it can also access the semaphore.)  Any process that can access the shared memory region can operate on the semaphore using `sem_post(3), sem_wait(3)`, and so on.
>
> -- Manual on Ubuntu.

初始化一个匿名信号量。

其中，`pshared` 为 0 表示该信号量在某个进程中的多个线程之间共享，该信号量应当能够被所有的线程访问，例如是一个全局变量或者是在堆上动态分配的变量；若不为 0 ，表示在进程间共享，那么该信号量应该位于共享内存 `shared memory` 中。由于 `fork` 操作产生的子进程，会自动继承父进程的 `memory mapping`，所以这些 `fork` 进程也能访问该信号量。

### sem_destroy

函数原型：

```c
int sem_destroy(sem_t *sem);
```

作用：

> `sem_destroy()` destroys the unnamed semaphore at the address pointed to by `sem`. Only a semaphore that has been initialized by `sem_init(3)` should be destroyed using `sem_destroy()`.
>
> Destroying  a  semaphore that other processes or threads are currently blocked on (in `sem_wait(3)`) produces undefined behavior.
>
> Using a semaphore that has been destroyed produces undefined results,  until  the  semaphore  has  been reinitialized using `sem_init(3)`.
>
> -- Manual on Ubuntu.

销毁一个已初始化的匿名信号量。

如果该信号量上还存在被阻塞的进程或线程，销毁该信号量将是一个 Undefined behavior .

使用一个已销毁的信号量同样是一个 Undefined behavior .

### sem_wait / trywait / timedwait

函数原型：

```c
int sem_wait(sem_t *sem);
int sem_trywait(sem_t *sem);
int sem_timedwait(sem_t *sem, const struct timespec *abs_timeout);
```

作用：

> `sem_wait()`  decrements  (locks)  the  semaphore pointed to by sem.  If the semaphore's value is greater than zero, then the decrement proceeds, and the function returns, immediately.  If the  semaphore  curently  has the value zero, then the call blocks until either it becomes possible to perform the decrement (i.e., the semaphore value rises above zero), or a signal handler interrupts the call.
>
> `sem_trywait()` is the same as `sem_wait()`, except that if the decrement cannot be immediately  performed, then call returns an error (errno set to `EAGAIN`) instead of blocking.
>
> `sem_timedwait()`  is  the same as `sem_wait()`, except that `abs_timeout` specifies a limit on the amount of time that the call should block if the decrement cannot  be  immediately  performed.   The  `abs_timeout` argument  points to a structure that specifies an absolute timeout in seconds and nanoseconds since the Epoch, 1970-01-01 00:00:00 +0000 (UTC).  This structure is defined as follows:
>
> ```c
> struct timespec {
>     time_t tv_sec;      /* Seconds */
>     long   tv_nsec;     /* Nanoseconds [0 .. 999999999] */
> };
> ```
>
> If the timeout has already expired by the time of the call, and the semaphore could not be locked immediately, then `sem_timedwait()` fails with a timeout error (errno set to `ETIMEDOUT`).
>
> If  the  operation can be performed immediately, then sem_timedwait() never fails with a timeout error, regardless of the value of `abs_timeout`.  Furthermore, the validity of `abs_timeout`  is  not  checked in this case.
>
> -- Manual on Ubuntu.

如果信号量的值 `sem_val` 大于 0 ，`sem_wait` 会对 `sem_val` 减 1 ，函数立即返回。如果 `sem_val` 为 0 ，`sem_wait` 将阻塞调用者（某个进程或线程），直到 `sem_val` 重新大于 0 或者调用者被 Linux 系统中的系统调用 `signal()` 函数中断 。

对于 `trywait` ，如果对 `sem_val` 的减 1 操作不能完成，`trywait` 不会阻塞调用者，而是返回一个 `error`，并设置 `errno` 为 `EAGAIN` .

对于 `timedwait` ，如果 `sem_val` 为 0，则阻塞等待，当阻塞时长超过 `abs_timeout` 返回失败 (`errno` 设置为 `ETIMEDOUT`) .

### sem_post 

函数原型：

```c
int sem_post(sem_t *sem);
```

作用：

> `sem_post()` increments (unlocks) the semaphore pointed to by `sem`.  If the semaphore's value consequently becomes **greater than zero**, then another process or thread blocked in a `sem_wait(3)` call will be woken up and proceed to lock the semaphore.

对信号量的值 `sem_val` 进行加 1 操作。如果重新大于 0 ，那么唤醒某个阻塞的线程或进程，该线程/进程将重新锁定该信号量。

### sem_getvalue

函数原型：

```c
int sem_getvalue(sem_t *sem, int *sval);
```

作用：

> `sem_getvalue()` places the current value of the semaphore pointed to sem into the integer pointed to by `sval`.

获取 `sem` 的值，并放到 `sval` 指向的内存。

## Examples

### Apple-Orange Problem

采用信号量机制，解决「苹果-橙子」问题：一个能放 N（这里 N 设为 3）个水果的盘子，爸爸只往盘子里放苹果，妈妈只放橙子，女儿只吃盘子里的橙子，儿子只吃苹果。

代码实现：

```c
#include <stdio.h>
#include <semaphore.h>
#include <pthread.h>
const int n = 3; // 果盘的容量
const int k = 3; // 每个人重复 k 次操作
sem_t apple, orange, empty;

void *father_worker(void *arg)
{
    int i = 0;
    for (i = 0; i < k; i++)
    {
        sem_wait(&empty);
        puts("[Father] apple++");
        sem_post(&apple);
    }
    return NULL;
}

void *mother_worker(void *arg)
{
    int i = 0;
    for (i = 0; i < k; i++)
    {
        sem_wait(&empty);
        puts("[Mother] orange++");
        sem_post(&orange);
    }
    return NULL;
}
void *son_worker(void *arg)
{
    int i = 0;
    for (i = 0; i < k; i++)
    {
        sem_wait(&apple);
        puts("\t[Son] apple--");
        sem_post(&empty);
    }
    return NULL;
}
void *daughter_worker(void *arg)
{
    int i = 0;
    for (i = 0; i < k; i++)
    {
        sem_wait(&orange);
        puts("\t[Daughter] orange--");
        sem_post(&empty);
    }
    return NULL;
}

int main()
{
    sem_init(&empty, 0, n);
    sem_init(&apple, 0, 0);
    sem_init(&orange, 0, 0);

    pthread_t father, mother, son, daughter;
    pthread_create(&father, NULL, father_worker, NULL);
    pthread_create(&mother, NULL, mother_worker, NULL);
    pthread_create(&son, NULL, son_worker, NULL);
    pthread_create(&daughter, NULL, daughter_worker, NULL);

    pthread_join(father, NULL), pthread_join(mother, NULL);
    pthread_join(son, NULL), pthread_join(daughter, NULL);

    sem_destroy(&empty), sem_destroy(&apple), sem_destroy(&orange);
    return 0;
}
```

### PC Problem

解决[上一篇文章](https://www.cnblogs.com/sinkinben/p/14087320.html) 提到的 PC 问题。

+ **全局变量**

```c
// global variables
#define CAPACITY 4 // buffer 的容量
#define N 8        // 依据题意，需要转换 8 个字符
sem_t mutex1, empty1, full1;
sem_t mutex2, empty2, full2;
buffer_t buf1, buf2;
```

`mutex1` 和 `mutex2` 的作用相当于 `pthead_mutex_t` ，是为了保证只有一个线程访问 `buf1` 或者 `buf2` 。

`empty1, empty2` 的信号量值将被初始化为 `buff` 的容量，表示可用的临界资源的数量（指 `buff` 中空闲位置的个数）。

`full1, full2` 的信号量的值将被初始化为 0，表示可用的临界资源的数量（指 `buff` 中可取走的 `item` 的数量）。

+ **buffer_t** 

```c
typedef struct
{
    char items[CAPACITY];
    int in, out;
} buffer_t;

void buffer_init(buffer_t *b) { b->in = b->out = 0; }
int buffer_is_full(buffer_t *b) { return ((b->in + 1) % CAPACITY) == (b->out); }
int buffer_is_empty(buffer_t *b) { return b->in == b->out; }
void buffer_put_item(buffer_t *buf, char item)
{
    buf->items[buf->in] = item;
    buf->in = (buf->in + 1) % CAPACITY;
}
char buffer_get_item(buffer_t *buf)
{
    char item = buf->items[buf->out];
    buf->out = (buf->out + 1) % CAPACITY;
    return item;
}
```

+ **producer**

```c
void *producer(void *arg)
{
    int i;
    char c;
    for (i = 0; i < N; i++)
    {

        sem_wait(&empty1);
        sem_wait(&mutex1);
        c = 'a' + i;
        buffer_put_item(&buf1, c);
        printf("[Producer] Put item [%c] into buf1.\n", c);
        sem_post(&mutex1);
        sem_post(&full1);
    }
    return NULL;
}
```

+ **consumer**

```c
void *consumer(void *arg)
{
    int i;
    char c;
    for (i = 0; i < N; i++)
    {

        sem_wait(&full2);
        sem_wait(&mutex2);
        c = buffer_get_item(&buf2);
        printf("\t[Consumer] Get item [%c] from buf2.\n", c);
        sem_post(&mutex2);
        sem_post(&empty2);
    }
    return NULL;
}
```

+ **calculator**

```c
void *calcultor(void *arg)
{
    int i;
    char c;
    for (i = 0; i < N; i++)
    {
        sem_wait(&full1);
        sem_wait(&mutex1);
        c = buffer_get_item(&buf1);
        sem_post(&mutex1);
        sem_post(&empty1);

        sem_wait(&empty2);
        sem_wait(&mutex2);
        buffer_put_item(&buf2, 'A' + c - 'a');
        sem_post(&mutex2);
        sem_post(&full2);
    }
}
```

+ **main函数**

```c
int main()
{
    buffer_init(&buf1), buffer_init(&buf2);
    sem_init(&mutex1, 0, 1), sem_init(&mutex2, 0, 1);
    sem_init(&empty1, 0, CAPACITY), sem_init(&empty2, 0, CAPACITY);
    sem_init(&full1, 0, 0), sem_init(&full2, 0, 0);

    pthread_t calc, prod, cons;
    pthread_create(&calc, NULL, calcultor, NULL);
    pthread_create(&prod, NULL, producer, NULL);
    pthread_create(&cons, NULL, consumer, NULL);

    pthread_join(prod, NULL), pthread_join(calc, NULL), pthread_join(cons, NULL);
    sem_destroy(&mutex1), sem_destroy(&mutex2);
    sem_destroy(&empty1), sem_destroy(&empty2);
    sem_destroy(&full1), sem_destroy(&full2);
    return 0;
}
```

