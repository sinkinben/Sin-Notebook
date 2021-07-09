## 多线程 Demo 实现

本文基于 C 标准库提供的网络通信 API，使用 TCP ，实现一个简单的多线程服务器 Demo 。

首先要看 API，这是一项十分无聊的工作，我看的头都晕了 🤒️ 。

## API

### 字节序转换

函数原型：

```c
#include <arpa/inet.h>
uint64_t htonll(uint64_t hostlonglong);
uint32_t htonl(uint32_t hostlong);
uint16_t htons(uint16_t hostshort);
uint64_t ntohll(uint64_t netlonglong);
uint32_t ntohl(uint32_t netlong);
uint16_t ntohs(uint16_t netshort);
```

`h` 表示 host, `n` 表示 network，这些函数的作用是把主机的字节序转换为网络的字节序（即小端到大端的转变）。

例如：

```c
#include <arpa/inet.h>
#include <stdio.h>
int main()
{
    uint32_t host = 0x01020304;     // high->low: 01 02 03 04
    uint32_t network = htonl(host); // high->low: 04 03 02 01
    printf("%p\n", network);     // 0x4030201
}
```

### socket

函数原型：

```c
#include <sys/socket.h>
int socket(int domain, int type, int protocol);
```

建立一个协议族为 `domain`, 协议类型为 `type`, 协议编号为 `protocol` 的套接字文件描述符。如果函数调用成功，会返回一个标识这个套接字的文件描述符，失败的时候返回-1。



`domain` 的取值：

```text
Name                Purpose                          Man page
AF_UNIX, AF_LOCAL   Local communication              unix(7)
AF_INET             IPv4 Internet protocols          ip(7)
AF_INET6            IPv6 Internet protocols          ipv6(7)
AF_IPX              IPX - Novell protocols
AF_NETLINK          Kernel user interface device     netlink(7)
AF_X25              ITU-T X.25 / ISO-8208 protocol   x25(7)
AF_AX25             Amateur radio AX.25 protocol
AF_ATMPVC           Access to raw ATM PVCs
AF_APPLETALK        AppleTalk                        ddp(7)
AF_PACKET           Low level packet interface       packet(7)
AF_ALG              Interface to kernel crypto API
```

`AF` 是 Address Family 的缩写，`INET` 是 Internet 的缩写。某些地方可能会使用 `PF`，即 Protocol Family，应该是同一个东西。

`type` 的取值：

```text
SOCK_STREAM     Provides sequenced, reliable, two-way, connection-based byte streams.  An out-of-band data transmission mechanism may be supported.

SOCK_DGRAM      Supports datagrams (connectionless, unreliable messages of a fixed maximum length).

SOCK_SEQPACKET  Provides a sequenced, reliable, two-way connection-based data transmission path for datagrams of fixed maximum length; a consumer is required to read an entire packet with each input system call.

SOCK_RAW        Provides raw network protocol access.

SOCK_RDM        Provides a reliable datagram layer that does not guarantee ordering.

SOCK_PACKET     Obsolete and should not be used in new programs; see packet(7).
```

`type` 常用的是 `STREAM` 和 `DGRAM` ，根据描述，可以确定前者对应 TCP，而后者对应 UDP ：

- `SOCK_STREAM` 套接字表示一个双向的字节流，与管道类似。流式的套接字在进行数据收发之前必须已经连接，连接使用 `connect()` 函数进行。一旦连接，可以使用 `read()` 或者 `write()` 函数进行数据的传输，流式通信方式保证数据不会丢失或者重复接收。
- `SOCK_DGRAM` 和 `SOCK_RAW` 这个两种套接字可以使用函数 `sendto()` 来发送数据，使用 `recvfrom()` 函数接受数据，`recvfrom()` 接受来自制定IP地址的发送方的数据。

对于第 3 个参数 `protocal`，用于**指定某个协议的特定类型**，即 `type` 类型中的某个类型。通常某协议中只有一种特定类型，这 样`protocol` 参数仅能设置为 0 ；但是有些协议有多种特定的类型，就需要设置这个参数来选择特定的类型。



### bind

函数原型：

```c
int bind(int sockfd, const struct sockaddr *addr, socklen_t addrlen);
```

如果函数执行成功，返回值为 0，否则为 `SOCKET_ERROR` 。

参数：

- `sockfd` 是一个有效的 socket 描述符（函数 `socket()` 的有效返回值）。
- `addrlen` 是第二个参数 `addr` 结构体的长度。
- `addr` 是一个 `sockaddr` 结构体指针，包含 IP 和端口等信息。

