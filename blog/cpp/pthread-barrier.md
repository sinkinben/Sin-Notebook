## pthread 屏障

屏障 (Barrier) 允许每个线程都处于等待状态，直到所有的合作线程都达到某一执行点，然后从该点继续执行。`pthread_join(tid)` 就是一种最简单的「屏障」行为，允许当前线程等待 `tid` 线程的完成。

Barrier 的范围更广，它允许任意数量的线程处于等待状态，直到所有线程完成工作，而线程不需要退出，所有线程到达 Barrier 后可以继续执行。

## API

相关数据结构：

+ `pthread_barrier_t` ：屏障的数据结构；
+ `pthread_barrierattr_t`：屏障属性的数据结构。

### init and destroy

函数原型：

```c
int pthread_barrier_destroy(pthread_barrier_t *barrier);
int pthread_barrier_init(pthread_barrier_t *restrict barrier,
                         const pthread_barrierattr_t *restrict attr, unsigned count);
```

作用：初始化/销毁 `barrier` .

`count` 参数表示在允许所有线程继续执行前，必须达到 barrier 的线程的数量。

### wait

函数原型：

```c
int pthread_barrier_wait(pthread_barrier_t *barrier);
```

返回值：若成功，返回 0 或者 `PTHREAD_BARRIER_SERIAL_THREAD(-1)`, 其中有且只有一个线程会返回 `PTHREAD_BARRIER_SERIAL_THREAD` ，其余返回 0 ，返回 `PTHREAD_BARRIER_SERIAL_THREAD` 的线程可以作为主线程，处理、合并其他线程的工作；失败则返回错误码。

作用：使得 barrier 的计数加一，如果不满足条件（即计数结果未达到初始化时设定的 `count` ），线程进入 sleep 休眠状态。如果当前线程调用 `wait` 后，满足了 barrier 条件，那么在该 barrier 上等待的所有线程都将被唤醒。



## Example

多线程排序，在主线程合并。

```c
#include <pthread.h>
#include <stdio.h>
#include <sys/time.h>
#include <string.h>
#include <limits.h>
#include <stdlib.h>
#include <stdint.h>
#include <assert.h>
#include <stdlib.h>

#define NTHR 8
#define N 8000000L
#define TNUM (N / NTHR)

int nums[N] = {0};
int snums[N] = {0};
pthread_barrier_t barrier;

int x = 0;
int cmp(const void *a, const void *b)
{
    int x = *(int *)a;
    int y = *(int *)b;
    return x - y;
}
void merge()
{
    uint64_t idx[NTHR];
    uint64_t i, minidx, sidx;
    int num;
    for (i = 0; i < NTHR; i++) idx[i] = i * TNUM;
    for (sidx = 0; sidx < N; sidx++)
    {
        num = 0x7fffffff;
        for (i = 0; i < NTHR; i++)
        {
            if ((idx[i] < (i + 1) * TNUM) && (nums[idx[i]] < num))
            {
                num = nums[idx[i]];
                minidx = i;
            }
        }
        snums[sidx] = nums[idx[minidx]];
        idx[minidx]++;
    }
}

void *worker(void *args)
{
    uint64_t idx = (u_int64_t)args;
    qsort(&nums[idx], TNUM, sizeof(int), cmp);
    pthread_barrier_wait(&barrier);
    return NULL;
}

int main()
{
    int i = 0, ret = 0;
    pthread_t tid;

    // init numbers array
    srandom(time(NULL));
    for (i = 0; i < N; i++)
        nums[i] = random() % 100;

    // init barrier
    pthread_barrier_init(&barrier, NULL, NTHR + 1);

    // create NTHR threads
    for (i = 0; i < NTHR; i++)
    {
        ret = pthread_create(&tid, NULL, worker, (void *)(i * TNUM));
        if (ret != 0)
        {
            puts(strerror(ret));
            exit(ret);
        }
    }
    // wait here
    pthread_barrier_wait(&barrier);
    merge();
    pthread_barrier_destroy(&barrier);

    // check whether if increase order
    for (i = 1; i < N; i++)
        assert(snums[i] >= snums[i - 1]);
    // print sorted result
    // for (i = 0; i < N; i++)
    //     printf("%d%c", snums[i], i == N - 1 ? '\n' : ' ');
    return 0;
}
```

