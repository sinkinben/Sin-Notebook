## Shell and Scripts

**`''` 和 `""` 的区别**

```shell
foo=bar
echo "$foo"
# 打印 bar
echo '$foo'
# 打印 $foo
```



**`$` 符号**

以下面函数为例子：

```shell
mcd () {
    mkdir -p "$1"
    cd "$1"
}
```

其中：

- `$0` 表示脚本名
- `$1 - $9` 表示函数的参数（或者脚本的参数）
- `$@` 表示所有参数
- `$#` 表示参数的个数
- `$?` 表示前一个命令的返回值
- `$$` 表示当前脚本的进程 PID
- `!!` 表示上一个输入的完整命令，包括参数。常见应用：当你因为权限不足执行命令失败时，可以使用 `sudo !!`再尝试一次。
- `$_` 表示上一条命令的最后一个参数。如果你正在使用的是交互式 shell，你可以通过按下 `Esc` 之后键入 . 来获取这个值。





**短路运算符 (Short-circuiting)**

- 命令 `true` 的返回码永远是`0`，`false` 的返回码永远是`1`。（注意下面的例子中，Shell 不是通过命令的 `Exit Code` 来判断逻辑表达式的真值的。）
- `;` 表示分隔 2 个命令语句。

```shell
false || echo "Oops, fail"
# Oops, fail

true || echo "Will not be printed"
#

true && echo "Things went well"
# Things went well

false && echo "Will not be printed"
#

false ; echo "This will always run"
# This will always run
```



**$ (CMD) 的用法**

