## pthread 条件变量

在上一篇博客[互斥量](https://www.cnblogs.com/sinkinben/p/13987031.html)中，解决了线程如何互斥访问临界资源的问题。

在开始本文之前，我们先保留一个问题：为什么需要条件变量，如果只有互斥量不能解决什么问题？

## API

### init/destroy

条件变量的数据类型是 `pthread_cond_t` .

初始化，销毁 API 为：

```c
int pthread_cond_init(pthread_cond_t *cond, const pthread_condattr_t *attr);
int pthread_cond_destroy(pthread_cond_t *cond);
```

### pthread_cond_wait

函数原型：

```c
int pthread_cond_wait(pthread_cond_t *cond, pthread_mutex_t *mutex);
```

作用：

> The `pthread_cond_wait` function atomically blocks the current thread waiting on the condition variable specified by `cond`, and releases the mutex specified by `mutex`. The waiting thread unblocks only after another thread calls `pthread_cond_signal`, or `pthread_cond_broadcast` with the same condition variable, and the current thread re-acquires the lock on mutex.
>
> —— Manual on MacOS.

在条件变量 `cond` 上阻塞线程，加入 `cond` 的等待队列，并释放互斥量 `mutex` . 如果其他线程使用同一个条件变量 `cond` 调用了 `pthread_cond_signal/broadcast` ，唤醒的线程会重新获得互斥锁 `mutex` .

### pthread_cond_timedwait

函数原型：

```c
int pthread_cond_timedwait(pthread_cond_t *cond, pthread_mutex_t *mutex, const struct timespec *abstime);
```

作用：与 `pthread_cond_wait` 类似，但该线程被唤醒的条件是其他线程调用了 `signal/broad` ，或者系统时间到达了 `abstime` 。

### pthread_cond_signal

函数原型：

```c
int pthread_cond_signal(pthread_cond_t *cond)
```

作用：

> The `pthread_cond_signal` function shall unblock at least one of the threads that are blocked no the specified condition variable `cond` (if any threads are blocked on `cond`).
>
> If more than one thread is blocked on a condition variable, the scheduling policy shall determine the order which threads are unblocked. 
>
> When each thread unblocked as a result of a `pthread_cond_broadcast()` or `pthread_cond_signal()`  returns  from its call to `pthread_cond_wait()` or `pthread_cond_timedwait()`, the thread shall own the `mutex` with which  it called  `pthread_cond_wait()` or `pthread_cond_timedwait()`.
>
> The thread(s) that are unblocked shall contend for the `mutex` according to the  scheduling   policy   (if applicable),   and   as   if   each  had  called `pthread_mutex_lock()`.
>
> The `pthread_cond_broadcast()` and `pthread_cond_signal()` functions  shall have no effect if there are no threads currently blocked on `cond`.
>
> ——Manual on Ubuntu.

唤醒一个在 `cond` 上等待的至少一个线程，如果 `cond` 上阻塞了多个线程，那么将根据调度策略选取一个。

当被唤醒的线程从 `wait/timedwait` 函数返回，将重新获得 `mutex` （但可能需要竞争，因为可能唤醒多个线程）。

### pthread_cond_broadcast

函数原型：

```c
int pthread_cond_broadcast(pthread_cond_t *cond);
```

作用：唤醒所有在 `cond` 上等待的线程。

## 生产者消费者问题

又称 PC (Producer - Consumer) 问题。

详细问题定义可以看：

+ [1] 百度百科：[生产者消费者问题](https://baike.baidu.com/item/生产者消费者问题/8412057) .
+ [2] 维基百科：[Producer-Consumer Problem](https://en.wikipedia.org/wiki/Producer–consumer_problem) .

具体要求：

+ 系统中有3个线程：生产者、计算者、消费者
+ 系统中有2个容量为4的缓冲区：buffer1、buffer2
+ 生产者生产'a'、'b'、'c'、‘d'、'e'、'f'、'g'、'h'八个字符，放入到buffer1; 计算者从buffer1取出字符，将小写字符转换为大写字符，放入到buffer2
+ 消费者从buffer2取出字符，将其打印到屏幕上

### buffer 的定义

`buffer` 实质上是一个队列。

```c
const int CAPACITY = 4;
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

### 一些全局变量

```c
const int CAPACITY = 4;  // buffer 的容量
const int N = 8;         // 依据题意，需要转换 8 个字符
buffer_t buf1, buf2;
pthread_mutex_t mutex1 = PTHREAD_MUTEX_INITIALIZER;  // 保证只有一个线程访问 buf1
pthread_mutex_t mutex2 = PTHREAD_MUTEX_INITIALIZER;  // 保证只有一个线程访问 buf2
pthread_cond_t empty1 = PTHREAD_COND_INITIALIZER;
pthread_cond_t empty2 = PTHREAD_COND_INITIALIZER;
pthread_cond_t full1 = PTHREAD_COND_INITIALIZER;
pthread_cond_t full2 = PTHREAD_COND_INITIALIZER;
```

几个条件变量的作用如下：

+ `empty1` 表示当 `buf1` 为空的时候，从 `buf1` 取数据的线程要在此条件变量上等待。
+ `full1` 表示当 `buf1` 为满的时候，向 `buf1` 写数据的线程要在此条件变量上等待。

其他同理。

### producer

代码思路解析：

+ 因为要对 `buf1` 操作，首先写一对 `pthread_mutex_lock/unlock` ，保证临界代码区内只有 `producer` 操作 `buf1` ；
+ 如果 `buf1` 是满的，那么就将 `producer` 线程阻塞在条件变量 `full1` 上，释放互斥量 `mutex1` （虽然不能写入，但要让别的线程能够读取 `buf1` 的数据）；
+ 进入临界区，把数据写入 `buf1` ；
+ 离开临界区，因为写入了一次数据，`buf1` 必定不为空，因此唤醒一个在 `empty1` 上等待的线程，最后释放 `mutex1` .

```c
void *producer(void *arg)
{
    int i = 0;
    // can be while(true)
    for (; i < N; i++)
    {
        pthread_mutex_lock(&mutex1);
        while (buffer_is_full(&buf1))
            pthread_cond_wait(&full1, &mutex1);
        buffer_put_item(&buf1, (char)('a' + i));
        printf("Producer put [%c] in buffer1. \n", (char)('a' + i));
        pthread_cond_signal(&empty1);
        pthread_mutex_unlock(&mutex1);
    }
    return NULL;
}
```

### consumer

思路与 `producer` 类似。

```c
void *consumer(void *arg)
{
    int i = 0;
    for (; i < N; i++)
    {
        pthread_mutex_lock(&mutex2);
        while (buffer_is_empty(&buf2))
            pthread_cond_wait(&empty2, &mutex2);
        char item = buffer_get_item(&buf2);
        printf("\tConsumer get [%c] from buffer2. \n", item);
        pthread_cond_signal(&full2);
        pthread_mutex_unlock(&mutex2);
    }
    return NULL;
}
```

### calculator

这是 `produer` 和 `consumer` 的结合体。

```c
void *calculator(void *arg)
{
    int i = 0;
    char item;
    for (; i < N; i++)
    {
        pthread_mutex_lock(&mutex1);
        while (buffer_is_empty(&buf1))
            pthread_cond_wait(&empty1, &mutex1);
        item = buffer_get_item(&buf1);
        pthread_cond_signal(&full1);
        pthread_mutex_unlock(&mutex1);

        pthread_mutex_lock(&mutex2);
        while (buffer_is_full(&buf2))
            pthread_cond_wait(&full2, &mutex2);
        buffer_put_item(&buf2, item - 'a' + 'A');
        pthread_cond_signal(&empty2);
        pthread_mutex_unlock(&mutex2);
    }
    return NULL;
}
```

### main 函数

```c
int main()
{
    pthread_t calc, prod, cons;
    // init buffer
    buffer_init(&buf1), buffer_init(&buf2);
	// create threads
    pthread_create(&calc, NULL, calculator, NULL);
    pthread_create(&prod, NULL, producer, NULL);
    pthread_create(&cons, NULL, consumer, NULL);
    pthread_join(calc, NULL);
    pthread_join(prod, NULL);
    pthread_join(cons, NULL);
    // destroy mutex
    pthread_mutex_destroy(&mutex1), pthread_mutex_destroy(&mutex2);
    // destroy cond
    pthread_cond_destroy(&empty1), pthread_cond_destroy(&empty2);
    pthread_cond_destroy(&full1), pthread_cond_destroy(&full2);
}
```

## 为什么需要条件变量

从上面的 Producer - Consumer 问题可以看出，`mutex` 仅仅能表达「线程能否获得访问临界资源的权限」这一层面的信息，而不能表达「临界资源是否足够」这个问题。

假设没有条件变量，`producer` 线程获得了 `buf1` 的访问权限（ `buf1` 的空闲位置对于 `producer` 来说是一种资源），但如果 `buf1` 是满的，`producer` 就没法对 `buf1` 操作。

对于 `producer` 来说，它不能占用访问 `buf1` 的互斥锁，但却又什么都不做。因此，它只能释放互斥锁 `mutex` ，让别的线程能够访问 `buf1` ，并取走数据，等到 `buf1` 有空闲位置，`producer` 再对 `buf1` 写数据。用伪代码表述如下：

```c
pthread_mutex_lock(&mutex1);
if (buffer_is_full(&buf1))
{
    pthread_mutex_unlock(&mutex1);
    wait_until_not_full(&buf1);
    pthread_mutex_lock(&mutex);
}
buffer_put_item(&buf1, item);
pthread_mutex_unlock(&mutex1);
```

而条件变量实际上是对上述一系列操作的一种封装。

## 为什么是 while

在上面代码中，使用 `pthread_cond_wait` 的时候，我们是通过这样的方式的：

```cpp
while (...)
    pthread_cond_wait(&cond, &mutex);
```

但这里为什么不是 `if` 而是 `while` 呢？

参考文章：https://www.cnblogs.com/leijiangtao/p/4028338.html 

### 解释 1 

```c
#include <pthread.h>
struct msg {
  struct msg *m_next;
  /* value...*/
};
 
struct msg* workq;
pthread_cond_t qready = PTHREAD_COND_INITIALIZER;
pthread_mutex_t qlock = PTHREAD_MUTEX_INITIALIZER;
 
void
process_msg() {
  struct msg* mp;
  for (;;) {
    pthread_mutex_lock(&qlock);
    while (workq == NULL) {
      pthread_cond_wait(&qread, &qlock);
    }
    mq = workq;
    workq = mp->m_next;
    pthread_mutex_unlock(&qlock);
    /* now process the message mp */
  }
}
 
void
enqueue_msg(struct msg* mp) {
    pthread_mutex_lock(&qlock);
    mp->m_next = workq;
    workq = mp;
    pthread_mutex_unlock(&qlock);
    /** 此时第三个线程在signal之前，执行了process_msg,刚好把mp元素拿走*/
    pthread_cond_signal(&qready);
    /** 此时执行signal, 在pthread_cond_wait等待的线程被唤醒，
        但是mp元素已经被另外一个线程拿走，所以，workq还是NULL ,因此需要继续等待*/
}
```

代码解析：

> 这里 `process_msg` 相当于消费者，`enqueue_msg` 相当于生产者，`struct msg* workq` 作为缓冲队列。
>
> 在 `process_msg` 中使用 `while (workq == NULL)` 循环判断条件，这里主要是因为在 `enqueue_msg` 中 `unlock` 之后才唤醒等待的线程，会出现上述注释出现的情况，造成 `workq==NULL`，因此需要继续等待。
>
> 但是如果将 `pthread_cond_signal` 移到 `pthread_mutex_unlock()` 之前执行，则会避免这种竞争，在 `unlock` 之后，会首先唤醒 `pthread_cond_wait` 的线程，进而 `workq != NULL` 总是成立。
>
> 因此建议使用 `while` 循环进行验证，以便能够容忍这种竞争。

### 解释 2 

`pthread_cond_signal` 在多核处理器上可能同时唤醒多个线程。

```c
//thread 1
while(0<x<10)
    pthread_cond_wait(...);

//thread 2
while(5<x<15)
    pthread_cond_wait(...);
```

如果某段时间内 `x == 8`，那么两个线程相继进入等待。

然后第三个线程，进行了如下操作：

```c
x = 12
pthread_cond_signal(...) 
// or call pthread_cond_broadcast()
```

那么可能线程 1、2 都被唤醒了（因为 `signal` 可能唤醒多个），但是，此时线程 1 仍然不满足 `while`，需要再次判断，然后进入下一次等待。

其次，即使 `signal` 只唤醒一个，上面我们提到，如果有多个线程都阻塞在同一个 `cond` 上，`signal` 会根据调度策略选取一个唤醒，那如果根据调度策略，唤醒的是线程 1 ，显然它还需要再一次判断是否需要继续等待（否则就违背了 `pthead_cond_wait` 的本意）。