`sockaddr` 的结构如下：

```c
struct sockaddr {
    sa_family_t sa_family;
    char sa_data[14];
};
// sa_familt_t 是无符号整型，Ubuntu 下是 unsigned short int
```

`sockaddr` 的存在是为了统一地址结构的表示方法 ，统一接口函数，使得不同的地址结构可以被 `bind(), connect(), recvfrom(), sendto()` 等函数调用。但一般的编程中并不直接对此数据结构进行操作，而使用另一个与之等价的数据结构 `sockaddr_in` :

```c
struct sockaddr_in {
    short int sin_family;        /* Address family */
    unsigned short int sin_port; /* Port number */
    struct in_addr sin_addr;     /* Internet address */
    unsigned char sin_zero[8];   /* Same size as struct sockaddr */
};
```

各字段解析：

- `sin_family` ：指代协议族，在 socket 编程中有 3 个取值 `AF_INET, AF_INET6, AF_UNSPEC` .
- `sin_port` ：存储端口号（使用网络字节顺序）
- `sin_addr` ：存储IP地址，使用 `in_addr` 这个数据结构
- `sin_zero` ：是为了让 `sockaddr` 与 `sockaddr_in` 两个数据结构保持大小相同而保留的空字节。

`in_addr` 的结构如下：

```c
typedef uint32_t in_addr_t;
struct in_addr{
    in_addr_t s_addr;
};
```

太阴间了。



### listen

```c
int listen(int sockfd, int backlog);
```

返回值：无错误，返回 0，否则 -1 。

作用：`listen` 函数使用主动连接套接字变为被连接套接口，使得一个进程可以接受其它进程的请求，从而成为一个服务器进程。在 TCP 服务器编程中 `listen` 函数把进程变为一个服务器，并指定相应的套接字变为被动连接。

`listen` 函数一般在调用 `bind` 之后，调用 `accept` 之前调用。

`backlog` 参数指定连接请求队列的最大个数。



### accept

```c
int accept(int sockfd, struct sockaddr *addr, socklen_t *addrlen);
```

接受连接请求，成功返回一个新的套接字描述符 `newfd` ，失败返回-1。返回值 `newfd` 与参数 `sockfd` 是不同的，`newfd` 专门用于与客户端的通信，而 `sockfd` 是专门用于 `listen` 的 socket 。

`addr` 和 `addrlen` 都是指针，用于接收来自客户端的 `addr` 的信息。



### inet_addr

函数原型：

```c
in_addr_t inet_addr(const char *cp);
```

将一个点分十进制的 IP 字符串转换为网络字节序的 `uint32_t` 。

**例子**

```c
int main()
{
    const char *ip = "127.0.0.1";  // 7f.00.00.01
    printf("%p\n", inet_addr(ip)); // 0x0100007f
}
```



### send

```cpp
#include <sys/types.h>
#include <sys/socket.h>
ssize_t send(int sockfd, const void *buf, size_t len, int flags);
ssize_t sendto(int sockfd, const void *buf, size_t len, int flags,
               const struct sockaddr *dest_addr, socklen_t addrlen);
ssize_t sendmsg(int sockfd, const struct msghdr *msg, int flags);
```

其中 `send(fd, buf, len, flags)` 与 `sendto(fd, buf, len, flags, NULL, 0)` 等价。



### recv

```c
#include <sys/types.h>
#include <sys/socket.h>
ssize_t recv(int sockfd, void *buf, size_t len, int flags);
ssize_t recvfrom(int sockfd, void *buf, size_t len, int flags,
                 struct sockaddr *src_addr, socklen_t *addrlen);
ssize_t recvmsg(int sockfd, struct msghdr *msg, int flags);
```

其中 `recv(fd, buf, len, flags)` 与 `recvfrom(fd, buf, len, flags, NULL, 0)` 等价。



### connect

```c
int connect(int sockfd, const struct sockaddr *addr, socklen_t addrlen);
```

成功返回 0 ，失败返回 -1 。

`sockfd` 是客户端进程创建的，用于与服务端通信的 socket ; `addr` 是目标服务器的 IP 地址和端口。



## 多线程服务器

本次实现的场景如下：

- 客户端可以具有多个，客户端主动连接服务器，允许每个客户端发送 `msg` 到服务器，并接受来自服务器的信息。
- 服务端对于每个申请连接到客户端，创建一个线程处理请求。对于客户端发送过来的 `msg`，然后服务器把 `msg` 加上一些其他字符串，发送回客户端。

