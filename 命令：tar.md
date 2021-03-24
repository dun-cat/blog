## 命令：tar 
### 压缩文件

``` bash 
# 把当前目录的文件a.js 压缩到当前目录且名为b.tar的压缩文件。
tar -cf b.tar a.js
# 完整写法
tar --create --file b.tar a.js
```

### 压缩目录下所有文件

``` shell
tar -cvf /tmp/b.tar -C /source_dir/ .
# 完整写法
tar --create --verbose --file /tmp/b.tar --cd /source_dir/ .

# -C 等同 --cd 等同 --directory
```

这里添加`-C`参数，在压缩前进入工作目录，可以避免把文件路径打包到压缩包内，压缩包可以建在系统临时目录`(/tmp/)`，跟着系统的临时目录清除规则来管理。

### 排除压缩项

通常我们需要排除一些依赖项或者版本管理目录(.git)。

``` shell

tar -cvf /tmp/b.tar -C /source_dir/ . --exclude .git

```

### 解压


``` bash
# 解压xxx.tgz文件，并显示细节
tar -zxvf xxx.tgz
```

* .tgz = .tar.gz = 表示压缩格式为gzip，所以解压应带上相应的解压格式-z

* -f：file = 处理的文件位置

* -c：create = 压缩文件

* -z：gzip = 以gzip格式解压或压缩

* -x：extract = 解压文件

* -v：verbose = 显示详情


### 查看压缩包内容

``` shell
tar -tf ./xxx.tar
# 完整写法
tar --list --file ./xxx.tar
```

其他命令查询 tar -h