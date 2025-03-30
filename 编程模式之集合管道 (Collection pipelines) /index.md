## 编程模式之集合管道 (Collection pipelines)  
### 简介

集合管道 (Collection pipelines) 是一种`编程模式`，它让把一些计算 (computation) 组织成一些`有序的`的操作 (operations) 。这些操作通过将`集合`作为一个操作的输出 (output) 并将其送入到下一个操作来组成。常见的操作有 `filter`、`map` 和 `reduce` 。

这种模式在`函数式编程`中很常见，在具有 `lambda` 的面向对象语言中也很常见。

集合管道是软件中最常见且令人满意的模式之一。它也会出现在 unix 命令行上，不同的环境有略微不同的形式，常见的操作有不同的名称，但是一旦你熟悉了这种模式，你就会忘不了它。

### Unix 下的集合管道模式

现在我们当前工作目录有两个文件：`demo.txt`和 `demo1.txt` 。

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

当我们`查找`当前目录下所有包含 `hello` 这个单词的文件时，可以通过下面 `grep` 命令来达成：

``` bash
➜ grep -l 'hello' ./*.*
./demo.txt
./demo1.txt
```

我们还想对每个文件进行`单词数统计`时，于是我们使用 `xargs wc -w` 命令：

``` bash
➜ grep -l 'hello' ./*.* | xargs wc -w
       3 ./demo.txt
       2 ./demo1.txt
       5 total
```

并且我们还想要按照单词个数`排序`，所以使用 `sort -nr` 命令：

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

在 Unix 中，`集合`就是一个文本文件，它的每一行就是集合的每一项，每一行包含若干个由空格分割的值。每个值的含义由其在行中的顺序给出。

这些操作是一个个 Unix 的进程，每个通过`管道操作符` (`|`) 生成的集合，由一个进程的`标准输出`通过管道传输到下一个进程的`标准输入`。

在`面向对象` (object-oriented) 编程中的集合就是`集合类` (list、array、set 等等) 。

在 JavaScript 语言中，集合管道的简单示例：

``` js
[1,2,3,4]
.map((n) => n+ 1)
.filter(n => n > 3)
.reduce((pre, current) => pre + current, 0)
// => 9
```

### 常见的操作符

下面是在集合管道中发现的经常使用的操作符。每种语言都对可用的操作以及它们的名称做出不同的选择，但下面试图通过它们的共同功能来看待它们。

#### map 或 collect

<img src="map.png" width=220 />

将给定函数应用于输入的每个元素并将结果放入输出。

#### concat

<img src="concat.png" width=220 />

将集合连接成一个集合。

#### difference

<img src="difference.png" width=220 />

从管道中删除提供的列表的内容。

#### distinct

<img src="distinct.png" width=220 />

删除重复元素。

#### drop

一种`切片` (slice) 形式，返回除前 n 个元素之外的所有元素。

#### filter 或 select

<img src="filter.png" width=220 />

在每个元素上运行一个`布尔函数`，并且只把返回 true 的元素传递到输出。

#### reject

和 `filter` 相反，返回与谓词不匹配的元素。

#### flat-map 或 mapcat

<img src="flat-map.png" width=220 />，在集合上映射一个函数并将结果展平为一级。

#### flatten (展平)

<img src="flatten.png" width=220 />，从集合中删除嵌套。

#### reduce 或 fold (折叠) 或 inject

<img src="reduce.png" width=220 />，使用提供的函数来组合输入元素，通常是单个输出值。

#### group-by

<img src="group-by.png" width=220 />，对每个元素运行一个函数并按结果对元素进行分组。

#### intersection

<img src="intersection.png" width=220 />，提供一些元素，保留在集合中和他们相同的元素。

#### slice

<img src="slice.png" width=220 />，返回给定的第一个和最后一个位置之间的列表子序列。

#### sort

<img src="sort.png" width=220 />，输出是基于提供的比较器 (comparator) 的输入的排序副本。

#### take

返回前 n 个元素的切片 (slice) 形式。

#### union

<img src="union.png" width=220 />，返回该集合或提供的集合中的元素，删除重复项。

参考资料：

\> [https://martinfowler.com/articles/collection-pipeline/#NestedOperatorExpressions](https://martinfowler.com/articles/collection-pipeline/#NestedOperatorExpressions)
