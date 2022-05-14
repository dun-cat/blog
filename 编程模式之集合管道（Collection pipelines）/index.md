## 编程模式之集合管道（Collection pipelines） 
### 简介

集合管道（Collection pipelines）是一种`编程模式`，它让把一些计算（computation）组织成一些`有序的`的操作（operations）。这些操作通过将`集合`作为一个操作的输出（output）并将其送入到下一个操作来组成。常见的操作有`filter`、`map=`和`reduce`。

这种模式在`函数式编程`中很常见，在具有`lambda`的面向对象语言中也很常见。

收集管道是软件中最常见且令人满意的模式之一。它也会出现在 unix 命令行上，不同的环境有略微不同的形式，常见的操作有不同的名称，但是一旦你熟悉了这种模式，你就会忘不了它。

### Unix 下的集合管道模式

现在我们当前工作目录有两个文件：`demo.txt` 和 `demo1.txt`。

demo.txt 的内容如下：

``` text
123456789
hello
hello
```

demo1.txt 的内容如下：

``` text
hello hello
```

当我们`查找`当前目录下所有包含`hello`这个单词的文件时，可以通过下面`grep`命令来达成：

``` bash
➜ grep -l 'hello' ./*.*
./demo.txt
./demo1.txt
```

我们还想对每个文件进行`单词数统计`时，于是我们使用`xargs wc -w`命令：

``` bash
➜ grep -l 'hello' ./*.* | xargs wc -w
       3 ./demo.txt
       2 ./demo1.txt
       5 total
```

并且我们还想要按照单词个数`排序`，所以使用`sort -nr`命令：

``` bash
➜ grep -l 'hello' ./*.txt | xargs wc -w | sort -nr
       5 total
       3 ./demo.txt
       2 ./demo1.txt
```

如果只想看`前两行`并且`移除 total`，所以继续添加以下命令：

``` bash
➜ grep -l 'hello' ./*.txt | xargs wc -w | sort -nr | head -2 | tail -1
       3 ./demo.txt
```

### 集合

在 Unix 中，集合就是一个文本文件，它的每一行就是集合的每一项，每一行包含若干个由空格分割的值。每个值的含义由其在行中的顺序给出。

这些操作是一个个 Unix 的进程，每个通过`管道操作符`（`|`）生成的集合，由一个进程的`标准输出`通过管道传输到下一个进程的`标准输入`。

在`面向对象`（object-oriented）编程中的集合就是`集合类`（list、array、set 等等），集合中的每一项都是`对象`（object）。

参考文献：

\> [https://martinfowler.com/articles/collection-pipeline/#NestedOperatorExpressions](https://martinfowler.com/articles/collection-pipeline/#NestedOperatorExpressions)
