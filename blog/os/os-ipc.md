## 进程通信机制

众所周知，IPC (Inter Process Communication) 机制包括有：

- 管道：包括匿名管道和命名管道，用于进程间单向传递数据。
- 消息队列：消息的链接表，存放在内核中。一个消息队列由一个标识符（即队列 ID）来标识。
- 信号量 (semaphore)：用于多进程同步，而不是传递数据。
- 共享内存：用于进程间双向传递数据。
- 套接字 (socket)：可参考《APUE》的第 16 章或者《UNP》的第一部分。
- 信号 (signal)：可参考 《APUE》的第 10 章。

本文主要介绍：管道、信号量、消息队列和共享内存 4 种。

## 管道

管道包括：

- 匿名管道 (PIPE): 用于父子进程、兄弟进程之间。
- 命名管道 (FIFO): 可作用于任意进程之间。Unix 中一切皆文件，所以 FIFO 管道也是一种特殊文件。可以通过 `mkfifo` 命令创建，库函数中也有 `mkfifo` 这一函数。

### 匿名管道

```c
#include <unistd.h>
int pipe(int pipefd[2]);
#define _GNU_SOURCE             /* See feature_test_macros(7) */
#include <fcntl.h>              /* Obtain O_* constant definitions */
#include <unistd.h>
int pipe2(int pipefd[2], int flags);
```

一般用于父子进程通信。调用 `pipe(fd)` 会得到一个匿名管道的输入端 `fd[1]` 和输出端 `fd[0]` 。

**例子**

实现 Shell 中的管道功能，即：`cat pipe.c | wc -l` 。

```c
#include <unistd.h>
#include <stdio.h>
#include <errno.h>
#include <string.h>
int main()
{
    // exec: lcmd | rcmd
    // e.g. cat pipe.c | wc -l
    char *lcmd[] = {"cat", "pipe.c", NULL};
    char *rcmd[] = {"head", "-n", "10", NULL};
    int fd[2];
    pipe(fd);
    pid_t pid;
    if ((pid = fork()) == 0)
    {
        dup2(fd[1], 1);
        close(fd[0]), close(fd[1]);
        execvp(lcmd[0], lcmd);
        // should not be here
        exit(-1);
    }
    else if (pid > 0)
    {
        waitpid(pid, NULL, 0);
        if ((pid = fork()) == 0)
        {
            dup2(fd[0], 0);
            close(fd[0]), close(fd[1]);
            execvp(rcmd[0], rcmd);
            // should not be here
            exit(-1);
        }
        else if (pid > 0)
        {
            close(fd[0]), close(fd[1]);
            waitpid(pid, NULL, 0);
        }
    }
}
```



### 命名管道

```c
#include <sys/types.h>
#include <sys/stat.h>
int mkfifo(const char *pathname, mode_t mode);
```

返回值：成功返回 0，出错返回 -1 并设置 `errno` .

命名管道实际上是一类特殊的文件，调用 `mkfifo` 后，内核会创建文件 `pathname` 。

**例子**

- `fifo-write.c` ：读取文件 `common.h` 的内容到缓冲区 `buf`，然后把 `buf` 写入管道。
- `fifo-read.c` ：读取管道的内容，输出到屏幕。

**common.h**

```c
#ifndef __COMMON_H
#define __COMMON_H
#include <stdio.h>
#include <stdlib.h>
#include <fcntl.h>
#include <sys/stat.h>
#include <time.h>
#include <unistd.h>
#include <string.h>
static inline void errexit(const char *msg) { perror(msg), exit(EXIT_FAILURE); }
#endif
```

**fifo-write.c**

```c
#include "common.h"
const char *fifopath = "./fifo";
const char *filepath = "./common.h";
int main()
{
    int pipefd, filefd, ret;
    char buf[BUFSIZ];
    pid_t pid = getpid();
    printf("pid = %d\n", pid);

    if (access(fifopath, F_OK) == -1)
    {
        if (mkfifo(fifopath, 0777) != 0)
            errexit("mkfifo");
    }
    if ((pipefd = open(fifopath, O_WRONLY)) < 0)  errexit("open pipe");
    if ((filefd = open(filepath, O_RDONLY)) < 0)  errexit("open file");

    int n = 0;
    do
    {
        memset(buf, 0, BUFSIZ);
        if ((n = read(filefd, buf, BUFSIZ)) <= 0)  break;
        write(pipefd, buf, n);
    } while (n > 0);
    close(pipefd), close(filefd);
}
```

**fifo-read.c**

```c
#include "common.h"
const char *fifopath = "./fifo";
int main()
{
    char buf[BUFSIZ] = {0};
    int pipefd = -1;
    pid_t pid = getpid();
    printf("pid = %d\n", pid);
    if ((pipefd = open(fifopath, O_RDONLY)) < 0)  errexit("open");
    int ret = 0;
    do
    {
        memset(buf, 0, BUFSIZ);
        ret = read(pipefd, buf, BUFSIZ);
        printf("%s", buf);
    } while (ret > 0);
    close(pipefd);
}
```

**运行结果**

编译：

```text
gcc fifo-read.c -o read
gcc fifo-write.c -o write
```

