## epoll

本文介绍 I/O 复用的重要知识点 `epoll` ，与之相关的还有 `select, pselect, poll` ，参考 [这篇文章](https://www.cnblogs.com/sinkinben/p/14491900.html) 。

首先来看一段 `man` 手册的介绍：

> The `epoll` API performs a similar task to `poll(2)`: monitoring multiple file descriptors to see if I/O is possible on any of them.  The `epoll` API can be used either as an edge-triggered or a level-triggered interface and scales well to large numbers of watched  file  descriptors.

翻译一下：

- `epoll` 的作用与 `poll` 类似，用于监控多个 I/O 描述符
- `epoll` 有 2 种模式：边缘触发 (Edge-Triggered, ET) 和水平触发 (Level-Triggered, LT)
- 可以应对大量描述符的场景

epoll API 是 Linux 内核 2.6 之后才引入的，目前也仅有 Linux 支持 epoll .



## API

与 `epoll` 相关的 API 主要有：create 函数，ctl 函数，wait 函数。



### epoll_create

```c
#include <sys/epoll.h>
int epoll_create(int size);      
int epoll_create1(int flags);  
// return epoll-fd if success, -1 if failed (and set errno)
```

我们把 `epoll` 看作是一个监控多个 I/O 描述符的数据结构，在下面的描述中， `epoll` 描述符， `epoll` 对象，epoll instance 是同一个意思。

`epoll_create` 返回一个 epoll 描述符（可以理解为该描述符指向一个 epoll 对象）。

对于参数 `size` ，在 Linux 2.6.8 之后，只要是任意的正数即可。在之前的版本中，`size` 是为了告诉内核，需要管理 `size` 个 I/O 描述符（但实际上，`size` 不是描述符个数的上限，如果描述符个数超过 `size` ，内核还是会自动申请更多的空间，因为 epoll 使用了红黑树去管理描述符）。因此，为了我们现在写的代码能够兼容旧版本内核，`size` 只需要使用任意正数即可。

与 `select` 不同，`select` 的第一个参数是最大描述符 + 1. 

对于 `epoll_create1` 的参数 `flags`：

- 如果 `flags = 0`，那么 `epoll_create1(0)` 等价于 `epoll_create(size)` .
- 其他情况：目前 `flags` 仅支持 `EPOLL_CLOEXEC` 一种值，与 `open` 函数的 `O_CLOEXEC` 类似，作用是：在 `fork` 出来的子进程中，如果执行 `exec` 系列函数，那么就关闭这个描述符（可通过 `man 2 open` 查看）。

注意：`epoll` 对象是会占用一个描述符的，可以在 `/proc/pid/fd` 中看到，因此不再使用的时候，需要调用 `close(epollfd)` 将其关闭。



### epoll_ctl

```c
int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event);
```

作用：用于管理一个 epoll 对象。

返回值：成功返回 0 ，失败 -1，并设置 `errno` .

参数解析：

- `epfd` 是 `epoll_create` 返回的 epoll 描述符；`fd` 是某个 I/O 描述符；`event` 是代表监听事件。
- `op` 可以是下列三种取值：
  - `EPOLL_CTL_ADD` : 向 `epfd` 添加一个需要被监听的 I/O 描述符 `fd `，并监听发生在这个 `fd` 上的 I/O 事件 `event` 。
    - 如果重复 ADD 同一个 `epfd` 两次会怎么样呢？可以参考 [man epoll](https://man7.org/linux/man-pages/man7/epoll.7.html) 的 Q&A 部分。
  - `EPOLL_CTL_MOD` : 把 `fd` 的监听事件改变为 `event` 。
  - `EPOLL_CTL_DEL` : 从 `epfd` 中删除 `fd` ，在 Linux 2.6.9 之后， `event` 此时可以为空，但在这之前，`event` 需要非空（但不起作用）。

`epoll_event` 的定义如下：

```c
typedef union epoll_data {
    void        *ptr;
    int          fd;
    uint32_t     u32;
    uint64_t     u64;
} epoll_data_t;

struct epoll_event {
    uint32_t     events;      /* Epoll events */
    epoll_data_t data;        /* User data variable */
};
```

其中，`data` 字段当 `epoll_wait` 返回时，存放已就绪的描述符。

`events` 字段是一系列比特位的组合，下面列举几个，更多详细的内容可以通过 `man epoll_ctl` 查看。

|   Mask Bit   |                 Description                  |
| :----------: | :------------------------------------------: |
|   EPOLLIN    |                  描述符可读                  |
|   EPOLLOUT   |                  描述符可写                  |
|   EPOLLPRI   |          有所谓的 urgent data 可读           |
| EPOLLONESHOT | 只监听一次事件（如果还需要监听，则再次添加） |
| **EPOLLET**  |    将 epoll 设置为 ET 模式（下面会讲解）     |
|   EPOLLERR   |              文件描述符发生错误              |





### epoll_wait

```c
int epoll_wait(int epfd, struct epoll_event *events, int maxevents, int timeout);
int epoll_pwait(int epfd, struct epoll_event *events, int maxevents, int timeout, const sigset_t *sigmask);
```

作用：等待 `epfd` 上的 I/O 事件，最多返回 `maxevents` 个事件。

返回值 `ret` :

- `ret > 0` : 表示就绪的描述符的个数；
- `ret = 0` : 在阻塞的 `timeout` 时间内，没有就绪的描述符；
- `ret = -1` : 错误，并设置 `errno` 。

当 `poll_wait` 返回时，`events[i].data` 包含了调用 `epoll_ctl` 时的配置信息， `events[i].data.fd` 存放是就绪的描述符，`events[i].events` 是待处理事件集合。

参数解析：

- `timeout` 指定 `epoll_wait` 阻塞的时长，单位是 ms 。
  - `timeout = 0`: 即使没有就绪事件发生，也立即返回。
  - `timeout = -1` : 一直阻塞，直到有就绪事件发生。
- `events` 是一个数组，用于存放已就绪的描述符和它的就绪事件，`maxevents` 指定返回的最大事件数，一般与数组长度相等。

`sigmask` 是一个信号集合（aka，信号屏蔽字），`epoll_wait, epoll_pwait` 的区别与 `select, pselect` 类似，可参考[这篇 blog](https://www.cnblogs.com/sinkinben/p/14491900.html) .

`epoll_pwait` 等价于：

```c
sigset_t origmask;
pthread_sigmask(SIG_SETMASK, &sigmask, &origmask);
ready = epoll_wait(epfd, &events, maxevents, timeout);
pthread_sigmask(SIG_SETMASK, &origmask, NULL);
```



## ET 和 LT

边缘触发 (Edge-Triggered, ET) 和水平触发 (Level-Triggered, LT) 是 epoll 的 2 种工作模式。

1. **LT 模式**

当 `epoll_wait()` 检测到描述符事件到达时，将此事件通知进程，进程可以不立即处理该事件，下次调用 `epoll_wait()` 会再次通知进程。是默认的一种模式，并且同时支持 Blocking 和 No-Blocking 的 socket。

2. **ET 模式**

和 LT 模式不同的是，通知之后进程必须立即处理事件，下次再调用 `epoll_wait()` 时不会再得到事件到达的通知。

ET 模式很大程度上减少了 epoll 事件被重复触发的次数，因此效率要比 LT 模式高。只支持 No-Blocking 的 socket，以避免由于一个 I/O 描述符的阻塞读/阻塞写操作把处理多个文件描述符的任务饿死。

**区别**：LT 事件不会丢弃，而是只要读 buffer 里面有数据可以让用户读取，则不断的通知，而 ET 则只在事件发生之时通知一次。

> **关于阻塞和非阻塞的 socket** 
>
> 阻塞与非阻塞就是 2 种典型的 I/O 模型，那么在 socket 编程上是怎么体现的呢？
>
> 像我这种长期在「新手村」写代码的人，平时用到的 socket 肯定都是阻塞的。简单来说，像常见的 socket 函数 `connect, accept, read, recv` ，调用之后必须要完成任务才返回的。相反非阻塞的 socket ，允许任务没完成直接返回，但在必要时需要设置 `errno` 告知用户发生了什么事情。
>
> 可以通过 `fcntl` 函数改变 socket 的文件表示，设置为非阻塞/阻塞：
>
> ```c
> fcntl(sockfd, F_SETFL, fcntl(sockfd, F_GETFL, 0) & ~O_NONBLOCK);  // blocking
> fcntl(sockfd, F_SETFL, fcntl(sockfd, F_GETFL, 0) | O_NONBLOCK);   // non-blocking
> ```
>
> 关于 `fcntl` 函数的更详细介绍，可以通过 `man` 浏览，或者查阅 APUE 的第 3 章。
>
> 阻塞与非阻塞 socket 的更多细节上区别可以参考 [这篇文章](https://blog.csdn.net/mayue_web/article/details/82873115) 。 
>
> TCP 非阻塞服务器的[例子](http://www.cs.tau.ac.il/~eddiea/samples/Non-Blocking/tcp-nonblocking-server.c.html)。



## 例子1：新手村教程

头文件使用：

```c
#include <sys/select.h>
#include <aio.h>
#include <unistd.h>
#include <stdlib.h>
#include <signal.h>
#include <arpa/inet.h>
#include <sys/socket.h>
#include <stdio.h>
#include <sys/epoll.h>
#include <errno.h>
#include <string.h>
```

### ET 模式

先看第一版代码，介绍几个 API 的使用。

```c
#define NR_EVENTS 16
int main()
{
    int nfds, i;
    int epfd = epoll_create1(0);
    struct epoll_event ev;
    struct epoll_event events[NR_EVENTS];
    ev.data.fd = STDIN_FILENO;
    ev.events = EPOLLIN | EPOLLET;
    epoll_ctl(epfd, EPOLL_CTL_ADD, STDIN_FILENO, &ev);
    while (1)
    {
        nfds = epoll_wait(epfd, events, NR_EVENTS, -1);
        for (i = 0; i < nfds; i++)
        {
            if (events[i].data.fd == STDIN_FILENO)
                printf("Hello, epoll!\n");
        }
    }
}
```

运行结果：

```text
$ gcc test.c; ./a.out
1
Hello, epoll!
2
Hello, epoll!
3
Hello, epoll!
<Ctrl+D>
Hello, epoll!
<Ctrl+D>
Hello, epoll!
^C
```

随便输入一些内容，回车，都会输出一个 `Hello, epoll!` .



### LT 模式

如果我们把 `ev.events` 改为：

```C
ev.events = EPOLLIN; // 默认为 LT 模式
```

那么运行结果为：

```
$ gcc test.c; ./a.out
1
Hello, epoll!
Hello, epoll!
...
```

会不断输出 `Hello, epoll!`，为什么会这样呢？因为输入缓冲区的数据没有被取走，默认的 LT 模式只要 I/O 描述符上数据可读，就会不断地通知进程。

那么，我们就用 `read` 把 `STDIN` 上的数据取走：

```c
#define NR_EVENTS 16
#define BUFSIZE 1024
int main()
{
    char buf[BUFSIZE] = {0};
    int nfds, i;
    int epfd = epoll_create1(0);
    struct epoll_event ev;
    struct epoll_event events[NR_EVENTS];
    ev.data.fd = STDIN_FILENO;
    ev.events = EPOLLIN;
    epoll_ctl(epfd, EPOLL_CTL_ADD, STDIN_FILENO, &ev);
    while (1)
    {
        nfds = epoll_wait(epfd, events, NR_EVENTS, -1);
        for (i = 0; i < nfds; i++)
        {
            if (events[i].data.fd == STDIN_FILENO)
            {
                read(STDIN_FILENO, buf, BUFSIZE);
                printf("%s", buf);
                bzero(buf, BUFSIZE);
            }
        }
    }
}
```

运行结果：

```text
$ gcc test.c; ./a.out
helo, sinkinben
helo, sinkinben
1
1
2
2
hello
hello
^C
```



## 例子2：TCP 服务器模型

摘抄自 `man epoll` 手册，该例子很好地说明了使用 epoll 编程时，服务器端的编程范式。

```c
int setnonblocking(int sockfd)
{
    fcntl(sockfd, F_SETFL, fcntl(sockfd, F_GETFL, 0) | O_NONBLOCK);
    return 0;
}

#define MAX_EVENTS 10
struct epoll_event ev, events[MAX_EVENTS];
int listen_sock, conn_sock, nfds, epollfd;

/* Code to set up listening socket, 'listen_sock',
   (socket(), bind(), listen()) omitted */

epollfd = epoll_create1(0);
if (epollfd == -1) {
    perror("epoll_create1");
    exit(EXIT_FAILURE);
}

// listen_sock registered in LT mode
ev.events = EPOLLIN;
ev.data.fd = listen_sock;
if (epoll_ctl(epollfd, EPOLL_CTL_ADD, listen_sock, &ev) == -1) {
    perror("epoll_ctl: listen_sock");
    exit(EXIT_FAILURE);
}
for (;;) {
    nfds = epoll_wait(epollfd, events, MAX_EVENTS, -1);
    if (nfds == -1) {
        perror("epoll_wait");
        exit(EXIT_FAILURE);
    }

    for (n = 0; n < nfds; ++n) {
        if (events[n].data.fd == listen_sock) {
            conn_sock = accept(listen_sock, (struct sockaddr *) &local, &addrlen);
            if (conn_sock == -1) {
                perror("accept");
                exit(EXIT_FAILURE);
            }
            setnonblocking(conn_sock);
            // conn_sock registered in ET mode
            ev.events = EPOLLIN | EPOLLET;
            ev.data.fd = conn_sock;
            if (epoll_ctl(epollfd, EPOLL_CTL_ADD, conn_sock, &ev) == -1) {
                perror("epoll_ctl: conn_sock");
                exit(EXIT_FAILURE);
            }
        } else {
            do_use_fd(events[n].data.fd);
        }
    }
}
```



## I/O 复用比较

参考《Linux高性能服务器编程》一书。

|      I/O       |                       select, pselect                        |                             poll                             |                            epoll                             |
| :------------: | :----------------------------------------------------------: | :----------------------------------------------------------: | :----------------------------------------------------------: |
|  监听事件集合  | 用户通过 3 个参数分别传入感兴趣的可读，可写及异常等事件；内核通过**对 3 个事件参数的修改**来反馈其中的就绪事件；这使得用户每次调用 select 都要重置这 3 个参数。 | 统一处理所有事件类型，因此只需要一个事件集参数；用户通过 `pollfd.events`传入感兴趣的事件，内核通过修改 `pollfd.revents` 反馈其中就绪的事件。 | 内核通过一个事件表直接管理用户感兴趣的所有事件。因此每次调用`epoll_wait` 时，无需反复传入用户感兴趣的事件。`epoll_wait` 系统调用的参数 `events` 仅用来反馈就绪的事件。 |
|  查询就绪事件  |                            $O(n)$                            |                            $O(n)$                            |                            $O(1)$                            |
| 最大描述符个数 |          一般由最大值限制，Linux 环境下常见是 1024           |                      `nfds_t` 的最大值                       |                            无限制                            |
|    工作模式    |                              LT                              |                              LT                              |                            LT, ET                            |
|    内核实现    |                 采用轮询检测就绪事件，$O(n)$                 |                 采用轮询检测就绪事件，$O(n)$                 |            采用回调函数的方式检测就绪事件，$O(1)$            |



I/O 复用这几个技术是一步一步发展过来的，依次为 `select/pselect -> poll -> epoll/kqueue` ，`kqueue` 我还没看过，暂且把它与 `epoll` 放在一起。

那么，后来出现的，一定是为了解决前面存在的问题的。`select` 存在哪些缺点呢？

- 每次调用 `select`，都需要把 `fd` 集合从用户态拷贝到内核态，这个开销在 `fd` 很多时会很大；
- 每次调用 `select` 都需要在内核遍历传递进来的所有 `fd` ，这个开销在 `fd `很多时也很大；
- 监听的文件描述符个数有限，一般是 1024 个；
- 从 `wait` 返回后，需要重新修改 3 个事件参数，才能再一次调用 `select` （参考下面的例子）；
- 需要遍历返回的描述符集合（或者说事件集合），来检测哪些描述符（事件）是就绪的。

```c
void test_select()
{
    char buf[BUFSIZ];
    fd_set readset;
    FD_ZERO(&readset);
    FD_SET(STDIN_FILENO, &readset);
    while (1)
    {
        select(STDIN_FILENO + 1, &readset, NULL, NULL, NULL);
        if (FD_ISSET(STDIN_FILENO, &readset))
        {
            bzero(buf, BUFSIZ);
            read(STDIN_FILENO, buf, BUFSIZ);
            printf("%s", buf);
            // 必须要有
            FD_ZERO(&readset);
            FD_SET(STDIN_FILENO, &readset);
        }
    }
}
```



`poll` 的作用其实与 `select` 类似，只不过是修改了事件的表示方法（参数与返回值分离），当从 `poll` 返回时，不需要重置原有的监听事件参数，它的缺点与 `select` 是类似的。

```c
void test_poll()
{
    char buf[BUFSIZ];
    struct pollfd pollev;
    pollev.fd = STDIN_FILENO;
    pollev.events = POLL_IN;
    pollev.revents = 0;
    while (1)
    {
        poll(&pollev, 1, -1);
        if (pollev.revents & POLLIN)
        {
            bzero(buf, BUFSIZ);
            read(STDIN_FILENO, buf, BUFSIZ);
            printf("%s", buf);
            pollev.revents = 0;  // 可有可无，但最好写上
        }
    }
}

```



Linux 2.6 之后出现 `epoll` ，它相对于 `select, poll` 有什么优点呢？

- 调用 `epoll_create` 时，在内核 cache 里建立 **红黑树** 用于存储以后 `epoll_ctl` 传来的 socket 外，还会再建立链表 `ready list` 。当 `epoll_wait` 调用时，仅观察 `ready list` 里有没有数据即可，有数据就返回，没有数据就 `sleep` ，时长由 `timeout` 确定。

- `epoll_create` 建立的红黑树来存放管理的 `fd`，所以在每次连接建立后，交给 epoll 管理时，需要将其添加到原先分配的空间中，后面再管理时就不需要频繁的从用户态拷贝管理的 `fd` 集合。因此，即使对同一  `fd` 多次调用 `epoll_ctl(epfd, op, fd, event)` ，也只会拷贝一次。
- 不采用轮询方式检测事件，而是通过更高效的回调函数 (Callback) 方式（😅 😅 😅 我也不理解这一点是什么意思）。
- 没有描述符个数限制。



## 总结

总算是写完了，写了点皮毛，至少会调包了😅😅😅，后面有空的话用 `epoll` 动手写一个多进程的 Client-Server 模型试试看😅😅😅。

I/O 复用好像还差 kqueue 😅😅😅 。



## References

- [1] https://man7.org/linux/man-pages/man7/epoll.7.html
- [2] [腾讯技术工程：网络 IO 演变发展过程和模型介绍](https://mp.weixin.qq.com/s/EDzFOo3gcivOe_RgipkTkQ)
- [3] https://www.cnblogs.com/Anker/p/3265058.html

