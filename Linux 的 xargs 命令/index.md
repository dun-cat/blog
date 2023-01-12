## Linux 的 xargs 命令 
### 简介

xargs (全称：`eXtended ARGumentS`) 是一个在 Unix 和大多数 Unix-like 系统上命令，它用于从[标准输入](https://en.wikipedia.org/wiki/Standard_streams) (standard input) 构建和执行命令。

> `标准输入`也就是标准流 (standard streams) 中的`stdin`。除此之外，标准流还有标准输出 (stdout) 和标准错误 (stderr) 。

### 语法

``` bash
xargs [options] [command [initial-arguments]]
```

### 描述

xargs 从标准输入读取条目。这些条目用`空格` (可以用`双引号`、`单引号`或`反斜杠`转义) 或者`换行` (newlines) 隔开。然后执行一次或者多次`命令(command)` (默认命令是 echo)，命令后面跟随的`initial-arguments`是从标准输入读取的条目。标准输入中的`空白行` (Blank lines) 会被忽略。

command 的命令行会不断地构建，直到达到系统定义的限制 (除非使用了 -n 和 -L 选项)。指定的命令将根据需要多次调用以用完输入项列表。通常，调用命令的次数要比输入中的项目少得多，这会带来显着的性能优势。一些命令也可以并行执行 (请参阅-P选项)。

因为 Unix 文件名可以包含`空格`和`换行`，所以这种默认行为通常是有问题的，xargs 会错误地处理了包含`空格`和`/`或`换行符`的文件名。在这些情况下，最好使用`-0`选项，它可以防止出现此类问题。使用此选项时，您需要确保为 xargs 生成输入的程序也使用一个 null 符作为分隔符。例如，如果该程序是 GNU find ，则 -print0 选项会为您执行此操作。

### 例子

1. 处理包含分隔符 (空格符、/、换行符) 的文件名。

如存在如下一些文件，按照上面的描述我们需要通过指定`-0`选项来解决此问题。

``` shell
➜  tmp find ./*.txt
./ hell.txt
./11 hell.txt
./11\n 23.txt
./demo.txt
```

``` shell

```
