## Linux 的 xargs 命令 
### 简介

xargs（全称：`eXtended ARGumentS`）是一个在 Unix 和大多数 Unix-like 系统上命令，它用于从[标准输入](https://en.wikipedia.org/wiki/Standard_streams)（standard input）构建和执行命令。

> `标准输入`也就是标准流（standard streams）中的`stdin`。除此之外，标准流还有标准输出（stdout）和标准错误（stderr）。

man 手册对 xargs 的描述如下：

此手册页描述 GNU 版本的 xargs. xargs 从标准输入读取条目。这些条目用`空格`（可以用`双引号`、`单引号`或`反斜杠`转义）或者`回车`隔开. 然后执行一次或者多次命令(默认 是 /bin/echo), 其 参数 是 initial-arguments 后面 再 加上 从 标准 输入 读入 的 参数. 标准 输入中 的 空格 被 忽略.

### 语法

``` bash
Usage: xargs [OPTION]... COMMAND [INITIAL-ARGS]...
```

```ts
const fn = () => {
  if("123" === 3) {

  }
  if("123" >= 3) {
    
  }
}
```