上面讲了这么多的 API，但其实基于 TCP 协议的网络编程的主要流程都是固定的:

```text
    [Server]         [Client]
     socket           socket
       |                |
      bind              |
       |                | 
     listen             |
       |                |
     accept  <------  connect
       |                |
      read   <------   write
       |                |
   (handle req)         |
       |                |
     write   ------>   read
       |                |
     close             close
```



### server

```c
#include <stdlib.h>
#include <stdio.h>
#include <sys/socket.h>
#include <sys/types.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include <pthread.h>
#include <string.h>
#include <unistd.h>
#define PORT 8887
#define QUEUE 10
const char *pattern = "Hello, I am the server. Your msg is received, which is: %s";

typedef struct
{
    struct sockaddr_in addr;
    socklen_t addr_len;
    int connectfd;
} thread_args;

void *handle_thread(void *arg)
{
    thread_args *targs = (thread_args *)arg;
    pthread_t tid = pthread_self();
    printf("tid = %u and socket = %d\n", tid, targs->connectfd);
    char send_buf[BUFSIZ] = {0}, recv_buf[BUFSIZ] = {0};
    while (1)
    {
        int len = recv(targs->connectfd, recv_buf, BUFSIZ, 0);
        printf("[Client %d] %s", targs->connectfd, recv_buf);
        
        if (strcmp("q\n", recv_buf) == 0)
            break;
        
        sprintf(send_buf, pattern, recv_buf);
        send(targs->connectfd, send_buf, strlen(send_buf), 0);

        memset(send_buf, 0, BUFSIZ), memset(recv_buf, 0, BUFSIZ);
    }
    close(targs->connectfd);
    free(targs);
    pthread_exit(NULL);
}

int main()
{
    int listenfd = socket(AF_INET, SOCK_STREAM, 0);
    printf("server is listening at socket fd = %d\n", listenfd);
    struct sockaddr_in addr;
    addr.sin_family = AF_INET;
    addr.sin_port = htons(PORT);
    addr.sin_addr.s_addr = htonl(INADDR_ANY);

    if (bind(listenfd, (struct sockaddr *)&addr, sizeof(addr)) == -1)
    {
        perror("bind error\n");
        exit(-1);
    }

    if (listen(listenfd, QUEUE) == -1)
    {
        perror("listen error\n");
        exit(-1);
    }

    while (1)
    {
        thread_args *targs = malloc(sizeof(thread_args));
        targs->connectfd = accept(listenfd, (struct sockaddr *)&targs->addr, &targs->addr_len);
        // int newfd = accept(sockfd, NULL, NULL);
        pthread_t tid;
        pthread_create(&tid, NULL, handle_thread, (void *)targs);
        pthread_detach(tid);
    }
    close(listenfd);
}
```



### client

```c
#include <stdio.h>
#include <sys/socket.h>
#include <sys/types.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>

#define PORT 8887
const char *target_ip = "127.0.0.1";

int main()
{
    int sockfd = socket(AF_INET, SOCK_STREAM, 0);
    printf("client socket = %d\n", sockfd);
    struct sockaddr_in addr;
    addr.sin_family = AF_INET;
    addr.sin_port = htons(PORT);
    addr.sin_addr.s_addr = inet_addr(target_ip);

    if (connect(sockfd, (struct sockaddr *)&addr, sizeof(struct sockaddr_in)) < 0)
    {
        perror("connect error\n");
        exit(-1);
    }

    char send_buf[BUFSIZ], recv_buf[BUFSIZ];
    while (fgets(send_buf, BUFSIZ, stdin) != NULL)
    {
        if (strcmp(send_buf, "q\n") == 0)
            break;

        send(sockfd, send_buf, strlen(send_buf), 0);
        printf("[Client] %s\n", send_buf);

        recv(sockfd, recv_buf, BUFSIZ, 0);
        printf("[Server] %s\n", recv_buf);

        memset(send_buf, 0, BUFSIZ), memset(recv_buf, 0, BUFSIZ);
    }
    close(sockfd);
    exit(0);
}
```



### 运行结果

编译：

```
gcc server.c -o server -lpthread
gcc client.c -o client
```

先运行 `server`，后运行多个 `client` .

<img src="https://gitee.com/sinkinben/pic-go/raw/master/img/20210224125958.png" style="width:95%;" />

需要注意的是，这里的服务器，客户端都是运行在同一机器上的，所以客户端使用的目标 IP 是 127.0.0.1 ，如果想进一步更全面地测试，应该把服务端运行在一个云服务器上，然后开放 8887 端口，再进行测试。