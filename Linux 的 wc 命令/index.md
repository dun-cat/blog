## Linux 的 wc 命令 
### 简介

打印每个文件的`换行符`、`单词数` (word count) 和`字节数`，如果指定了多个文件，则打印总行数。没有文件，或者当文件为`-`时，读取`标准输入` (stdin) 。

### 语法

``` bash
wc [OPTION]... [FILE]...
wc [OPTION]... --files0-from=F
```

### 例子

有如下一个文件 `demo.txt` ：

``` text
我们是社会主义接班人。
火锅底料你喜欢哪一种？
How to setup cron jobs in Ubuntu
```

通过 `wc demo.txt`，打印输出`换行符`数、`单词数`及`字节数`。

``` shell
wc demo.txt
    2      11     101 demo.txt
```

* 第一个`2`意思是文本中包含 2 个换行符，所以文本是三行内容。如果需要统计出 3 行，那么需要再最后一行加入终结符 (换行符) ；
* 第二个`11`指的是有 11 个单词。可以看到这里的统计其实并不准确。因为 wc 并不能分割中文单字，除非我们在中文的每个字中间加个空格。当然这是愚蠢的；
* 第三个`101`表示字节数为 101 个。

参考资料：

\> [https://linux.die.net/man/1/wc](https://linux.die.net/man/1/wc)
