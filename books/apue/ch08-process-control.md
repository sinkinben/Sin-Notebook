## [APUE] 进程控制

📒 APUE 一书的第八章学习笔记。

## 进程标识

大家都知道使用 PID 来标识的。

系统中的一些特殊进程：

- PID = 0: 调度进程，也称为交换进程 (Swapper)
- PID = 1: `init` 进程，自检结束后由内核调用，读取与系统初始化相关的文件，如 `/etc/init.d/*, /etc/rc*/` . `init` 进程是一个以 `root` 启动的普通进程，而不是像 Swapper 是一个内核进程。**`init` 是所有孤儿进程的父进程。**
- PID = 2: 页守护进程 (Page Daemon), 为虚拟存储器的分页操作提供支持。

关于进程标识的 API ：

```c
#include <unistd.h>
pid_t getpid(void);   // Returns: process ID of calling process
pid_t getppid(void);  // Returns: parent process ID of calling process
uid_t getuid(void);   // Returns: real user ID of calling process
uid_t geteuid(void);  // Returns: effective user ID of calling process
gid_t getgid(void);   // Returns: real group ID of calling process
gid_t getegid(void);  // Returns: effective group ID of calling process
```



## fork

```c
#include <unistd.h>
pid_t fork(void); // Returns: 0 in child, process ID of child in parent, −1 on error
```

`fork` 的一些特点：

- 调用 1 次，返回 2 次；
- 为什么将子进程的 ID 返回给父进程？一个进程可有多个子进程，但没有函数可以获得所有子进程的 ID 。
- 为什么 `fork` 返回给子进程的是 0 ？因为子进程的 PID 不可能为 0 ，它的父进程 PID 可以由 `getppid()` 获取。

`fork` 返回后，父子进程都会在 `fork` 的调用点继续执行。子进程会获得父进程的数据空间、堆和栈的副本，但应当注意的是子进程拥有的是副本，而不是父子进程一同共享这些数据。**父子进程共享的只有程序的 text 段。**

由于 `fork` 之后经常会跟着 `exec` 函数，所以很多时候并不修改父进程的数据段和堆栈。为了针对这一特点进行优化，实现当中采用了写时复制 (Copy On Write), 父子进程共享这些区域，但内核会将它们的权限修改为只读 (Read-Only). 如果父子进程中的一个试图修改这些区域，则内核只会为被修改区域的那块内存拷贝一份副本，通常是虚拟存储器中的“一页”。

一般来说，`fork` 之后，父子进程是并发执行的，为此还需要实现进程间的同步操作（例如信号）。

`fork` 一般有 2 种常见用法：

1. 父进程复制自己，父子进程同时执行不同的代码段。这种情况常见于网络服务进程：父进程等待客户端的请求，当请求到达时，父进程调用 `fork` ，使子进程处理该请求，而父进程继续等待下一请求。
2. 一个进程需要执行不同的程序。例如 Shell 程序，子进程从 `fork` 返回之后调用 `exec` 系列函数。在某些系统中，会把 `fork, exec` 封装为一种操作 `spawn` .



**例子**

```cpp
#include "apue.h"
int globalvar = 123;
char buf[] = "a write to stdout\n";
int main()
{
    int var = 233;
    pid_t pid;
    if (write(STDOUT_FILENO, buf, sizeof(buf) - 1) != sizeof(buf) - 1) err_sys("write err\n");
    if ((pid = fork()) < 0) err_sys("fork err\n");
    else if (pid == 0) var++, globalvar++;
    else sleep(2);
    printf("pid=%d, globalvar=%d, var=%d\n", pid, globalvar, var);
    return 0;
}
```

输出：

```
$ ./a.out 
a write to stdout
pid=0, globalvar=124, var=234
pid=15438, globalvar=123, var=233
```

**文件共享**

`fork` 之后，子进程会拥有父进程的文件描述符表的副本，如下图所示。

<img src="https://gitee.com/sinkinben/pic-go/raw/master/img/20210126203912.png" style="width:70%;" />

