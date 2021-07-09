## pthread 读写锁

pthread 读写锁 (Read Write Lock, rwlock) 把对共享资源的访问者分为读者和写者，读者仅仅对共享资源进行读访问，写者仅仅对共享资源进行写操作。

如果使用互斥量 mutex，读者和写者都必须独占 mutex 以独占共享资源，在读写锁机制下，同一时刻允许有多个读者读访问共享资源，仅仅有写者才须要独占资源。相比互斥量，读写锁因为允许多个读者同一时候访问共享资源，进一步提高了多线程的并发度。

## API

相关数据结构：

+ `pthread_rwlock_t` : 读写锁的数据结构；
+ `pthread_rwlockattr_t` : 读写锁属性的数据结构。

### init and destroy

函数原型：

```c
int pthread_rwlock_destroy(pthread_rwlock_t *rwlock);
int pthread_rwlock_init(pthread_rwlock_t *restrict rwlock, const pthread_rwlockattr_t *restrict attr);
```

作用：初始化/销毁一个读写锁。

### rdlock/tryrdlock/timedrdlock

函数原型：

```c
int pthread_rwlock_rdlock(pthread_rwlock_t *rwlock);
int pthread_rwlock_tryrdlock(pthread_rwlock_t *rwlock);
int pthread_rwlock_timedrdlock(pthread_rwlock_t *restrict rwlock, const struct timespec *restrict abs_timeout);
```

作用：

+ `rdlock` 申请读锁 `rwlock` ，若申请失败则阻塞当前线程；
+ `tryrdlock` 如果申请失败会立即返回，而不会阻塞线程；
+ `timedrdlock` 申请读锁 `rwlock`，如果申请失败，会阻塞线程直到某一时刻，该时刻由 `abs_timeout` 指定。

### wrlock/trywrlock/timedwrlock

函数原型：

```c
int pthread_rwlock_trywrlock(pthread_rwlock_t *rwlock);
int pthread_rwlock_wrlock(pthread_rwlock_t *rwlock);
int pthread_rwlock_timedwrlock(pthread_rwlock_t *restrict rwlock, const struct timespec *restrict abs_timeout);
```

作用：与 `rdlock` 类似。



### unlock

函数原型：

```c
int pthread_rwlock_unlock(pthread_rwlock_t *rwlock);
```

作用：释放 `rwlock` .



## Example

参考 APUE 一书的例子。

在本节中，通过读写锁机制实现以下场景：存在一个任务队列，需要由多线程并发完成队列中的任务，但是任务的分配由主线程完成，主线程把该任务所属的线程 ID `j_id` 放在任务的数据结构中，只有该 `j_id` 线程才能操作这一任务。

通过读写锁机制实现上述任务队列 `queue` 的并发控制：允许多个线程读取队列，但只允许单个线程修改队列。

<img src="https://gitee.com/sinkinben/pic-go/raw/master/img/20210113133011.png" alt="image-20210113133011106" style="width:80%;" />

代码实现：

```c
#include <stdlib.h>
#include <pthread.h>
struct job
{
    struct job *j_next;
    struct job *j_prev;
    pthread_t j_id; /* tells which thread handles this job */
                    /* ... more stuff here ... */
};
struct queue
{
    struct job *q_head;
    struct job *q_tail;
    pthread_rwlock_t q_lock;
};

/*
 * Initialize a queue.
 */
int queue_init(struct queue *qp)
{
    int err;

    qp->q_head = NULL;
    qp->q_tail = NULL;
    err = pthread_rwlock_init(&qp->q_lock, NULL);
    if (err != 0)
        return (err);
    /* ... continue initialization ... */
    return (0);
}

/*
 * Insert a job at the head of the queue.
 */
void job_insert(struct queue *qp, struct job *jp)
{
    pthread_rwlock_wrlock(&qp->q_lock);
    jp->j_next = qp->q_head;
    jp->j_prev = NULL;
    if (qp->q_head != NULL)
        qp->q_head->j_prev = jp;
    else
        qp->q_tail = jp; /* list was empty */
    qp->q_head = jp;
    pthread_rwlock_unlock(&qp->q_lock);
}

/*
 * Append a job on the tail of the queue.
 */
void job_append(struct queue *qp, struct job *jp)
{
    pthread_rwlock_wrlock(&qp->q_lock);
    jp->j_next = NULL;
    jp->j_prev = qp->q_tail;
    if (qp->q_tail != NULL)
        qp->q_tail->j_next = jp;
    else
        qp->q_head = jp; /* list was empty */
    qp->q_tail = jp;
    pthread_rwlock_unlock(&qp->q_lock);
}

/*
 * Remove the given job from a queue.
 */
void job_remove(struct queue *qp, struct job *jp)
{
    pthread_rwlock_wrlock(&qp->q_lock);
    if (jp == qp->q_head)
    {
        qp->q_head = jp->j_next;
        if (qp->q_tail == jp)
            qp->q_tail = NULL;
        else
            jp->j_next->j_prev = jp->j_prev;
    }
    else if (jp == qp->q_tail)
    {
        qp->q_tail = jp->j_prev;
        jp->j_prev->j_next = jp->j_next;
    }
    else
    {
        jp->j_prev->j_next = jp->j_next;
        jp->j_next->j_prev = jp->j_prev;
    }
    pthread_rwlock_unlock(&qp->q_lock);
}

/*
 * Find a job for the given thread ID.
 */
struct job *job_find(struct queue *qp, pthread_t id)
{
    struct job *jp;

    if (pthread_rwlock_rdlock(&qp->q_lock) != 0)
        return (NULL);

    for (jp = qp->q_head; jp != NULL; jp = jp->j_next)
        if (pthread_equal(jp->j_id, id))
            break;

    pthread_rwlock_unlock(&qp->q_lock);
    return (jp);
}
```

