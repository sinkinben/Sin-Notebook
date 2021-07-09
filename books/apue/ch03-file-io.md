## [APUE] 文件 I/O

文件操作相关 API：open, read, write, lseek, close.

多进程共享文件的相关 API：dup, dup2, fcntl, sync, fsync, ioctl. 



## 文件操作 API

### open and openat

函数原型：

```cpp
#include <fcntl.h>
int open(const char *path, int oflag, ... /* mode_t mode */ );
int openat(int fd, const char *path, int oflag, ... /* mode_t mode */ );
// Both return: file descriptor if OK, −1 on error
```

`oflag` 是下列选项的组合（通过或运算 `|` ）：

+ 必选且只能选择一个：`O_RDONLY, O_WRONLY, O_RDWR`
+ 可选项
  + `O_APPEND`: 写文件时追加到尾端。
  + `O_CLOEXEC`
  + `O_CREAT`: 文件不存在时创建；若使用该选项，需要 `mode` 参数，指定文件到访问权限。
  + `O_DIRECTORY`: 如果 `path` 不是目录，出错。
  + `O_EXCL`: 如果同时指定了 `O_CREAT`，而文件已存在，则出错。
  + `O_NOCITY`
  + `O_NOFOLLOW`: 如果 `path` 是一个符号链接，则出错。
  + `O_NOBLOCK`：如果 `path` 是一个 FIFO、一个块特殊文件或一个字符特殊文件，则为本次打开操作和后续的I/O操作设置非阻塞模式 (Nonblocking Mode) .
  + `O_SYNC`: 使每次 `write` 操作等待物理 I/O 完成，包括由该 `write` 操作引起的文件属性的更新所需要的 I/O 。
  + `O_DSYNC`
  + `O_RSYNC`
  + `O_TRUNC`：如果文件存在，且打开模式**可写**，那么长度截断为 0 。
  + `O_TTYINIT`

`mode` 参数可设定文件的权限，取值及其含义如下图所示：

<img src="https://gitee.com/sinkinben/pic-go/raw/master/img/20210120174943.png" style="width:400px"/>

`openat` 的 `fd` 参数可以传入一个目录的 `fd`:

```c
int dir = open(".", O_RDONLY | O_DIRECTORY);
int fd = openat(dir, "test.c", O_RDONLY);
```

`openat` 的引入是为了解决 2 个问题：

- 让线程可以使用**相对路径**打开目录中的文件。因为同一进程的所有线程共享当前的工作目录，因此很难让不同的线程在同一时间工作在不同的目录。
- 避免 `TOCTTOU` 问题。

`TOCTTOU` 指的是 Time Of Check To Time Of Use, 其基本 idea 是：如果有 2 个基于文件的函数调用，其中第二个调用依赖于第一个调用的结果，那么这个程序是脆弱的 (Vulnerable). 因为如果这 2 个操作不是原子操作，在 2 次调用之间文件内容可能被改变。

### creat

函数原型：

```c
int creat(const char *path, mode_t mode);
// Returns: file descriptor opened for write-only if OK, −1 on error
```

`creat` 函数相当于：`open(path, O_WRONLY | O_CREAT | O_TRUNC, mode);` 

`mode` 表示文件的访问权限，将在后续章节解析。

### close

函数原型：

```c
int close(int fd);
// Returns: 0 if OK, −1 on error
```

关闭文件，释放该进程加在该文件上的所有记录锁。

进程结束时，内核会自动关闭所有它打开的文件，所以 `close` 有时候可有可无。

### lseek

```c
off_t lseek(int fd, off_t offset, int whence);
// Returns: new file offset if OK, −1 on error
```

对于 `offset` 参数的解析，取决于 `whence` 的值：

- `SEEK_SET`：该文件的偏移量设置为距文件开始处的 `offset` 个字节
- `SEEK_CUR`：该文件的偏移量设置为**当前位置**加上 `offset` 的值，这时候 `offset` 可为负数。
- `SEEK_END`：该文件的偏移量设置为**文件长度**加上 `offset` 的值，这时候 `offset` 可为负数。

如果 lseek 成功执行，返回新的文件偏移量，否则返回 -1 。如果 `fd` 指向的是一个 FIFO、管道或者 socket，lseek 返回 -1，并把 `errno` 设置为 `ESPIPE (Illegal Seek)` . lseek 不引起任何 IO 操作，仅仅把当前偏移量记录在内核当中，用于下一次的读写操作。

**例子1**

```c
int main()
{
    if (lseek(STDIN_FILENO, 0, SEEK_CUR) != -1) puts("can seek");
    else puts("can not seek");
}
```

