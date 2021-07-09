##  [UNP] IO 复用

📖 UNP Part-2: [ Chapter 6. I/O Multiplexing: The select and poll Functions](https://www.masterraghu.com/subjects/np/introduction/unix_network_programming_v1.3/ch06.html) 的读书笔记。

在 [这篇博客](https://www.cnblogs.com/sinkinben/p/14475209.html) 的最后，我们对文章中的服务器-客户端模型保留了这么一个问题：客户端同时存在 socket 和 stdin 两种 I/O ，但是它处理发方式仅仅是「运行到哪就读取哪」，即所谓的「阻塞型 I/O」，不能及时处理另外一个 I/O 所输入的信息。

解决这一问题的方法是 I/O 复用 (I/O Multiplex)。

I/O 复用适用于以下场合：

- 需要同时处理多个有关 I/O 的描述符（即上述的场景）
- 需要同时处理多个套接字
- 一个 TCP 服务器既要处理 `listen` 套接字，又要处理已连接的套接字
- 一个服务器既要处理 UDP，又要处理 TCP

参考资料：

- [CS-Notes](https://github.com/CyC2018/cs-notes)
- [腾讯技术工程：网络 IO 演变发展过程和模型介绍](https://mp.weixin.qq.com/s/EDzFOo3gcivOe_RgipkTkQ)
- [UNP Chapter 6](https://www.masterraghu.com/subjects/np/introduction/unix_network_programming_v1.3/ch06lev1sec2.html)



## I/O模型

Unix 环境下的 I/O 模型：

- 阻塞型 I/O (blocking I/O)
- 非阻塞型 I/O (non-blocking I/O)
- I/O 复用 (I/O Multiplexing): `select, poll` 函数
- 信号驱动 I/O (Signal Driven I/O): `SIGIO` 信号
- 异步 I/O (Asynchronous I/O): POSIX `aio_xxx` 系列函数

网络通信中，数据的传输一般经历 3 个层次：

```text
+---------------+
|应用进程（用户态）|
+---------------+
|操作系统（内核态）|
+---------------+
|  网络硬件接口   |
+---------------+
```

一个输入操作一般分为 2 个过程：

1. 等待数据准备好
2. 从内核向进程复制数据

对于 socket 套接字上的输入操作，第一步是等待数据从网络中传达，当所等待的报文分组到达时，它会复制到内核中的某个缓冲区。第二步是把内核缓冲区的数据复制到应用进程。

> PS: 有个叫「零拷贝」的知识点，可以在上述过程减少拷贝次数，具体措施是：将用户进程的部分地址空间与内核缓冲区建立内存映射（与进程通信中的共享内存类似）。
>
> 可以参考：[Link1](https://github.com/hope-valley/interview-question-set/blob/master/操作系统/知识点.md), [Link2](https://www.jianshu.com/p/fad3339e3448) .



## 阻塞型 I/O

常见的 `stdin` 都是阻塞型 I/O 。阻塞型 I/O 具体过程如下图所示。

<img src="https://gitee.com/sinkinben/pic-go/raw/master/img/20210306155816.png" style="width:67%;" />

如果通过阻塞 I/O 的方式调用 `recvfrom` ，该系统调用直到数据报文**到达且复制到用户进程中**，或者发生错误（例如被信号处理函数中断）才返回。



## 非阻塞 I/O

非阻塞 I/O 的具体行为是：相对于阻塞型 I/O 而言，当所请求的 I/O 操作需要阻塞时，非阻塞 I/O 模型不阻塞进程，而是返回一个错误，其过程如下图所示。

<img src="https://gitee.com/sinkinben/pic-go/raw/master/img/20210306160309.png" style="width:67%;" />

如果通过非阻塞 I/O 的方式调用 `recvfrom`，那么就要通过上图轮询 (Polling) 的方式，但显然这种方式是十分耗费 CPU 时间片。

```c
while (recvfrom(sockfd, buf, len, flags, src_addr, addrlen))
{
    if (errno == EWOULDBLOCK) continue;
    else
    {
        // do something according to buf
    }
}
```



## I/O 复用

<img src="https://gitee.com/sinkinben/pic-go/raw/master/img/20210306161459.png" style="width:67%;" />

与 I/O 复用相关的 API 是 `select` 和 `poll` ，这里通过讲解 `select` 函数和 `poll` 函数的具体行为来解释上图的过程。

### select

函数原型：

```C
/* According to POSIX.1-2001, POSIX.1-2008 */
#include <sys/select.h>
/* According to earlier standards */
#include <sys/time.h>
#include <sys/types.h>
#include <unistd.h>
int select(int nfds, fd_set *readfds, fd_set *writefds,
           fd_set *exceptfds, struct timeval *timeout);
// Returns: positive count of ready descriptors, 0 on timeout, –1 on error

void FD_CLR(int fd, fd_set *set);    // turn off the bit for fd in fdset
void FD_SET(int fd, fd_set *set);    // turn on the bit for fd in fdset
void FD_ZERO(fd_set *set);           // clear all bits in fdset
int  FD_ISSET(int fd, fd_set *set);  // is the bit for fd in fdset ?
```

`timeval` 的结构如下：

```C
struct timeval {
    time_t         tv_sec;     /* seconds */
    suseconds_t    tv_usec;    /* microseconds */
};
```

调用 `select` 会使进程阻塞，等待事件的到来，当且仅当下列 2 个条件发生时，进程唤醒：

- 阻塞超过指定的时间 `timeout` 。
- 描述符集合 `fdset` 中的任意一个或多个事件发生。

下面举例说明，所谓的「事件」是什么？

- `{1,4,5}` 中的任意描述符准备好读；
- `{1,4,5}` 中的任意描述符准备好写；
- `{1,4,5}` 中的任意描述符有异常事件等待处理；

下面对每个参数进行解析。

`timeval` 有 3 种可能：

- 空指针：永远等待下去，仅在有描述符准备好 I/O 时才返回；
- 零值：不等待，两个字段均为 0 ，这种情况不会阻塞进程，`select` 检查描述符后返回，也是所谓的轮询方式。
- 非零值：等待一段固定的时间，在有描述符准备好 I/O 时返回，但最多等待 `timeval` 。

`fd_set` 是一个结构体，成员是一串比特位（每个比特位代表描述符 `fd` 是否在这个集合当中），可以通过上面的 `FD_XXX` 系列函数来操作这些比特位。

`readfds, writefds, exceptfds` 三个参数指定让内核测试读、写和是否有异常的描述符集合，目前支持的异常事件有 2 个：

> - **The arrival of out-of-band data for a socket. It will be described in more detail in [Chapter 24](https://www.masterraghu.com/subjects/np/introduction/unix_network_programming_v1.3/ch24.html#ch24).** （Out-of-band Data 是一类特殊的数据，可参考 [IBM Out-of-band Data](https://www.ibm.com/support/knowledgecenter/ssw_ibm_i_71/rzab6/coobd.htm) ）
> - The presence of control status information to be read from the master side of a pseudo-terminal that has been put into packet mode. *We do not talk about pseudo-terminals in this book.* （与伪终端相关，UNP 一书不讨论这个，所以可忽略）

`select` 函数执行时会修改这三个 `fd_set` 参数，**当它返回时，这 3 个 `fd_set` 里存放的就是已经就绪的描述符，**因此可以通过 `FD_ISSET` 去检查哪些描述符已经就绪（如下面的代码所示）。

`nfds` 是待测试的描述符的个数，它的值是待测试的最大描述符 + 1，这样 `[1, 2, ..., nfds-1]` 都会被测试。

`select` 的主要作用是：在所有指定要求测试的描述符，只要有一个描述符有事件发生，那么阻塞在 `select` 上的进程就会被唤醒（即 `select` 函数返回），当 `select` 函数返回后，可以通过遍历 `fdset`，来找到就绪的描述符。

PS:「测试」一词的意思是让内核检测描述符是否有事件发生。

这是 I/O 复用模型的一个典型例子，通过 `select` 也就能够解决本文开头提出的问题。

**例子**

```c
fd_set fd_in, fd_out;
struct timeval tv;

// Reset the sets
FD_ZERO( &fd_in );
FD_ZERO( &fd_out );

// Monitor sock1 for input events
FD_SET( sock1, &fd_in );

// Monitor sock2 for output events
FD_SET( sock2, &fd_out );

// Find out which socket has the largest numeric value as select requires it
int largest_sock = sock1 > sock2 ? sock1 : sock2;

// Wait up to 10 seconds
tv.tv_sec = 10;
tv.tv_usec = 0;

// Call the select
int ret = select( largest_sock + 1, &fd_in, &fd_out, NULL, &tv );

// Check if select actually succeed
if ( ret == -1 )
    // report error and abort
else if ( ret == 0 )
    // timeout; no event detected
else
{
    if ( FD_ISSET( sock1, &fd_in ) )
        // input event on sock1

    if ( FD_ISSET( sock2, &fd_out ) )
        // output event on sock2
}
```



### pselect

函数原型：

```c
#include <sys/select.h>
int pselect(int nfds, fd_set *readfds, fd_set *writefds,
            fd_set *exceptfds, const struct timespec *timeout,
            const sigset_t *sigmask);
// Returns: count of ready descriptors, 0 on timeout, –1 on error
```

`pselect` 是有 POSIX 规范制定的，与 `select` 的区别如下：

1. 时间的结构体不同（就是时间单位变了，需要跟 POSIX 的 API 对接上）

```c
struct timespec {
    long    tv_sec;         /* seconds */
    long    tv_nsec;        /* nanoseconds */
};
```

注意到 select 的时间参数是没有 const 修饰的，原因是 select 执行后可能会改变 timeval 的值，改为还有多少时间剩余，而 pselect 不会修改 timeout 。

2. 增加参数 `sigmask`

如果 `sigmask` 为空，那么 `pselect` 与 `select` 的行为一致。

`sigmask` 是一个信号集合（其实就是一个比特位表示一个信号），用于指定进程的屏蔽信号，完成后恢复。它其实相当于 `select` 的下列操作：

```c
sigset_t originmask;
sigset_t sigmask;
sigprocmask(SIG_SETMASK, &sigmask, &originmask);  // save the original signal mask
int ret = select(nfds, readfs, writefds, exceptfds, timeout);
sigprocmask(SIG_SETMASK, &originmask, NULL);      // revert the signal mask of current process
```



### poll

函数原型：

```c
#include <poll.h>
int poll(struct pollfd *fds, nfds_t nfds, int timeout);
```

`pollfd` 结构体：

```c
struct pollfd {
    int   fd;         /* file descriptor */
    short events;     /* requested events */
    short revents;    /* returned events */
};
```

`poll` 的功能与 `select` 类似，也是等待一组描述符中的一个成为就绪状态，但工作方式不一样。

此外， `poll` 可以通过 `events` 指定某个 `fd` 的事件，而 `revents` 是作为返回值使用的。

事件也是通过比特位来表示的，具体如下图所示。

<img src="https://www.masterraghu.com/subjects/np/introduction/unix_network_programming_v1.3/files/06fig23.gif">

例子：

```C
// The structure for two events
struct pollfd fds[2];

// Monitor sock1 for input
fds[0].fd = sock1;
fds[0].events = POLLIN;

// Monitor sock2 for output
fds[1].fd = sock2;
fds[1].events = POLLOUT;

// Wait 10 seconds
int ret = poll( &fds, 2, 10000 );
// Check if poll actually succeed
if ( ret == -1 )
    // report error and abort
else if ( ret == 0 )
    // timeout; no event detected
else
{
    // If we detect the event, zero it out so we can reuse the structure
    if ( fds[0].revents & POLLIN )
        fds[0].revents = 0;
        // input event on sock1

    if ( fds[1].revents & POLLOUT )
        fds[1].revents = 0;
        // output event on sock2
}
```



### 小结

`select` 与 `epoll` 的区别如下：

1. `select` 能处理的最大连接，默认是 1024 个，可以通过修改配置来改变，但终究是有限个；而 `poll` 理论上可以支持无限个；
2. `select` 和 `poll` 在管理海量的连接时，会频繁的从用户态拷贝到内核态，比较消耗资源。
3. 如果平台支持并且对实时性要求不高，应该使用 `poll` 而不是 `select` 。

其实，`epoll` 也是 I/O 复用的重要知识点，但 UNP 这一章节没提到，后面有时间再整理这一部分内容。





## 信号驱动 I/O

这里需要信号相关知识，可以参考 APUE 一书的第 10 章。

信号驱动 I/O ：内核在描述符数据就绪时发送 `SIGIO` 信号通知进程。

<img src="https://gitee.com/sinkinben/pic-go/raw/master/img/20210306165416.png"  style="width:67%;" />

首先自定义 I/O 处理函数 `sigio_handler` ，通过 `signal(SIGIO, sigio_handler)` 指定当 `SIGIO` 信号发生时，执行该信号处理程序。

我们知道，信号这一机制本身就是异步的，那么信号驱动 I/O 其实也能算是一种特殊的「异步 I/O」，但与 POSIX 的异步 I/O 有所不同，下面会提到这一点。



## 异步 I/O

异步 I/O 是由 POSIX 规范来定义的，与之相关的 API 是 `aio_xxx` 系列函数。

与信号驱动 I/O 的区别是：信号驱动 I/O 是内核通知进程何时可以启动 I/O 操作；而 POSIX 的异步 I/O 是内核通知进程 I/O 操作何时完成，如下图所示。

<img src="https://gitee.com/sinkinben/pic-go/raw/master/img/20210306170041.png" style="width:67%;" />

在信号驱动 I/O 中，需要我们主动去调用 `recvfrom` ，但在这里的异步 I/O 中，这一操作也是通过 `aio_read` 来完成的。

### aio_read

函数原型：

```c
#include <aio.h>
int aio_read(struct aiocb *aiocbp);
```

调用该函数会立即返回，进程不会阻塞。

可通过参数向 `aio_read` 传递描述符、缓冲区指针、缓冲区大小（与 `read` 函数类似）和文件偏移量（与 `lssek` 类似），并告诉内核 I/O 操作完成时如何通知进程。

在 GUN Lib 下，`aiocb` 结构定义为：

```c
struct aiocb
{
  int aio_fildes;		/* File desriptor.  */
  int aio_lio_opcode;		/* Operation to be performed.  */
  int aio_reqprio;		/* Request priority offset.  */
  volatile void *aio_buf;	/* Location of buffer.  */
  size_t aio_nbytes;		/* Length of transfer.  */
  struct sigevent aio_sigevent;	/* Signal number and value.  */

  /* Internal members.  */
  struct aiocb *__next_prio;
  int __abs_prio;
  int __policy;
  int __error_code;
  __ssize_t __return_value;

#ifndef __USE_FILE_OFFSET64
  __off_t aio_offset;		/* File offset.  */
  char __pad[sizeof (__off64_t) - sizeof (__off_t)];
#else
  __off64_t aio_offset;		/* File offset.  */
#endif
  char __glibc_reserved[32];
};
```



## I/O 模型比较

- 同步 I/O：将数据从内核缓冲区复制到应用进程缓冲区的阶段（第二阶段），应用进程会阻塞。
- 异步 I/O：第二阶段应用进程不会阻塞。

上述介绍的前 4 种，阻塞型、非阻塞型、I/O 复用、信号驱动都是属于同步 I/O，因为真正调用 `recvfrom` 执行 I/O 操作的时候可能会阻塞进程。

<img src="https://gitee.com/sinkinben/pic-go/raw/master/img/20210306190446.png" style="width:80%;" />









