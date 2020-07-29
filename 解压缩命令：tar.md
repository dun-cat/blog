## 解压缩命令：tar 
### 压缩

``` bash
# 把文件a.js 压缩成文件名为b.tar的压缩文件。
tar -cf b.tar a.js
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

> 解压文件，只能解压到当前目录下。

其他命令查询 tar -h