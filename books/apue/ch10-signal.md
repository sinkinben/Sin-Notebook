## [APUE] Signal

📖 APUE 第 10 章读书笔记。

## 引言

信号是一种软中断 (Software Interrupts)。

- 硬中断是外部设备对 CPU 的中断；
- 软中断通常是硬中断服务程序对内核的中断；
- 信号则是由内核（或其他进程）对某个进程的中断

软中断其实是利用硬件中断的概念，用软件方式进行模拟，实现宏观上的异步执行效果。

中断有若干个过程（微机原理这门阴间的课程上学过的），比如中断请求、中断判优、保护现场之类的。



## 信号的概念

每个 Signal 都有一个名字，在 Unix 下通常以 `SIG` 开头，比如：

- `SIGABRT` 是 `abort` 的信号，当进程调用 `abort()` 函数产生这种信号。
- `SIGALRM` 是闹钟信号，由 `alarm()` 函数的定时器超时后产生该信号。

`SIGXXX` 其实是一个整数，在 Ubuntu 下，在头文件 `signum.h` 可以找到所有的信号，范围是 `1 - 31` .

为什么没有 0 信号呢？这是因为 `kill(pid, signo)` 函数保留了 `kill(pid, signo = 0)` 作为特殊用途。

常见的场景：

- Ctrl + C，产生中断信号 `SIGINT` ，停止一个进程。
- 硬件异常信号 `SIGSEGV` ，当除数为 0 ，无效内存引用时发生。

当某个信号发生时，必须要告诉内核按下列的 3 种方式之一去处理：

- **忽略信号 (Ignore)**：大多数信号都可以忽略，但 `SIGKILL` 和 `SIGSTOP` 不能忽略。这 2 种信号不能忽略的原因是它们向内核和超级用户提供了终止进程的可靠方法。此外，如果忽略 `SIGSEGV` 这种硬件异常信号，则进程后续的行为是未定义的。
- **捕获信号 (Catch)**：提供一个函数，与信号绑定，当信号发生时，内核自动调用这个函数。`SIGKILL` 和 `SIGSTOP` 不能被捕获。
- **执行默认动作 (Default)**：对于大多数信号的默认动作是终止该进程。

下图列出了所有信号，说明哪种系统支持这一信号，通过 `·` 来表示。（😅 其实我没搞懂 terminate 和 stop 的区别在哪）

`core` 表示在进程的当前工作目录中的 core 文件复制该进程的内存镜像，以便于后续调试进程的终止状态（查找错误原因）。

<img src="https://gitee.com/sinkinben/pic-go/raw/master/img/20210225155220.png" style="width:80%;" />

## 函数 signal

```c
#include <signal.h>
typedef void (*sighandler_t)(int);
sighandler_t signal(int signum, sighandler_t handler);
// Returns: previous disposition of signal (see following) if OK, SIG_ERR on error
```

注册信号处理函数。成功返回 **原有的** 信号处理程序，失败返回 `SIG_ERR` 。

`SIG_ERR` 表明上看是一个函数指针，但实际上在 `signum.h` 被定义为：

```c
/* Fake signal functions.  */
#define SIG_ERR	((__sighandler_t) -1)		/* Error return.  */
#define SIG_DFL	((__sighandler_t) 0)		/* Default action.  */
#define SIG_IGN	((__sighandler_t) 1)		/* Ignore signal.  */
```

其实也不一定非要 -1，0 和 1 。只要这三个值不是一个有效的函数地址就可以了。

根据 `signal` 的返回值，我们可以知道信号已有的处理行为，但如果我们只是想知道信号的处理行为，而改变它的处理行为呢？这时候 `signal` 就不管用了，需要用到 `sigaction` 。

当一个进程 `fork` 出一个子进程，**子进程会继承父进程的信号处理方式。**因为子进程是父进程的镜像，因此信号的处理函数的地址在子进程中同样有效。

例如下列程序运行后，按下 `CTRL+C` ：

```c
#include <signal.h>
#include <stdio.h>
#include <stdlib.h>
void sigint_handler(int signo)
{
    printf("\nsigno = %d\n", signo);
    exit(0);
}
int main()
{
    signal(SIGINT, sigint_handler);
    while (1) ;
}
```

则会输出：

```text
^C
signo = 2
```

Unix 还保留了 `SIGUSR1` 和 `SIGUSR2` 2 个信号，留给用户自定义。

