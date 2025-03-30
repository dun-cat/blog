## Linux 的 tar 命令 
### 简介

tar 是 Unix 和类 Unix 系统上的`归档打包工具`，tar 代表未压缩的 tar 文件。已压缩的 tar 文件则附加数据压缩格式的扩展名，如经过 `gzip` 压缩后的  tar 文件，扩展名为`.tar.gz`。

新版 tar 已能自动调用多种压缩工具执行压缩。已压缩的 tar 文件也叫 `tarball`，大部分自由软件的源代码采用 tarball 的形式发布。

由于受到 `DOS8.3` 文件名格式的限制，常使用下列缩写：

* .tgz 等价于 .tar.gz
* .tbz 与 tb2 等价于 .tar.bz2
* .taz 等价于 .tar.Z
* .tlz 等价于 .tar.lzma
* .txz 等价于 .tar.xz

### 常见参数

* -f：file = 处理的文件位置
* -c：create = 创建新 tar 文件
* -x：extract = 提取文件
* -v：verbose = 显示详情
* -t，--list 列出tar文件中包含的文件的信息
* -z，--gzip，--gunzip，--ungzip 调用gzip执行压缩或解压缩
* -J，--xz，--lzma 调用XZ Utils执行压缩或解压缩。依赖XZ Utils

### 归档文件

``` shell
# 把当前目录的文件a.js 归档到当前目录且名为b.tar的归档文件。
tar -cf b.tar a.js
# 完整写法
tar --create --file b.tar a.js
```

### 归档目录下所有文件

``` shell
tar -cvf /tmp/b.tar -C /source_dir/ .
# 完整写法
tar --create --verbose --file /tmp/b.tar --cd /source_dir/ .

# -C 等同 --cd 等同 --directory
```

这里添加`-C`参数，在归档前进入 `source_dir` 作为工作目录，可以避免把文件路径打包到 tar 包内，提取文件时直接提取到当前目录 (另一方面讲，提取文件时可能覆盖已有文件) 。

tar 包可以创建在系统的临时目录`(/tmp/)`，跟着系统的临时目录清除规则来管理。

### 排除归档项

通常我们需要排除一些依赖目录或者版本管理目录(.git)。

``` shell
tar --exclude .git --exclude node_modules -cvf /tmp/b.tar -C ./ . 
# 排除 .git 和 node_modules 目录
```

### 提取文件

``` bash
tar -xvf xxx.tar
# 完整写法
tar --extract --verbose --file xxx.tar
```

### 查看包内容

``` shell
tar -tf ./xxx.tar
# 完整写法
tar --list --file ./xxx.tar
```

### 压解缩参数

添加`-z`参数配合`-c`和`-x`参数，使用 `gzip` 进行tar包的压解缩操作。

``` shell
# 压缩文件
tar -czvf /tmp/b.tar -C ./ . 
# 解压文件
tar -xzvf /tmp/b.tar  /target_dir/ 
```

参考资料：

\> [https://zh.wikipedia.org/wiki/Tar](https://zh.wikipedia.org/wiki/Tar)
