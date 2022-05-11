## Linux 的 xargs 命令 
### 简介

xargs（全称：`eXtended ARGumentS`）是一个在 Unix 和大多数 Unix-like 系统上命令，它用于从[标准输入](https://en.wikipedia.org/wiki/Standard_streams)（standard input）构建和执行命令。

> `标准输入`也就是标准流（standard streams）中的`stdin`。除此之外，标准流还有标准输出（stdout）和标准错误（stderr）。

它把`标准输入`的 input 转换成`参数`（arguments）给一个命令。

### 语法

``` bash
Usage: xargs [OPTION]... COMMAND [INITIAL-ARGS]...
```
