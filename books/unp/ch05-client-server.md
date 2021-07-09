## [UNP] TCP 多进程服务器

📖 UNP Part-2: [Chapter 5. TCP Client/Server Example](https://www.masterraghu.com/subjects/np/introduction/unix_network_programming_v1.3/ch05.html) 的读书笔记。

阅读本文前，建议先阅读[多线程服务器的实现](https://www.cnblogs.com/sinkinben/p/14440863.html)，熟悉常见的 TCP 网络通信 API 的基本使用。

本章的主要内容是基于 TCP 协议，实现一个多进程服务器的 Demo，作者假设了若干个场景，借此来说明在代码细节上需要注意的一些问题。



**常用命令**

```
netstat -a | grep 9877
ps -t pts/16 -o pid,ppid,tty,stat,args,wchan
```

`pts/16` 中的 16 需要修改。



**文件说明**

|              文件              |                描述                |
| :----------------------------: | :--------------------------------: |
| `client-v1.c` 和 `server-v1.c` |       原始版本的多进程服务器       |
|         `server-v2.c`          |       添加捕获信号 `SIGCHLD`       |
|         `client-v2.c`          |     发起 5 个 TCP 连接的客户端     |
|         `server-v3.c`          | 改进信号处理函数 `sigchild_hander` |
|            `unp.h`             |      头文件声明和一些辅助函数      |



**预备知识**

- 进程控制 API：`fork, signal` .
- 网络通信 API：`socket, listen, bind, accept, connect` .



代码：https://github.com/sinkinben/unp-code/tree/master/ch05



## client-v1 和 server-v1

本次实验基于 `{client, server}-v1.c` 两个程序。

### 代码

代码逻辑没什么好讲的，TCP 编程的几个流程都是固定的。

`client-v1.c` 代码如下：

```C
#include "unp.h"
int main(int argc, char *argv[])
{
    int sockfd;
    struct sockaddr_in servaddr;

    sockfd = socket(AF_INET, SOCK_STREAM, 0);

    bzero(&servaddr, sizeof(servaddr));

    servaddr.sin_family = AF_INET;
    servaddr.sin_port = htons(SERVE_PORT);
    servaddr.sin_addr.s_addr = inet_addr(SERVE_IP);

    if (connect(sockfd, (struct sockaddr *)&servaddr, sizeof(servaddr)) < 0)
        err_sys("connect error");
    str_cli(stdin, sockfd);
}
```

`server-v1.c` 代码如下：

```c
#include "unp.h"
int main()
{
    int listenfd, connfd;
    pid_t childpid;
    socklen_t clilen;
    struct sockaddr_in cliaddr, servaddr;

    listenfd = socket(AF_INET, SOCK_STREAM, 0);

    bzero(&servaddr, sizeof(servaddr));
    servaddr.sin_family = AF_INET;
    servaddr.sin_addr.s_addr = htonl(INADDR_ANY);
    servaddr.sin_port = htons(SERVE_PORT);

    bind(listenfd, (struct sockaddr *)&servaddr, sizeof(servaddr));

    listen(listenfd, LISTENQ);

    while (1)
    {
        clilen = sizeof(cliaddr);
        connfd = accept(listenfd, (struct sockaddr *)&cliaddr, &clilen);
        if ((childpid = fork()) == 0)
        {
            close(listenfd);
            str_echo(connfd);
            exit(0);
        }
        close(connfd);
    }
}
```

`str_cli` 和 `str_echo` 这 2 个函数都是在 `unp.h` 中定义的。



### 启动

运行 `server` 后，通过 `netstat -a` 查看网络状态：

```text
$ netstat -a
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address           Foreign Address         State      
tcp        0      0 *:9877                  *:*                     LISTEN
```

此时，`server` 处于 `accept` 阻塞状态。

运行一个 `client` , 再次查看网络状态：

```text
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address           Foreign Address         State   
tcp        0      0 *:9877                  *:*                     LISTEN     
tcp        0      0 localhost:9877          localhost:45004         ESTABLISHED
tcp        0      0 localhost:45004         localhost:9877          ESTABLISHED
```

可以看到，`server` 与 `client` 已经完成 3 次握手 🤝，建立 TCP 连接。

此时，有 3 个进程处于阻塞状态：

- 进入下一次等待 `accept` 的 `server` 进程；
- 在 `fgets` 上等待输入的客户进程 `client` ;
- `server` 进程 `fork` 出来的子进程，等待来自于 `connfd` 的输入。

通过命令 `ps -t pts/16 -o pid,ppid,tty,stat,args,wchan` 查看这几个进程的状态：

```text
  PID  PPID TT       STAT COMMAND     WCHAN
18394 24824 pts/16   S    ./server    inet_csk_accept
18449 24824 pts/16   S+   ./client    wait_woken
18450 18394 pts/16   S    ./server    sk_wait_data
24824 24823 pts/16   Ss   -bash       wait
```



### 终止

在 `client` 中输入一些内容，检查是否能正常工作。

```
$ ./client 
sinkinben
sinkinben
hello, world
hello, world
^D
```

通过 Ctrl+D 结束输入，终止 `client` 。

再次查看 `9877` 端口的相关连接：

```
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address           Foreign Address         State   
tcp        0      0 *:9877                  *:*                     LISTEN     
tcp        0      0 localhost:45004         localhost:9877          TIME_WAIT  
```

可以看到一个处于 `TIME-WAIT` 状态的 TCP 连接。

下面看分析一下终止的过程，以下描述中，「服务器」特指在 `server` 上 `fork` 出来与客户端通信的子进程。

1. 当客户端输入 Ctrl+D 时，`fgets` 返回一个空指针，`str_cli` 函数结束；随后 `client` 的 main 函数也结束，内核关闭当前进程的所有描述符。
2. 在关闭 `socket` 描述符之前，发送一个 FIN 到服务器，服务器 TCP 给予一个 ACK 响应。此时，服务器进入 CLOSE-WAIT 状态，客户端进入 FIN-WAIT2 状态（下图中的前 2 个箭头）。
3. 当服务器接收到 FIN 时，服务器的子进程在 `read` 函数上阻塞，接收到 FIN，`read` 函数返回 0 ，因此 `str_echo` 结束，随后子进程也通过 `exit(0)` 退出。此时，子进程的 `socket` 描述符也会被内核关闭，关闭之前，向客户发送 FIN，进入 LAST-ACK 状态（下图的第 3 个箭头）。
4. 客户端收到来自服务端的 FIN，发送 ACK 后，进入 TIME-WAIT 状态；服务端收到 ACK 后，断开 TCP 连接，进程结束（下图的第 4 个箭头）。



<img src="https://gitee.com/sinkinben/pic-go/raw/master/img/20210223150012.jpg" style="width:60%">

**但服务器的子进程真的结束了吗**？

再次查看进程状态：

```
$ ps -t pts/16 -o pid,ppid,tty,stat,args,wchan
  PID  PPID TT       STAT COMMAND                     WCHAN
18394 24824 pts/16   S    ./server                    inet_csk_accept
18450 18394 pts/16   Z    [server] <defunct>          exit
24824 24823 pts/16   Ss+  -bash                       wait_woken
```

这是，我们会发现子进程处于僵死状态 `<defunct>` ，这是因为父进程没有调用 `wait/waitpid` .

当一个子进程结束（不论是正常终止还是异常中止），内核会向父进程发送 `SIGCHILD` 信号。但是这里我们既没有调用 `wait/waitpid`，也没有捕获这个信号，所以子进程就进入 `<defunct>` 状态。

> ⚠️ **区分 2 个重要概念**
>
> - 孤儿进程：一个父进程退出，而它的一个或多个子进程还在运行，那么那些子进程将成为孤儿进程。孤儿进程将被 `init` 进程所收养，并由 `init` 进程对它们完成状态收集工作。
> - 僵死进程：一个进程使用 `fork` 创建子进程，如果子进程退出，而父进程并没有调用 `wait` 或 `waitpid` 获取子进程的状态信息，那么子进程的进程描述符仍然保存在系统中。



## server-v2: 捕获 SIGCHLD

实验程序：`server-v2.c` 和 `client-v1.c` 。

改进后的版本为 `server-v2.c` ，加入 `SIGCHLD` 的信号处理：

```c
void sigchild_handler(int signo)
{
    pid_t pid;
    int status;
    pid = wait(&status);
    printf("child pid [%d] terminated. \n", pid);
    return;
}
```

与 `client-v1.c` 一起运行，可以正常使用，不会产生僵死进程。



## client-v2: 多个客户连接

实验程序：`{server-v2, client-v2}.c` .

`client-v2.c` 的主要改动是：新建 5 个 socket，发起 5 次 connect 。代码如下：

```c
#include "unp.h"
int main(int argc, char *argv[])
{
    int i, sockfd[5];
    struct sockaddr_in servaddr;

    bzero(&servaddr, sizeof(servaddr));
    servaddr.sin_family = AF_INET;
    servaddr.sin_port = htons(SERVE_PORT);
    servaddr.sin_addr.s_addr = inet_addr(SERVE_IP);

    for (i = 0; i < 5; i++)
    {
        sockfd[i] = socket(AF_INET, SOCK_STREAM, 0);
        connect(sockfd[i], (struct sockaddr *)&servaddr, sizeof(servaddr));
    }
    str_cli(stdin, sockfd[0]);
}
```

运行结果：

```
$ ./server &
[1] 21499
$ ./client 
sss
sss
sss
sss
^D
child pid [21597] terminated. 
child pid [21596] terminated. 
child pid [21595] terminated. 
```

查看进程：

```
$ ps -t pts/16 -o pid,ppid,tty,stat,args,wchan
  PID  PPID TT       STAT COMMAND                     WCHAN
21499 24824 pts/16   S    ./server                    inet_csk_accept
21598 21499 pts/16   Z    [server] <defunct>          exit
21599 21499 pts/16   Z    [server] <defunct>          exit
24824 24823 pts/16   Ss+  -bash                       wait_woken
```

可以发现，这一版本产生了异常：有 2 个僵死进程（多试几次，数量不一样）。

为什么会这样呢？

如下图所示，客户端终止前，其 5 个 TCP 连接分别向服务端的 5 个子进程发送 FIN，子进程接收到 FIN，`read` 调用返回 0 ，`str_echo` 结束，随后调用 `exit` ，退出前向父进程发送 `SIGCHLD` 信号（一共 5 个），而这 5 个 `SIGCHLD` 信号**几乎是同一时间内发送到父进程的**。

按道理来说，信号处理程序 `sigchild_handler` 一共调用 5 次才符合我们预期的结果，但实际上并没有。这是因为 **Unix 信号是不排队的**，「不排队」的意思指的是：针对同一类型的信号，只能有一个待处理信号。例如，一个进程接受了一个 `SIGCHLD` 的信号，在执行 `SIGCHLD` 的信号处理程序的时候，来了两个 `SIGCHLD` 信号，那么只有一个 `SIGCHLD` 会成为待处理信号。

<img src="https://gitee.com/sinkinben/pic-go/raw/master/img/20210303105636.png" style="width:67%;" />



## server-v3: 改进 sigchild_handler

本次实验基于 `server-v3.c` 和 `client-v2.c` 。

关于 `wait/waitpid` 的使用可以参考 [APUE](http://www.apuebook.com/) 一书，或者[这一篇 blog](https://www.cnblogs.com/sinkinben/p/14389741.html) 。 

改进后的 `sigchild_handler` 如下：

```C
void sigchild_handler(int signo)
{
    pid_t pid;
    int status;
    while ((pid = waitpid(-1, &status, WNOHANG)) > 0)
        printf("child pid [%d] terminated. \n", pid);
    return;
}
```

运行测试结果：

```
$ ./client 
sss
sss
^D
$ child pid [28022] terminated. 
child pid [28023] terminated. 
child pid [28024] terminated. 
child pid [28025] terminated. 
child pid [28026] terminated. 
```

5 个子进程都能正常结束。



## 模拟服务器端进程终止

本次实验基于 `server-v3.c, client-v2.c` 。

1. 运行服务器和客户端，查看相关进程：

```text
sinkinben@adc-Vostro-270:~/workspace/unp$ ps -t pts/1 -o pid,ppid,tty,stat,args,wchan
  PID  PPID TT       STAT COMMAND                     WCHAN
 3377  3376 pts/1    Ss   -bash                       wait
 3740  3377 pts/1    S    ./server                    inet_csk_accept
 3782  3377 pts/1    S+   ./client                    wait_woken
 3783  3740 pts/1    S    ./server                    sk_wait_data
 3784  3740 pts/1    S    ./server                    sk_wait_data
 3785  3740 pts/1    S    ./server                    sk_wait_data
 3786  3740 pts/1    S    ./server                    sk_wait_data
 3787  3740 pts/1    S    ./server                    sk_wait_data
```


2. 关闭一个子进程: `kill 3783`，子进程向客户端会发送 FIN，（随后应当会接收来自客户端的 ACK，即完成 TCP 四次挥手的前 2 次），然后子进程正式结束。

3. 运行 `server` 的终端会输出：

```text
child pid [3783] terminated.
```

4. 查看各个 TCP 连接的状态：

```text
Proto Recv-Q Send-Q Local Address           Foreign Address         State
tcp        0      0 *:9877                  *:*                     LISTEN     
tcp        0      0 localhost:59852         localhost:9877          ESTABLISHED
tcp        0      0 localhost:59858         localhost:9877          ESTABLISHED
tcp        0      0 localhost:59856         localhost:9877          ESTABLISHED
tcp        0      0 localhost:9877          localhost:59856         ESTABLISHED
tcp        0      0 localhost:9877          localhost:59852         ESTABLISHED
tcp        1      0 localhost:59850         localhost:9877          CLOSE_WAIT 
tcp        0      0 localhost:9877          localhost:59854         ESTABLISHED
tcp        0      0 localhost:9877          localhost:59858         ESTABLISHED
tcp        0      0 localhost:59854         localhost:9877          ESTABLISHED
```

可以发现，服务器子进程结束之后，（重点看第 8 行）客户端还存在着一个单向的 TCP 连接 `localhost:59850 -> localhost:9877` ，其状态处于 `CLOSE-WAIT` 。

理论上，处于 `CLOSE-WAIT` 状态的 TCP，**应当是能够单向发送数据的**。但这里情况比较特殊：TCP 另一端的子进程已经被 `kill` ，但客户端还不知道，这时候，客户端继续发送数据会怎么样呢？



5. 回到运行 `client` 的终端，尝试继续输入一些内容：

```text
$ ./client 
sss
sss
child pid [3783] terminated.            // kill 3783
ssss                                    // new input
str_cli: server terminated prematurely  // crash
child pid [3784] terminated. 
child pid [3786] terminated. 
child pid [3785] terminated. 
child pid [3787] terminated. 
```

`server terminated prematurely` 这一字符串是在 `str_cli` 中的 `if` 分支输出的（参考 `unp.h` 的相关）。

那么，发生这种情况的原因是什么呢？我们结合上述过程来分析一下 `str_cli` 的代码：

```c
void str_cli(FILE *fp, int sockfd)
{
    char sendline[MAXLINE], recvline[MAXLINE];
    while (fgets(sendline, MAXLINE, fp) != NULL)
    {
        write(sockfd, sendline, strlen(sendline));
        if (Readline(sockfd, recvline, MAXLINE) == 0)
            err_quit("str_cli: server terminated prematurely");
        fputs(recvline, stdout);
    }
}
```

当 `kill 3783` 执行时，`client` 进程阻塞于 `fgets` ，**服务端发送过来的 FIN 还没读取到**。回到上面的第 4 步看一下，`client` 的 TCP 连接的 `Recv-Q = 1`，其实就是指这个 FIN 。

当输入 `ssss` 按下回车键后，服务端和客户端的情况如下：

- 客户端：调用 `write` 发数据发送到服务器的 `sockfd` ，之后调用 `Readline -> readline -> read` 会读取到 FIN ，然后 `read` 返回 0 ，最后执行 `err_quit("str_cli: server terminated prematurely")` 这一行代码。

- 服务端：打开该 `sockfd` 的子进程已经终止，于是响应一个 RST，但客户端「看不到」这个 RTS 。这个「看不到」可能有 2 种情况：一是 RTS 到达前客户端已经 `err_quit`；二是子进程调用 `err_quit` 前，RTS 已到达，但是没有通过 `read` 读取。

上面的致命问题是：当 FIN 到达 `sockfd` 时，`client` 进程阻塞于标准输入 `fgets` 上，不能及时处理这一个 FIN。

从这一场景可以看出，目前的服务器-客户端模型存在这么一个问题：客户端同时存在 socket 和 stdin 两种 I/O ，但是它仅仅是「运行到哪就读取哪」，不能及时处理另外一个 I/O 所输入的信息（如上面所述的情况）。因此，需要所谓的 I/O 复用 (I/O Multiplexing)，这也许是下一篇博客的内容了。



## 客户端连续发送 2 次数据

```text
sinkinben@adc-Vostro-270:~/workspace/unp/ch05$ ps -t pts/2 -o pid,ppid,tty,stat,args,wchan                                                                    
  PID  PPID TT       STAT COMMAND                     WCHAN
22162 22161 pts/2    Ss   -bash                       wait
22248 22162 pts/2    S    ./server                    inet_csk_accept
22249 22162 pts/2    S+   ./client                    wait_woken
22250 22248 pts/2    S    ./server                    sk_wait_data
22251 22248 pts/2    S    ./server                    sk_wait_data
22252 22248 pts/2    S    ./server                    sk_wait_data
22253 22248 pts/2    S    ./server                    sk_wait_data
22254 22248 pts/2    S    ./server                    sk_wait_data
```

执行 `kill 22250`

```
sinkinben@adc-Vostro-270:~/workspace/unp/ch05$ ./client 
hi
hi
child pid [22250] terminated. 

```