```c
#include <signal.h>
#include <stdio.h>
#include <stdlib.h>
void sigusr1_handler(int signo)
{ printf("SIGUSR1 is received, signo = %d\n", signo); }
void sigusr2_handler(int signo)
{ printf("SIGUSR2 is received, signo = %d\n", signo); }
int main()
{
    signal(SIGUSR1, sigusr1_handler);
    signal(SIGUSR2, sigusr2_handler);
    while (1);
}
```

输出：

```
$ ./a.out &
[1] 20783
$ kill -USR1 20783
SIGUSR1 is received, signo = 10
$ kill -USR2 20783
SIGUSR2 is received, signo = 12
```



## 不可靠信号

在早期的 Unix 版本当中，信号是不可靠的，不可靠是指：信号可能会丢失，当一个信号发生时，进程可能不知道这个信号的存在。同时，进程对信号的控制能力也很差，有时候用户希望通知内核阻塞某个信号，但不要忽略它，在它发生的时候记录下来，然后等进程准备好的时候再做处理，这种阻塞信号在当时也不具备。



## 被中断的系统调用

在早期的 Unix 中，如果进程在执行一个很慢的系统调用时被阻塞，在这期间捕获到一个信号，则该系统调用会被中断而不再执行。同时，该系统调用把 `errno` 设置为 `EINTR` 。

为了支持这种特性，把系统调用分为 2 类：Slow system calls and the others . Slow system call 是可能会使进程永远阻塞的一类系统调用：

- 如果某些数据不存在（如读管道），则读操作可能会使调用者永远阻塞。
- `pause` 函数，它会使调用进程休眠直至捕获到一个信号。
- `wait` 函数，当所有子进程都不会结束时，父进程调用 `wait` 可能永远阻塞。



## 可重入函数

可重入函数，即 Reentrant Functions 。

如果进程捕获到一个信号时，它正在执行正常的指令序列，那么这时候**正常的指令序列会被信号处理程序临时中断**，进程首先执行信号处理程序。

如果进程可以从信号处理程序返回（即不会调用 `exit` 或者 `longjmp`），则**可以回到被中断的指令处继续执行**。

根据上述的 2 点特性，假设这样的场景：如果进程正在执行 `malloc`，在堆上申请空间，此时捕获到信号并跳转执行，而信号处理程序又调用了 `malloc`，这是会发生什么呢？

这可能造成进程崩溃，因为 `malloc` 维护了一个链表，在跳转到信号处理程序时，进程可能在修改该链表。

Signal Unix Specification 提供了**可以在信号处理程序中被安全调用的函数**。这些函数是可重入的 (Reentrant) ，并且被称为是异步信号安全的 (Async-signal safe) 。除了可重入以外，在信号处理期间，这些函数会阻塞任何引起不一致的信号发送。

⚠️ 「可重入」一词的意思就是：在某个函数执行期间，被信号处理程序中断，当信号处理完成，可以回到被中断的指令处继续执行，不会引起任何错误，这时候该函数就称为是「可重入」的。

下图列出了一些常见的可重入函数（不在这个图中的函数大多数是不可重入的）。

<img src="https://gitee.com/sinkinben/pic-go/raw/master/img/20210225171108.png" style="width:80%;" />



## 可靠信号术语与语义

当一个信号发生时，内核通常在进程表中设置一个标志。当对信号采取了这种动作，我们就说向进程传递 (Delivery) 了一个信号。在信号产生 (Generation) 和 传递 (Delivery) 之间的时间间隔内，称信号是未决的 (Pendding) 。

如果为进程产生了一个阻塞的信号，而且对该信号的处理是默认动作或捕获信号，则为该进程将此信号保持为未决状态，直到该进程对此信号解除阻塞，或者将对此信号的动作更改为忽略。

**内核在递送一个原来被阻塞的信号给进程时（而不是在产生该信号时），才决定对它的处理方式。**于是进程在信号递送给它之前仍可改变对该信号的动作。进程调用 `sigpending` 函数来判定哪些信号是设置为阻塞并处于未决状态的。





## kill 和 raise

`kill` 将信号发送给进程或者进程组，`raise` 允许进程向自身发送信号。

```c
#include <signal.h>
int kill(pid_t pid, int signo); 
int raise(int signo);
// Both return: 0 if OK, −1 on error
```

显然，调用 `raise(signo)` 等价于 `kill(getpid(), signo)` 。