所以：

- 父进程的重定向`dup`也会被子进程继承。
- 父子进程共享某一打开文件的偏移量。如果父子进程同时对该文件进行写操作（但没有任何同步机制），那么就会造成数据的混乱。

## vfork

```c
#include <sys/types.h>
#include <unistd.h>
pid_t vfork(void);
```

`vfork` 用于创建一个子进程，而该子进程的目的是执行 `exec` 系列函数。

`vfork` 并不会把父进程的地址空间完全复制到子进程中，因为考虑到子进程会马上调用 `exec` （因而不会引用该地址空间的数据），不过在它调用 `exec, exit` 之前，它一直在父进程的地址空间中运行。但如果子进程后续没有调用 `exec` 或者 `exit`，是一种未定义行为。

`vfork` 和 `fork` 的另外一个重要区别是：`vfork` 保证子进程先运行，在它调用 `exec, exit` 之后父进程才可能被调度运行（如果这 2 个调用依赖于父进程的进一步动作，那么会产生死锁）。

**例子**

```c
int globvar = 6;
int main()
{
    int var = 88;
    pid_t pid;
    printf("before vfork\n");
    if ((pid = vfork()) < 0) err_sys("vfork err");
    else if (pid == 0)
    {
        globvar++, var++;
        _exit(0);
    }
    printf("pid = %u, globvar = %d, var = %d\n", pid, globvar, var);
    return 0;
}
```

输出：

```
before vfork
pid = 3449, globvar = 7, var = 89
```

结果表明子进程修改了父进程的数据。



## wait and waitpid

当一个子进程结束（不论是正常终止还是异常中止），内核会向父进程发送 `SIGCHILD` 信号。因为子进程中止是一个异步事件（这可以在父进程运行的任何时候发生），因此该信号也是内核向父进程发送的异步信号。

父进程接收到某一信号时，采取的措施可以是忽略，也可以是调用信号处理函数。对于 `SIGCHILD` 默认的措施是忽略。

```c
#include <sys/wait.h>
pid_t wait(int *statloc);
pid_t waitpid(pid_t pid, int *statloc, int options);
// Both return: process ID if OK, 0 (see later), or −1 on error
```

作用：

1. 如果所有子进程都在运行中，阻塞调用进程（即父进程）。
2. 如果一个子进程已经结束，正等待父进程获取它的结束状态，则父进程取得子进程的中止状态后立即返回。
3. 如果没有任何子进程，则返回 -1（通过 `strerror(errno)` 获取的错误信息为 `No child processes`）。

二者的区别：

- 在一个子进程结束前，`wait` 使得父进程阻塞（只要有 1 个子进程结束，父进程就唤醒，返回值是刚刚结束的子进程的 `pid` ）；而 `waitpid` 可以通过参数设置，使得父进程不阻塞。
- `wait` 可以选择等待某一进程 `pid` 。
- `waitpid(-1, &status, 0)` 等价于 `wait(&status)` .

下面解析 3 个参数 `pid, statloc, options` .

`statloc` 用于获取子进程的结束状态，不同的比特位表示不同的含义，可以通过以下宏定义获取相关信息。

<img src="https://gitee.com/sinkinben/pic-go/raw/master/img/20210128213024.png" style="width:75%;" />

在 `waitpid` 中 `pid` 的解释如下：

- `pid == -1`: 等待任意一个子进程。
- `pid > 0` : 等待 `pid` 指定的进程。
- `pid == 0` : 等待 Group ID 等于调用进程组 ID 的**任意一个子进程**。
- `pid < -1` : 等待 Group ID 等于 `pid` 绝对值的任意一个子进程。

`options` 可以为 0 ，或者以下常量的或运算 `|` 的结果：

<img src="https://gitee.com/sinkinben/pic-go/raw/master/img/20210129144401.png" style="width:75%;" />

**例子 1**

