## [Linux] 动态链接库和静态链接库

二者的区别：

- 动态链接库只有在另外一个模块调用其所包含的函数时才被启动，即**在运行时加载到进程空间**。
- 静态链接库**在编译期通过链接 (Link) 的方式装入程序的进程空间**。

文件格式：

- 动态链接库：Linux 下，常见的是 `.so` ；Windows 下，常见的是 `.dll` 。
- 静态链接库：Linux 下，常见的是 `.a` 。

下面来看如何编写一个静态/动态链接库。



## 静态链接库的编写

假设有 `say.c` 文件如下：

```c
#include <stdio.h>
void say(const char *str) { printf("%s\n", str); }
```

编译：

```bash
# say.c -> say.o
gcc -c say.c
# say.o -> libsay.a
ar rcs libsay.a say.o
```

需要注意的是，Linux 环境下，库的名称必须为 `libxxx.a` 。

在 `main.c` 中使用：

```C
// 需要声明, 相当于头文件的作用
void say(const char *);
int main()
{
    say("hello, sinkinben!");
}
```

编译：

```sh
# main.c -> main.o
gcc -c main.c 
# link: main.o -> a.out
gcc main.o -L . -lsay
```

其中，`-L` 参数指定链接库的位置，`-lsay` 类似于 `-lpthread` ，把上述的 `libsay.a` 与 `main.o` 链接。

执行：

```text
$ ./a.out 
hello, sinkinben!
```





## 动态链接库的编写

同样使用上述的 `say.c` 和 `main.c` 。

编译成动态链接库：

```shell
# say.c -> libsay.so
gcc -shared -fPIC say.c -o libsay.so
# main.c -> main.o
gcc -c main.c -o main.o
# main.o -> a.out
gcc main.o -L . -lsay
```

运行 `a.out`，但意外发生了：

```text
./a.out: error while loading shared libraries: libsay.so: cannot open shared object file: No such file or directory
```

显然是因为程序没找到 `libsay.so` 的位置，可以通过环境变量去设置：

```shell
export LD_LIBRARY_PATH="./:$LD_LIBRARY_PATH"
```

然后就可以执行 `a.out` 。

但是为什么会这样呢？我们在编译的时候不是通过 `-L .` 去告知编译器我们的 `libsay.so` 在当前目录下吗？

对于 `a.out`，可以通过 `ldd a.out` 命令查看它的动态链接库的情况：

```text
$ ldd a.out 
        linux-vdso.so.1 =>  (0x00007fff109f5000)
        libsay.so => ./libsay.so (0x00007fca740ec000)
        libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007fca73d22000)
        /lib64/ld-linux-x86-64.so.2 (0x00007fca742ee000)
```

后面的地址显然是该 Lib 在进程空间中的虚拟地址；可以发现 `libsay.so` 指向的位置是 `./libsay.so` ，到这一步，还算是符合我们的预期。

但在实际运行时， Linux 环境下的共享库的寻找和加载是由 `/lib/ld.so` 实现的。 `ld.so` 会在标准路经 `/lib, /usr/lib` 中寻找应用程序用到的共享库。

也就是说，编译时提供的路径仅仅用于生成可执行文件，但真正查找动态链接库的时候，`ld.so` 只会在某些路径中查找，我们加入的环境变量 `LD_LIBRARY_PATH` 实际上是让 `ld.so` 也在该路径下查找动态链接库。

但据说通过环境变量去解决这一问题是不太好的，可以参考 [Why LD_LIBRARY_PATH is bad](http://xahlee.info/UnixResource_dir/_/ldpath.html) .

比较「主流」的做法是修改 `/etc/ld.so.conf` 文件，具体可以参考 [这篇文章](https://www.cnblogs.com/kex1n/p/5993498.html) .