`kill` 的 `pid` 参数有 4 种情况：

- `pid > 0` : 发送信号给进程 `pid` 。
- `pid == 0` : 将信号发送给与该进程同一进程组的所有进程。
- `pid < 0` : 将信号发送给进程组 ID 等于 `pid` 绝对值的所有进程。
- `pid == -1` : 将信号发送给该进程有权限向它们发送信号的所有进程。

特别的，`kill(pid, 0)` 会执行正常的错误检查，但不发送任何信号，常用于确定进程 `pid` 是否依然存在。



## alarm 和 pause

`alarm` 设定一个定时器，如果超时，那么产生 `SIGALRM` 信号，如果不忽略或者捕获该信号，默认处理行为是终止进程。

```c
#include <unistd.h>
unsigned int alarm(unsigned int seconds);
// Returns: 0 or number of seconds until previously set alarm
```

`pause` 挂起进程直到捕获到一个信号。

```c
#include <unistd.h>
int pause(void);
// Returns: −1 with errno set to EINTR
```



## 信号集

信号集 (Signal Set) 通过一个结构体来表示：

```c
# define _SIGSET_NWORDS	(1024 / (8 * sizeof (unsigned long int)))
typedef struct
  {
    unsigned long int __val[_SIGSET_NWORDS];
  } __sigset_t;
#endif
```

信号集相关 API：

```c
#include <signal.h>
int sigemptyset(sigset_t *set);
int sigfillset(sigset_t *set);
int sigaddset(sigset_t *set, int signo); 
int sigdelset(sigset_t *set, int signo);
// All four return: 0 if OK, −1 on error 
int sigismember(const sigset_t *set, int signo);
// Returns: 1 if true, 0 if false, −1 on error
```

作用：

- `sigemptyset` ：清除 `set` 中的所有信号。
- `sigfillset`：使得 `set` 包括所有信号。
- `sigaddset, sigdelset`：向集合 `set` 中添加或者删除 `signo`。
- `sigismember`: `signo` 是否在 `set` 当中。

个人有一点不太明白的是：信号的范围是 1-31 ，实际上我们可以通过一个 `uint32_t` 来表示信号集合，然后上述 API 其实都可以通过简单的位操作实现了。搞不懂 Ubuntu 上的这种实现考虑到了什么样的场景。

正如书本给出的样例代码：

```c
#include <signal.h>
#include <errno.h>
/*
 * <signal.h> usually defines NSIG to include signal number 0.
 */
#define SIGBAD(signo) ((signo) <= 0 || (signo) >= NSIG)
int sigaddset(sigset_t *set, int signo)
{
    if (SIGBAD(signo))
    {
        errno = EINVAL;
        return (-1);
    }
    *set |= 1 << (signo - 1);
    return (0);
}
int sigdelset(sigset_t *set, int signo)
{
    if (SIGBAD(signo))
    {
        errno = EINVAL;
        return (-1);
    }
    *set &=  ̃(1 << (signo - 1));
    return (0);
}
int sigismember(const sigset_t *set, int signo)
{
    if (SIGBAD(signo))
    {
        errno = EINVAL;
        return (-1);
    }
    return ((*set & (1 << (signo - 1))) != 0);
}
```



## sigprocmask

```c
int sigprocmask(int how, const sigset_t *restrict set, sigset_t *restrict oset);
// Returns: 0 if OK, −1 on error
```

在「不可靠信号」一节中提到，在某些场景下可能需要阻塞信号，直到进程准备就绪才执行信号处理程序，这就需要 `sigprocmask` 。

参数解析：

- `oset` 如果为非空指针，那么进程当前的信号屏蔽集合通过 `oset` 返回；
- `set` 如果为非空指针，则 `how` 指示如何修改当前的信号屏蔽字：
  - `SIG_BLOCK` : 当前屏蔽集合与参数 `set` 取并集后，设置为进程的信号屏蔽字，即：`set` 包括了希望阻塞的信号。
  - `SIG_UNBLOCK` : 当前屏蔽集合与 `set` 的补集取交集，即：`set` 包括了希望解除阻塞的信号。
  - `SIG_SETMASK` : 把 `set` 设置为进程的屏蔽集合。

注意：不能阻塞 `SIGKILL` 和 `SIGSTOP` 信号；**`sigprocmask` 只能在单线程进程中使用，多线程场景下，应当使用 `pthread_sigmask` .**



