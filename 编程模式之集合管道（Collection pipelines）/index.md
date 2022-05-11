## 编程模式之集合管道（Collection pipelines） 
### 简介

集合管道（Collection pipelines）是一种`编程模式`，它让把一些计算（computation）组织成一些`有序的`的操作（operations）。这些操作通过将`集合`作为一个操作的输出（output）并将其送入到下一个操作来组成。常见的操作有`filter`、`map=`和`reduce`。

这种模式在`函数式编程`中很常见，在具有`lambda`的面向对象语言中也很常见。

收集管道是软件中最常见且令人满意的模式之一。它也会出现在 unix 命令行上，不同的环境有略微不同的形式，常见的操作有不同的名称，但是一旦你熟悉了这种模式，你就会忘不了它。

### Unix 下的集合管道模式

现在我们当前工作目录有两个文件：`demo.txt` 和 `demo1.txt`。

`demo.txt`的内容如下：

``` text
123456789
hello
hello
```

`demo1.txt`的内容如下：

``` text
hello hello
```

当我们找出当前目录下所有包含`hello`这个字符串的文件时，可以通过下面命令来达成：

``` bash
➜ grep -l 'hello' ./*.*
./demo.txt
./demo1.txt
```

如果我们还想统计一共有多少个`hello`时，我们可以像下面这样：

``` bash
➜ grep -l 'hello' ./*.* | xargs wc -w
       3 ./demo.txt
       2 ./demo1.txt
       5 total
```