运行结果：

```
$ ./a.out < /etc/passwd
can seek
$ cat /etc/passwd | ./a.out 
can not seek
```

`<` 符号的作用是重定向输入。

一般情况下，当前偏移量应当为非负数，但某些设备（Linux中一切皆文件）允许它为负数。此外，偏移量可以大于文件长度，这种情况下，对文件的下一次写操作将「加长」文件，在文件中形成一个「空洞」（字节均值为 0 ），**空洞不一定会占据磁盘空间**，具体取决于文件系统的实现。

**例子2：空洞文件**

```c
char buf1[] = "abcdefghij";
char buf2[] = "ABCDEFGHIJ";
int main()
{
    int fd = -1;
    if ((fd = creat("file.hole", FILE_MODE)) == -1) err_sys("creat error");
    if (write(fd, buf1, 10) != 10)                  err_sys("write error");
    if (lseek(fd, 16384, SEEK_SET) == -1)           err_sys("lseek error");
    if (write(fd, buf2, 10) != 10)                  err_sys("write2 error");
    // now offset is at 16394
}
```

运行结果：

```
$ ll file.hole 
-rw-r--r-- 1 sinkinben sinkinben 16394 1月  20 15:20 file.hole
$ od -c file.hole 
0000000   a   b   c   d   e   f   g   h   i   j  \0  \0  \0  \0  \0  \0
0000020  \0  \0  \0  \0  \0  \0  \0  \0  \0  \0  \0  \0  \0  \0  \0  \0
*
0040000   A   B   C   D   E   F   G   H   I   J
0040012
```

`od -c` 以八进制输出文件内容，`hex(16394) = 0x400a` . 

创建一个同样长度但没有空洞的文件 `file.nohole`：

```
$ ls -sl file.*
 8 -rw-r--r-- 1 sinkinben sinkinben 16394 1月  20 15:31 file.hole
20 -rw-r--r-- 1 sinkinben sinkinben 16394 1月  20 15:31 file.nohole
```

可以看出，`file.nohole` 占据了 20 个磁盘块。

### read

函数原型：

```c
ssize_t read(int fd, void *buf, size_t nbytes);
// Returns: number of bytes read, 0 if end of file, −1 on error
```

可能存在返回值（实际读到的字节数）小于要求读取的字节数 `nbytes` 的情况：

- 读取普通文件：当前 offset 离文件末端只有 30 字节，而要求读取 100 字节。
- 读取终端设备：通常一次最多读取一行。
- 从网络 socket 中读取：网络的缓冲机制可能造成上述情况。
- 从管道或者 FIFO 读取：与读取普通文件类似，剩下的字节数不足。
- 当某一信号造成读取中断。

read 操作一般都会采用预读机制 (Read Ahead) 提高性能，预读的数据放入到 Cache 当中，那么下一次读取就不用读取磁盘。

### write

函数原型：

```c
ssize_t write(int fd, const void *buf, size_t nbytes);
// Returns: number of bytes written if OK, −1 on error
```

与 read 操作类似。返回值通常与 `nbytes` 相等，否则表示出错。出错的原因可能为磁盘已满，或者超过一个进程的文件长度限制。

## 文件共享

在 Unix 系统中，内核为每个进程都建立了一个**文件描述符表 (即下图的 Process Table Entry, 名字是我自己翻译的)**，进程打开某个文件都过程如下图所示。

进程的每一个 `fd` 都有对应的文件指针 (File Pointer) 指向某一个文件表项 (File Table Entry) ，该表项包括当前打开文件的状态信息和一个 `v-node` 指针。其中 `v-node` 包含了文件的类型和操作该文件的函数指针等信息，还包括一个指向文件 `inode` 的指针。

<img src="https://gitee.com/sinkinben/pic-go/raw/master/img/20210120155351.png"  style="width:80%;" />

如下图所示，如果 2 个进程同时打开了同一个文件，那么这 2 个 File Table Entry 的 `v-node` 指针将会指向同一个 `v-node` 。由图中的过程可以看出，不同进程打开同一文件，每个进程对文件的偏移量是独立的，文件的状态信息 (File Status Flags) 也是独立的。

<img src="https://gitee.com/sinkinben/pic-go/raw/master/img/20210120155959.png" style="width:80%;" />

基于这个过程，可以对上述对一些 IO 操作的特征进行解析：