## sigpending

```c
#include <signal.h>
int sigpending(sigset_t *set);
// Returns: 0 if OK, −1 on error
```

获取当前进程处于 Pendding 状态的信号集合，通过 `set` 返回。

**例子**

```c
#include "apue.h"
#include <signal.h>
static void sig_quit(int signo)
{
    printf("caught SIGQUIT\n");
    if (signal(SIGQUIT, SIG_DFL) == SIG_ERR)  err_sys("can't reset SIGQUIT");
}
int main()
{
    sigset_t newmask, oldmask, pendmask;
    if (signal(SIGQUIT, sig_quit) == SIG_ERR) err_sys("can't not catch SIGQUIT");
    
    // Block SIGQUIT and save current signal mask
    sigemptyset(&newmask);
    sigaddset(&newmask, SIGQUIT);
    
    if (sigprocmask(SIG_BLOCK, &newmask, &oldmask) < 0)  err_sys("SIG_BLOCK error");
    sleep(5);  // SIGQUIT here will remain pending
    
    if (sigpending(&pendmask) < 0)  err_sys("sigpending error");
    if (sigismember(&pendmask, SIGQUIT))  printf("\nSIGQUIT pending\n");
    
    if (sigprocmask(SIG_SETMASK, &oldmask, NULL) < 0)  err_sys("SIG_SETMASK error");
    printf("SIGQUIT unblocked\n");
    sleep(5);
    exit(0);
}
```

运行结果：

```text
$ ./a.out 
^\^\^\^\^\
SIGQUIT pending
caught SIGQUIT
SIGQUIT unblocked
```



## sigaction

```c
int sigaction(int signo, const struct sigaction *restrict act,
struct sigaction *restrict oact);
// Returns: 0 if OK, −1 on error
```

结构体：

```c
struct sigaction {
    void     (*sa_handler)(int);
    void     (*sa_sigaction)(int, siginfo_t *, void *);
    sigset_t   sa_mask;
    int        sa_flags;
    void     (*sa_restorer)(void);
};
```

`sigaction` 的作用是检查或者修改与 `signo` 相关联的信号处理程序。

参数：

- 若 `act` 指针非空，则修改信号的处理程序为 `act` 。
- 若 `oact` 指针非空，则通过 `oact` 返回原有的信号处理程序。

---

看不动了，真鸡儿无聊 😅 ，2021/03/01





## 总结

**1. SIGINT、SIGQUIT、SIGTERM 与 SIGSTOP**

- SIGINT：程序终止 (interrupt) 信号, 在用户键入 INTR 字符 (Ctrl + C) 时发出，用于通知前台进程组终止进程。 
- SIGQUIT：和 SIGINT 类似, 但由 QUIT 字符 (Ctrl + \\) 来控制. 进程在因收到 SIGQUIT 退出时会产生core文件, 在这个意义上类似于一个程序错误信号。 
- SIGTERM: 程序结束 (terminate) 信号, 与 SIGKILL 不同的是，**该信号可以被阻塞和处理**。通常用来要求程序自己正常退出，shell命令kill 缺省产生这个信号。如果进程终止不了，我们才会尝试 SIGKILL。 
- SIGSTOP：停止 (stopped) 进程的执行。注意它和 terminate 以及 interrupt 的区别: 该进程还未结束, 只是**暂停执行**。本信号不能被阻塞, 处理或忽略. 



**2. 进程收到多个信号的情况**

- 待处理信号被阻塞：Unix 信号函数会阻塞统一类型的待处理信号。就比如，一个程序接受了一个 SIGCHLD 的信号，那么在执行 SIGCHLD 的信号的时候，下一的 SIGCHLD 的信号就会被阻塞，成为待处理信号。
- 待处理信号不会排队等待。即针对同一类型的信号，只能有一个待处理信号。例如，一个进程接受了一个 SIGCHLD 的信号，在执行 SIGCHLD 的信号处理程序的时候，来了两个 SIGCHLD 信号，那么只有一个 SIGCHLD 会成为待处理信号。
- 系统调用可以被中断。像 `read`，`wait`，`accept` 这样的系统调用潜在地会阻塞进程一段时间，称为慢速系统调用 (*low system call*) 。在某些系统中，当处理程序捕获到一个信号，被中断的系统调用在信号处理程序进行后不再返回，而是立即返回给用户一个错误条件，并将 `errno` 设置为 `EINTR` 。

