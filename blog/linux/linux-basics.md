## Linux Note

记录 Linux 的常用工具的基本使用方法。



## tmux

tmux 能让我们把程序放在后台运行，退出 bash 后，程序在 Console 的输出仍然能够保留。

命令：

+ `tmux new -s xxx` : 新建一个 session
+ `tmux a -t xxx` : 重新进入某个 session

快捷键：

+ `Ctrl + B, [` : 通过方向键上下滑动，查看程序输出的内容
+ `Ctrl + B, D` : 离开当前 session
+ `Ctrl + B, C` : 创建一个新的窗口
+ `Ctrl + B, N/P` : 切换 Next/Previous Windows
+ `Ctrl + B, %`：划分左右两个窗格
+ `Ctrl + B, "`：划分上下两个窗格