先运行 `write` 后运行 `read` .

运行结果：

<img src="https://gitee.com/sinkinben/pic-go/raw/master/img/20210317150337.png"  style="width:70%;" />

## POSIX 信号量

信号量同样分为：

- 命名信号量：常用于进程间的同步。
- 匿名信号量：可参考 [pthread 信号量](https://www.cnblogs.com/sinkinben/p/14087750.html) 。
  - 用于多线程同步时，信号量需要放在线程都能访问到的内存，在 `sem_init` 中参数 `pshared` 设置为 0 。
  - 用于多进程同步时，信号量需要放在共享内存（保证不同进程都能访问这个信号量），在 `sem_init` 中参数 `pshared` 设置为 1 。


无论是命名还是匿名的信号量，PV 操作都是一致的：

```C
// wait
int sem_wait(sem_t *sem);
int sem_trywait(sem_t *sem);
int sem_timedwait(sem_t *sem, const struct timespec *abs_timeout);
// post
int sem_post(sem_t *sem);
// get value
int sem_getvalue(sem_t *sem, int *sval);
```


匿名信号量相关 API ：

```c
// init
int sem_init(sem_t *sem, int pshared, unsigned int value);
// sem_destroy destroys the unnamed semaphore
int sem_destroy(sem_t *sem);
```

命名信号量相关 API ：
```c
// open, return SEM_FAILED if error occurs
sem_t *sem_open(const char *name, int oflag);
sem_t *sem_open(const char *name, int oflag, mode_t mode, unsigned int value);
// close
int sem_close(sem_t *sem);
// unlink
int sem_unlink(const char *name);
```

`sem_close` 用于关闭一个命名信号量 `sem` ，但并不会释放 `sem` ，因为 `sem` 是多进程共享的。

`sem_unlink` 用于将信号量的名字从内核中删除（但还不会正式销毁），当**打开该信号量的所有进程**都调用了 `sem_close(sem)` 之后，信号量才会正式销毁。

`sem_open` 用于创建一个命名信号量：

- `name` 是标识信号量的字符串；
- `oflag` 可以是下面 3 种情况之一：
  -  0 表示打开一个已存在的信号量；
  - `O_CREAT` 表示信号量不存在则创建；
  - `O_CREAT | O_EXCL` 表示如果信号量 `name` 存在则返回错误。

**例子：实现互斥原语**

使用值为 1 的信号量实现互斥量。

使用进程 ID 和计数器 `cnt` 去对信号量命名，但这里我们没有对 `cnt` 实现同步操作，为什么呢？因为当 2 个线程竞争时，同时使用同一个 `cnt` 值去创建信号量时，`sem_open` 会因为同一个 `name` 给失败线程返回 `SEM_FAILED` ，失败线程会进入下一次循环。

最后还调用了 `sem_unlink` ，这可以保证其他进程不能访问到该信号量，也就保证了这个 `slock` 结构只能在当前的进程中用于多线程同步。

```c
struct slock
{
    sem_t *semp;
    char name[BUFSIZ];
};
struct slock *s_alloc()
{
    struct slock *sp;
    static int cnt = 0;
    if ((sp = malloc(sizeof(struct slock))) == NULL)
        return NULL;
    do
    {
        snprintf(sp->name, sizeof(sp->name), "/%ld.%d", (long)getpid(), cnt++);
        // S_IRWXU means: Read, write, and execute by owner
        sp->semp = sem_open(sp->name, O_CREAT | O_EXCL, S_IRWXU, 1);
    } while ((sp->semp == SEM_FAILED) && (errno == EEXIST));
    if (sp->semp == SEM_FAILED)
    {
        free(sp);
        return NULL;
    }
    sem_unlink(sp->name);
    return sp;
}

void s_free(struct slock *sp)
{
    sem_close(sp->semp);
    free(sp);
}

int s_lock(struct slock *sp)    { return sem_wait(sp->semp); }
int s_trylock(struct slock *sp) { return sem_trywait(sp->semp); }
int s_unlock(struct slock *sp)  { return sem_post(sp->semp); }
```





## XSI IPC

POSIX (Portable Operating System Interface) 表示的是一系列的标准函数库，是为了提高 Unix 系统的可移植性而提出的。

XSI (UniX Open System Interface) 相当于 POSIX 的超集，XSI IPC 包括 3 种：信号量、共享内存和消息队列。



### 预备知识

对于 XSI 中的 3 种 IPC 结构，都是通过一个非负整数 ID 来标记的。在用户看来，每个 IPC 对象都与一个 key 相关联（数据类型是 `key_t`）。

IPC 对象的 key 可以通过 `ftok` 获取：

```c
#include <sys/types.h>
#include <sys/ipc.h>
key_t ftok(const char *pathname, int proj_id);
```

`man ftok` 的描述：

> The `ftok()` function uses the identity of the file named by the given `pathname` **(which must refer to an existing, accessible file)** and the least significant **8 bits** of `proj_id` **(which must be nonzero)** to generate a `key_t` type System V IPC key, suitable for use with `msgget(2)`, `semget(2)`, or `shmget(2)`.

每个 IPC 对象都有一个对应的权限结构体，记录了该 IPC 对象的权限信息：

```c
// #include <sys/ipc.h>
struct ipc_perm
{
    __key_t __key;           /* Key.  */
    __uid_t uid;             /* Owner's user ID.  */
    __gid_t gid;             /* Owner's group ID.  */
    __uid_t cuid;            /* Creator's user ID.  */
    __gid_t cgid;            /* Creator's group ID.  */
    unsigned short int mode; /* Read/write permission.  */
    unsigned short int __pad1;
    unsigned short int __seq; /* Sequence number.  */
    unsigned short int __pad2;
    __syscall_ulong_t __glibc_reserved1;
    __syscall_ulong_t __glibc_reserved2;
};
```

对于参数 `mode` 可以按照下列表格的进行权限初始化：

<img src="https://gitee.com/sinkinben/pic-go/raw/master/img/20210318160845.png" style="width:45%;" />

XSI IPC 存在的缺点：

- 没有引用计数。如果一个进程创建了消息队列，并放入了一些消息，然后进程终止，但该消息队列不会被删除，会一直存在，直到发生下面的事件：
  - 某个进程调用 `msgrcv` 或者 `msgctl` 去读或者删除消息。
  - 某个进程执行 `ipcrm` 命令删除消息队。
- 与管道 FIFO 相比，最后一个引用 FIFO 的进程终止时，虽然它的 `name` 还在内核中，但它的数据已经被释放，并且可以像普通文件那样删除一个 FIFO 管道。
- 不使用描述符，也就就是说，我们不能使用 `select, poll, epoll` 这些 I/O 复用技术去监听 IPC 事件。



### 信号量

除了 POSIX 的信号量，在 Unix 系统中，还有 XSI 信号量机制，相关 API 如下：

```c
#include <sys/sem.h>
#include <sys/ipc.h>
// 创建或获取一个信号量组：若成功返回信号量集ID，失败返回-1
int semget(key_t key, int num_sems, int sem_flags);
// 对信号量组进行操作，改变信号量的值：成功返回0，失败返回-1
int semop(int semid, struct sembuf semoparray[], size_t numops);
// 控制信号量的相关信息
int semctl(int semid, int sem_num, int cmd, ...);
```

还没来得及看 😅😅😅 。



### 共享内存

取决于不同的实现，内核会为每一个共享内存对象维护一个数据结构：

```c
struct shmid_ds
{
    struct ipc_perm shm_perm;     /* operation permission struct */
    size_t          shm_segsz;    /* size of segment in bytes */
    pid_t           shm_cpid;     /* pid of creator */
    pid_t           shm_lpid;     /* pid of last shmop() */
    shmatt_t        shm_nattch;   /* number of current attaches */
    time_t          shm_atime;    /* last-attach time, time of last shmat() */
    time_t          shm_dtime;    /* last-detach time */
    time_t          shm_ctime;    /* last-change time */
    // ... other attributions
};
```

`shmatt_t` 是一个无符号整数。

`shmget` 函数：

```c
#include <sys/ipc.h>
#include <sys/shm.h>
int shmget(key_t key, size_t size, int shmflg);
// 若成功，返回 Shared Memory ID，失败返回 -1 并设置 errno
```

初始化一个共享内存时，需要下列准备工作：
- `key` 是通过 `ftok` 生成的，它可以决定是创建一个 Shared Memory 还是引用一个已存在的 Shared Memory 。
- 对于 `shmid_ds` ：
  - `shm_lpid, shm_nattch, shm_atime, shm_dtime` 都置为 0 。
  - `shm_ctime` 设置为当前时间。
  - `shm_segsz` 设置为参数的 `size` 。

对于参数 `size` ，它指定共享内存的长度（以字节为单位），通常是页大小的整数倍（如果不是，向上取整）。如果是引用一个现有的共享内存，那么 `size = 0` 。

`shmflg` 是权限标志，它的作用与 `open` 函数的 `mode` 参数一样，如果要想在 `key` 标识的共享内存不存在时，创建它的话，可以与 `IPC_CREAT` 做或操作。

`shmctl` 函数：

```c
int shmctl(int shmid, int cmd, struct shmid_ds *buf);
// return 0 if success, -1 if failed
```

`cmd` 参数可以是下面 5 种：

- `IPC_STAT` : 读取 `shmid` 这个对象的 `shmid_ds` ，存储到 `buf` 当中。
- `IPC_SET` ：将 `buf` 中的 `uid, gid, mode` 参数设置到与 `shmid` 绑定的 `shmid_ds` 。
- `IPC_RMID` : 删除 `shmid` 这个共享内存对象。
- `SHM_LOCK` ：对共享内存加锁，只能由超级用户执行。
- `SHM_UNLOCK` ：对共享内存解锁，只能由超级用户执行。



`shmat, shmdt` 函数：

```c
#include <sys/types.h>
#include <sys/shm.h>
void *shmat(int shmid, const void *shmaddr, int shmflg);
int shmdt(const void *shmaddr);
```

`shmat` 用于将内核中的共享内存与当前进程空间建立映射。



### 消息队列







## References