- 完成一次 write 操作后，File Table Entry 中的 offset 将会增加写入的字节数。如果当前的 offset 超过了 i-node 中的文件大小 (current file size) ，那么就将 current file size 设置为当前的 offset 。
- 使用 `O_APPEND` 打开一个文件，File Table Entry 中的 file status flags 会记录这个 `O_APPEND` 。每次 write 操作执行时，首先会把 current file offset 设置为 i-node 中的 current file size。
- 若使用 lseek 定位到文件末端，则会把 offset 设置为 file size 。
- lseek 只修改 File Table Entry 中的 offset，不进行任何 IO 操作。

如果进程进行了 `fork` 操作，那么 Process Table Entry 中的文件描述符表也会被子进程拷贝，所以也有可能有多个 File Pointer 指向同一个 File Table Entry 。类似，`dup` 操作也会使得**同一进程**中的 2 个不同的 `fd` 指向同一个 File Table Entry。

## 原子操作

在多进程场景下，需要对同一个日志文件进行写操作，那么就有可能会出现**进程 A 的内容被进程 B 的内容覆盖**的情况（因为文件偏移量是独立的）。

因此，写操作需要实现为一个原子操作（要么全做，要么全不做），才能满足上述场景的要求。

### pread and pwrite

函数原型：

```C
ssize_t pread(int fd, void *buf, size_t nbytes, off_t offset);
// Returns: number of bytes read, 0 if end of file, −1 on error
ssize_t pwrite(int fd, const void *buf, size_t nbytes, off_t offset); 
// Returns: number of bytes written if OK, −1 on error
```

作用：从离文件开始处的 `offset` 位置开始，读取 `nbytes` 个字节。

 `pread` 的行为相当于调用 `lseek` 后再次调用 `read` ，但 `pread` 是一个原子操作，这意味着：

- 调用 `pread` 过程中，无法中断其定位 lseek 和 read 操作。
- 不更新当前的文件偏移量。

`pwrite` 与之类似。



## dup and dup2

```c
int dup(int fd);
int dup2(int fd, int fd2);
// Both return: new file descriptor if OK, −1 on error
```

作用：把 `fd` 复制为一个新的描述符。如果传入的 `fd` 无效，那么返回 -1 。

`dup` 返回的总是可用对文件描述符中的最小值（也就是从 3 开始）。

对于 `dup2` 的 `fd2` 参数，用于指定新描述符的值，如果 `fd2` 已经打开，会先关闭它：

- `fd1 == fd2` : 返回 `fd2`，且不关闭它。
- 如果 `fd1` 无效，那么返回 -1.
- 如果 `fd1` 有效，那么把 `fd1` 复制为 `fd2` ，返回 `fd2` 。

如下图所示，经过 `dup` 操作后，会有多个文件指针指向同一个 File Table Entry。

<img src="https://gitee.com/sinkinben/pic-go/raw/master/img/20210120165102.png"  style="width:80%;" />

## sync, fsync and fdatasync

现在的计算机通常都会有 Cache，为了提高 IO 性能，除了在 `read` 一小节提到的预读机制外，还有延迟写机制 (Delayed Write) 。当我们向文件写数据时，首先会拷贝到高速缓冲区当中，后面再把高速缓冲区中的数据写到磁盘上（通过排队 FIFO 的顺序）。

在某些场景下，我们需要缓冲区的数据和磁盘的数据保持一致。因此需要 `sync, fsync, fdatasync` 这三个函数。

函数原型：

```c
int fsync(int fd); 
int fdatasync(int fd);
// Returns: 0 if OK, −1 on error
void sync(void);
```

`sync` 的作用：

- The `sync` function simply **queues all the modified block buffers for writing and returns**; it does not wait for the disk writes to take place. （不等待磁盘操作完成）

- `sync` is normally called periodically (usually every 30 seconds) from a system daemon, often called `update`. The command `sync` also calls the sync function.（`sync` 通常由系统的一个守护进程 `update` 来周期性调用，命令 `sync` 也会调用这个函数。）

`fsync` 只对 `fd` 这一个文件实现同步操作，并且等待磁盘 IO 的完成才返回。

`fdatasync` 与 `fsync` 类似，但它**只更新文件的数据**，而 `fsync` 还会更新文件的属性（包括权限信息等）。

## fcntl and ioctl

函数原型：

```c
int fcntl(int fd, int cmd, ... /* int arg */ );
// Returns: depends on cmd if OK (see following), −1 on error
int ioctl(int fd, int request, ...);
// Returns: −1 on error, something else if OK
```

`fcntl` 可以改变文件 `fd` 的属性信息。`ioctl` 一般用于外部设备（比如实现驱动程序） 的 IO 操作。



## 总结

APUE 看得好无聊，看着看着就想睡觉。