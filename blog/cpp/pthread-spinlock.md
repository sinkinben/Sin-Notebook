## pthread 自旋锁

自旋锁 (Spin Lock) 与互斥量类似，但它并不会使得线程进入阻塞状态，而是在获得自旋锁之前使线程处于忙等状态 （aka, 自旋状态）。那么，自旋锁存在的意义是什么？

> A spin lock could be used in situations where locks are held for short periods of times and threads don’t want to incur the cost of being descheduled.

如果线程调度的开销比忙等的开销要大，那么显然让线程进入忙等状态更有助于提高并发度。

## API

相关数据结构：

+ `pthread_spinlock_t` : 自旋锁数据结构；

### init and destory

函数原型：

```c
int pthread_spin_destroy(pthread_spinlock_t *lock);
int pthread_spin_init(pthread_spinlock_t *lock, int pshared);
```

初始化/销毁自旋锁 `lock`，`pshared` 的取值及其描述如下：

- `PTHREAD_PROCESS_SHARED` : 进程间共享自旋锁（该锁应当分配在共享内存上）。
- `PTHREAD_PROCESS_PRIVATE` : 单个进程内共享。 

### lock/trylock/unlock

函数原型：

```c
int pthread_spin_lock(pthread_spinlock_t *lock);
int pthread_spin_trylock(pthread_spinlock_t *lock);
int pthread_spin_unlock(pthread_spinlock_t *lock);
```

作用：

- `lock` 申请自旋锁, 在获得锁之前保持自旋状态；
- `trylock` 如果申请自旋锁失败，立即返回 `EBUSY` 错误（表示 Device or resource busy），也就是说，`trylock` 并不能使线程自旋；
- `unlock` 释放自旋锁。

对已锁定的自旋锁再次调用 `lock` ，是一种未定义行为（可能返回 `EDEADLK` 错误）。对未锁定的自旋锁调用 `unlock` 与之同理。