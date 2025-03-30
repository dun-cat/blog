## Linux 的 xargs 命令 
### 简介

xargs (全称：`eXtended ARGumentS` ) 是一个在 Unix 和大多数 Unix-like 系统上命令，它用于从[标准输入](https://en.wikipedia.org/wiki/Standard_streams) (standard input) 构建和执行命令。

> `标准输入`也就是标准流 (standard streams) 中的 `stdin` 。除此之外，标准流还有标准输出 (stdout) 和标准错误 (stderr) 。

### 语法

``` bash
xargs [options] [command [initial-arguments]]
```

### 描述

xargs 从标准输入读取条目。这些条目用`空格` (可以用`双引号`、`单引号`或`反斜杠`转义) 或者`换行` (newlines) 隔开。然后执行一次或者多次`命令(command)` (默认命令是 echo)，命令后面跟随的 `initial-arguments` 是从标准输入读取的条目。标准输入中的`空白行` (Blank lines) 会被忽略。

command 的命令行会不断地构建，直到达到系统定义的限制 (除非使用了 -n 和 -L 选项)。指定的命令将根据需要多次调用以用完输入项列表。通常，调用命令的次数要比输入中的项目少得多，这会带来显着的性能优势。一些命令也可以并行执行 (请参阅-P选项)。

因为 Unix 文件名可以包含`空格`和`换行`，所以这种默认行为通常是有问题的，xargs 会错误地处理了包含`空格`和`/`或`换行符`的文件名。在这些情况下，最好使用`-0`选项，它可以防止出现此类问题。使用此选项时，您需要确保为 xargs 生成输入的程序也使用一个 null 符作为分隔符。例如，如果该程序是 GNU find ，则 -print0 选项会为您执行此操作。

### 例子

首先我们读取当前目录`.txt`后缀的文件：

``` shell
➜  tmp find ./*.txt
./ hell.txt
./11 hell.txt
./11\n 23.txt
./demo.txt
```

在 Unix 系统下，上面的文件名都是合法的。并且结果是一个`集合`。

> 在 Unix 中，集合就是一个文本文件，它的每一行就是集合的每一项，通过管道操作符 (|) 可以把上一个命令的标准输出传输到下一个进程的标准输入。这是一种[集合管道模式](/articles/programmin-pattern-collection-pipeline/)的应用。

然后，通过管道操作符连接 xargs 命令，我们把标准输入转成参数 (initial-arguments)。

``` shell
➜  tmp find ./*.txt | xargs
./ hell.txt ./11 hell.txt ./11n 23.txt ./demo.txt
```

xargs 的默认命令是 echo，所以打印了参数，并且默认他们以`空格`隔开。值得**注意**的是文件`./11\n 23.txt`变成了`./11n 23.txt`，反斜杠 `\` 作为转义符处理了。

这里我们需要知道如果集合包含`'`、`"`或 `\` ，那么作为命令参数他们是有含义存在的，所以我们的**文件名尽可能得不要包含这些字符**。

重新命名文件后，如下展示：

``` shell
➜  tmp find ./*.txt | xargs
./a.txt ./b.txt ./c.txt ./d.txt
```

结合 `wc -l` 命令，可以统计每个文件的行数：

``` shell
➜  tmp find ./*.txt | xargs wc -l
       1 ./a.txt
       2 ./b.txt
       1 ./c.txt
       2 ./d.txt
       6 total
```
