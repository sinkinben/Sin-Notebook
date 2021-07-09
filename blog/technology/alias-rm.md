## 给rm命令加保险

众所周知，脑残可以学习，但是手残没法治。相信每一位喜欢用终端操作电脑的同学都曾手误使用 `rm` 命令把不该删除的文件删了。然而，使用 `rm` 删除的文件是不会进去回收站的。

所以，最好的方法就是我们自定义一个命令 `del` ，以后通过自定义的 `del` 删除文件。当然，很难做到完全替代 `rm`，但是对于日常使用是足够的。

下面是准备实现的功能：
+ `del file1 file2 ...`： 把每一个文件移入回收站。
+ `del dir`：因为对目录操作的风险较大，因此这里只给出提示信息，让用户自行使用 `rm` 删除。

下面来看 Ubuntu 下的回收站的结构。回收站的路径是 `~/.local/shared/Trash/`，其结构只有三个目录：
+ `files`：被删除的文件的位置。
+ `info`：记录被删除文件的操作信息，包括原路径和删除时间。
+ `expunged`：没查，不知道(>_<)。

```bash
sin@ubuntu:~$ cd .local/share/Trash/
sin@ubuntu:~/.local/share/Trash$ tree .
.
├── expunged
├── files
│   └── testdel
└── info
    └── testdel.trashinfo

3 directories, 2 files
sin@ubuntu:~/.local/share/Trash$ cat info/*
[Trash Info]
Path=/home/sin/workspace/testdel
DeletionDate=2020-02-19T15:42:14
```

那么，实现上面的需求就很简单了：在 `～/.bashrc` 中加入我们的命令函数，然后通过 `alias` 重命名为我们想要的名称就可以了。

```sh
# my alias, add by sinkinben at 2020/02/19
alias del='trash'
write_trashinfo()
{
    info_path='/home/sin/.local/share/Trash/info/'
    abs_path=$info_path$1'.trashinfo'
    echo '[Trash Info]' > $abs_path
    echo Path=$2/$1 >> $abs_path
    echo DeletionDate=`date +"%Y-%m-%dT%H:%M:%S"` >> $abs_path
}
trash()
{
    src_path=`pwd`
    trash_path='/home/sin/.local/share/Trash/files/'
    for x in $@
    do
        if [ -d $x ]
        then
            echo '[BE CAREFUL!] ' $x ' is a directory.'
            echo 'You should use rm to finish this operation by yourself.'
            continue
        fi
        if [ -f $x ]
        then 
            mv $x $trash_path
            write_trashinfo $x $src_path
        fi
    done
}
```

重启一下终端，输入 `alias`，可以找到新加入的 `del` 命令：
```bash
alias alert='notify-send --urgency=low -i "$([ $? = 0 ] && echo terminal || echo error)" "$(history|tail -n1|sed -e '\''s/^\s*[0-9]\+\s*//;s/[;&|]\s*alert$//'\'')"'
alias del='trash'
alias egrep='egrep --color=auto'
alias fgrep='fgrep --color=auto'
alias grep='grep --color=auto'
alias l='ls -CF'
alias la='ls -A'
alias ll='ls -alF'
alias ls='ls --color=auto'
```

+ 测试 1：删除普通的文件
  ```bash
  sin@ubuntu:~/workspace$ touch f1 f2 f3
  sin@ubuntu:~/workspace$ ls
  f1  f2  f3  scripts  Sin-Notebook  sjtu-courses    snake.py
  sin@ubuntu:~/workspace$ del f*
  sin@ubuntu:~/workspace$ tree ~/.local/share/Trash/
  /home/sin/.local/share/Trash/
  ├── expunged
  ├── files
  │   ├── f1
  │   ├── f2
  │   ├── f3
  │   └── testdel
  └── info
      ├── f1.trashinfo
      ├── f2.trashinfo
      ├── f3.trashinfo
      └── testdel.trashinfo
  
  3 directories, 8 files
  sin@ubuntu:~/workspace$ cat ~/.local/share/Trash/info/*
  [Trash Info]
  Path=/home/sin/workspace/f1
  DeletionDate=2020-02-19T15:52:12
  [Trash Info]
  Path=/home/sin/workspace/f2
  DeletionDate=2020-02-19T15:52:12
  [Trash Info]
  Path=/home/sin/workspace/f3
  DeletionDate=2020-02-19T15:52:12
  [Trash Info]
  Path=/home/sin/workspace/testdel
  DeletionDate=2020-02-19T15:42:14
  ```

+ 测试2： 删除目录
  ```bash
  sin@ubuntu:~/workspace$ mkdir dir1
  sin@ubuntu:~/workspace$ del dir1/
  [BE CAREFUL!]  dir1/  is a directory.
  You should use rm to finish this operation by yourself.
  ```

当然，这有一个缺点，对于被删除的同名文件，这个 `del` 命令就很捉急了。比如：

```bash
del dir1/test
del dir2/test
```

显然，在回收站中，`dir1/test` 这个文件就被 `dir2/test` 给覆盖了（包括文件内容和日志信息）。但是如果使用 Ubuntu 的文件管理器进行删除，回收站是能够处理这种同名情况的。

好了，又水了一篇文章，祝各位小改改身体健康。