```cpp
#include <sys/wait.h>
#include "apue.h"
void pr_exit(int status)
{
    if (WIFEXITED(status))
        printf("normal termination, exit status = %d\n", WEXITSTATUS(status));
    else if (WIFSIGNALED(status))
        printf("abnormal termination, signal number = %d%s\n",
               WTERMSIG(status),
#ifdef WCOREDUMP
               WCOREDUMP(status) ? " (core file generated)" : "");
#else
               "");
#endif
    else if (WIFSTOPPED(status))
        printf("child stopped, signal number = %d\n", WSTOPSIG(status));
}

int main()
{
    pid_t pid;
    int status;
    // case-1: childs exits with 7
    if ((pid = fork()) < 0)   err_sys("fork err\n");
    else if (pid == 0)        exit(7);
    if (wait(&status) != pid) err_sys("wait err\n");
    pr_exit(status);

    // case-2: child aborts
    if ((pid = fork()) < 0)   err_sys("fork err\n");
    else if (pid == 0)        abort();
    if (wait(&status) != pid) err_sys("wait err\n");
    pr_exit(status);

    // case-3: 0 as the divider in child
    if ((pid = fork()) < 0)   err_sys("fork err\n");
    else if (pid == 0)        status /= 0;
    if (wait(&status) != pid) err_sys("wait err\n");
    pr_exit(status);
    return 0;
}
```

输出：

```text
normal termination, exit status = 7
abnormal termination, signal number = 6 (core file generated)
abnormal termination, signal number = 8 (core file generated)
```

**例子 2 : 僵尸进程**

```c
#include "apue.h"
#include <sys/wait.h>
int main()
{
    pid_t pid;
    if ((pid = fork()) < 0)  err_sys("fork err");
    else if (pid == 0)
    {
        if ((pid = fork()) < 0) err_sys("fork err");
        else if (pid > 0)       exit(0);
        // child-2 continues when its parent exit
        // then child-2's parent will be init (pid=1)
        sleep(2);
        printf("second child, parent pid = %u\n", getppid());
        exit(0);
    }
    if (waitpid(pid, NULL, 0) != pid) err_sys("waitpid err");
    exit(0);
}
// Output: second child, parent pid = 1
```

## waitid

```c
#include <sys/wait.h>
int waitid(idtype_t idtype, id_t id, siginfo_t *infop, int options);
// Returns: 0 if OK, −1 on error
```

`waitid` 与 `waitpid` 相比，具有更多的灵活性。

`waitid` 允许等待指定的某一子进程，但它使用 2 个单独的参数表示要等待的子进程的所属类型。

`idtype` 的含义如下：

<img src="https://gitee.com/sinkinben/pic-go/raw/master/img/20210129153225.png" style="width:75%;" />

`options` 是下列常量按位或运算的结果：

<img src="https://gitee.com/sinkinben/pic-go/raw/master/img/20210129153423.png" style="width:75%;" />

## Race Condition

`fork` 之后不能保证父进程与子进程哪一个先执行，因此容易发生 Race Condition，解决竞争问题需要同步机制。

显然 `wait` 是一种同步操作，保证了父进程在子进程结束后才能运行。

反过来，如果子进程想等待父进程结束，可以通过**轮询 (Polling)**的方式：

```c
while (getppid() != 1)
	sleep(1);
```

子进程每隔 1 秒被唤醒，然后进行条件测试，满足条件后才能继续运行。但这种轮询方式浪费 CPU 的时间片，效率是极其低下的。

因此，多进程之间需要有某种形式的信号发送与接收方法，来实现多进程的同步。这些内容将在后面继续讨论。



## exec

终于看到本章的重点内容了。

当进程调用 `exec` 函数，该进程的内容就被完全替换为指定的新程序，新程序从它的 `main` 开始执行。应当注意的是：`exec` **不会创建新的进程**，所以调用前后的进程 ID 不会变，`exec` 只是用磁盘上的某一程序替换了当前的 text 段，数据段，堆和栈。

