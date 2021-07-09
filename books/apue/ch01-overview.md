## [APUE] UNIX 基础知识



## 登录

可以在 `/etc/passwd` 中查看登录名：

```
sinkinben:x:1001:1001:sinkinben,,,:/home/sinkinben:/bin/bash
```

由 6 个冒号分割成 7 个字段组成，依次为：登录名 (Login Name)，加密口令 (Encrypted Password)，用户 ID (Numeric User ID), 用户组 ID (Numeric User Group ID), 注释字段 (Comment Field), Home 目录路径 (Home Directory), Shell 程序 (Shell Program) .

## 文件和目录

一个 `ls` 程序：

```c
#include <dirent.h>
#include <stdio.h>
int main(int argc, char *argv[])
{
    DIR *dp;
    struct dirent *dirp;
    if (argc != 2) err_quit("usage: ls directory_name");
    if ((dp = opendir(argv[1])) == NULL) err_sys("can't open %s", argv[1]);
    while ((dirp = readdir(dp)) != NULL) printf("%s\n", dirp->d_name);
    closedir(dp);
    return 0;
}
```

## 输入和输出

文件描述符 (File Descriptor) 是一个非负整数，内核用来标识进程访问的文件。当内核打开或创建一个文件，读写操作都可以通过文件描述符来完成。

三个特殊的文件描述符：

+ `STDIN_FILENO(0)`： 标准输入
+ `STDOUT_FILENO(1)`：标准输出
+ `STDERR_FILENO(2)`：标准错误输出

文件描述符从小到大分配（从 3 开始），不同进程打开相同的文件，其返回的文件描述符可以不同。对于 `fork` 产生的子进程，会自动继承父进程的所有文件描述符。

`mycat` 程序代码：

```c
#include "../apue.h"
#include <unistd.h>
#define BUFFSIZE 4096
int main(void)
{
    int n;
    char buf[BUFFSIZE];
    while ((n = read(STDIN_FILENO, buf, BUFFSIZE)) > 0)
        if (write(STDOUT_FILENO, buf, n) != n)
            err_sys("write error");
    if (n < 0)
        err_sys("read error");
    return 0;
}
```

 `getcputc` 代码：

```c
#include "../apue.h"
#include <stdio.h>
int main()
{
    int c;
    while ((c = getc(stdin)) != EOF)
        if (putc(c, stdout) ==  EOF)
            err_sys("putc err");
    if (ferror(stdin))
        err_sys("getc err");
}
```

## 用户标识

**User ID**

`/ect/passwd` 文件中的 `User ID` 用于向系统标识各个不同的用户，系统在确定用户登录名的同时（即新建用户时），确定其用户 ID ，用户不能更改其用户 ID。

User ID 为 0 的用户是 `root` ，或称为 Superuser。

**Gruop ID**

Group ID 也是系统在新建一个用户时分配的。一般来说，在 `/etc/passwd` 中，多个登录项会具有相同的 Group ID 。这种权限设置允许组内成员访问某一资源（如文件），而组外成员则不能。

上述两个信息项可以通过 `getuid()` 和 `getgid()` 获得，需要 `unistd.h`。

## 信号 Signal

信号 (Signal) 用于通知进程发生了某种情况。例如，发生除数为 0  的情况，则将 `SIGFPE` 信号发送给该进程。进程有 3 种信号处理方式：

- 忽略信号。
- 按系统默认方式处理。对于 `SIGFPE` ，系统默认终止进程。
- 提供一个函数，信号事件发生时调用该函数，这种操作称为捕获信号 (Catching Signal)。

常见信号：

- `Ctrl + C` or `Ctrl + \` 
- 调用 `kill()` 函数，要求调用者是被 kill 进程的所有者。

Catching Signal 的使用例子：