```c
#include <unistd.h>
int execl(const char *pathname, const char *arg0, ... /* (char *)0 */ );
int execv(const char *pathname, char *const argv[]);
int execle(const char *pathname, const char *arg0, ... /* (char *)0, char *const envp[] */ );
int execve(const char *pathname, char *const argv[], char *const envp[]); 
int execlp(const char *filename, const char *arg0, ... /* (char *)0 */ ); 
int execvp(const char *filename, char *const argv[]);
int fexecve(int fd, char *const argv[], char *const envp[]);
// All seven return: −1 on error, no return on success
```

先说 `pathname` 与 `filename` 的区别：

- `pathname` 是相对于当前工作目录的路径；
- `filename`: 如果包含 `/` 符号，就将其视为路径；否则在 `PATH` 环境变量包含的目录中查找。

如果 `execlp, execvp` 的 `filename` 指向的不是一个由 Linker 产生的二进制可执行文件，那么会认为 `filename` 指向的是一个 Shell 脚本，调用 `/bin/sh` 或者 `/bin/bash` 执行之。比如：

```c
// Content of file 'echo3': echo $1 $2 $3
execlp("/home/sinkinben/workspace/apue/echo3", "echo3", "sin", "kin", "ben", NULL);
// or
char* argv[] = {"echo3", "sin", "kin", "ben", NULL};
execvp("/home/sinkinben/workspace/apue/echo3", argv);
```

`fexecve` 根据调用者提供的 `fd` 来寻找可执行文件。调用者可以使用文件描述符验证所需要的的文件存在，并且无竞争地执行该文件。否则如果在调用 `exec` 前，`pathname, filename` 指向的可执行文件的内容被恶意篡改，容易引发安全漏洞。

第二个区别是参数列表的传递方式（函数名字的 `l` 表示 `list`, `v` 表示 `vector`）。

- `l` 表示将调用的命令行参数通过一个单独的参数传递（如上面的 `execlp` ），最后带一个 `NULL` 。
- `v` 表示命令行参数需要组合成一个数组的形式（如上面的 `execvp`）。

对于 `execle, execve` 允许通过 `char *const envp[]` 设置环境表（`e`表示 `envp`）。

此外，函数名还有一个 `p` 的 `execlp, execvp` ，其中 `p` 表示该函数以 `filename` 作为参数，可以在 `PATH` 中寻找可执行文件。

下图为 7 个 `exec` 函数的对比。

<img src="https://gitee.com/sinkinben/pic-go/raw/master/img/20210129172755.png" style="width:75%;" />

下图为 7 个 `exec` 的关系图。

<img src="https://gitee.com/sinkinben/pic-go/raw/master/img/20210129173059.png" style="width:65%;" />

对于 `fexecve` 而言，它会把 `fd` 参数转换为形如 `/proc/{pid}/fd/{x}` 的路径（该路径「指向」某一可执行文件）。

**例子**

```c
char *env_init[] = {"USER=unknown", "PATH=/tmp", NULL};
int main()
{
    pid_t pid;
    if ((pid = fork()) < 0) err_sys("fork err");
    else if (pid == 0)
    {
        if (execle("/tmp/echoall", "echoall", "arg1", "arg2", NULL, env_init) < 0)
            err_sys("execle err");
    }
    waitpid(pid, NULL, 0);
    exit(0);
}
```

其中 `echoall` 是一个打印 `argv` 和 `environ` 的程序（编译后放在 `/tmp` 下）：

```c
int main(int argc, char *argv[])
{
    int i;
    extern char **environ;
    for (i = 0; i < argc; i++) printf("argv[%d] = %s\n", i, argv[i]);
    for (i = 0; environ[i] != NULL; i++) puts(environ[i]);
}
```

运行结果：

```
argv[0] = echoall
argv[1] = arg1
argv[2] = arg2
USER=unknown
PATH=/tmp
```

## 例子

最后来看个例子，如何实现 Shell 中的管道 `|` 功能。

```cpp
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

