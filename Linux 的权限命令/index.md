## Linux 的权限命令 
### chgrp 命令 (change group)

把 test.log 文件所在的用户组改为users。

修改文件所在的用户组

``` bash
chgrp users test.log
```

### chown 命令 (change owner)

把 test.log 文件所有者改为 lumin。

修改文件的所有者

``` bash
chown lumin test.log
```

### chmod 命令 (change mode)

权限有三种类型：`read(r)` 可读、`write(w)` 可写、`execute(e)` 可执行。

权限所要设置的对象也有三种类型：`owner(o)` 文件所有者、`group(g)` 文件所在用户组、`other(o)` 其它用户、`all(a)` 前面三项。

修改文件权限写法有两种：`数字写法`和`符号写法`。

 `x = 1` , `w = 2` , `r = 4` , 无任何权限 = 0。

可进行加法运算：`rwx = 4 + 2 + 1 = 7` 。

### 修改权限的多种写法

#### 数字写法

让 test.log 文件的的权限改变为：user 拥有全部权限、group 只有 read 权限、other 只有 write 权限。
修改权限：

``` bash
# 每一位数字分别设置一个对象：user group other
chmod 742 test.log

# 修改前的test.log文件属性
-rw-rw-r-- 1 lumin520 lumin520 0 Nov 20 23:16 test.log
#修改后的test.log文件属性
-rwxr---w- 1 lumin520 lumin520 0 Nov 20 23:16 test.log
```

#### 符号写法

让 test.log 文件的的权限改变为：user 去掉执行权限、group 添加 write 权限、other 只有执行权限。
修改权限：

``` bash
# user (u) 去掉(-) 执行(x)权限
chmod u-x test.log
# group(g) 添加(+) 写(w)权限
chmod g+w test.log
# other(o) 设置(=) 执行(x)权限
chmod o=x test.log

#修改前的test.log文件属性
-rwxr---w- 1 lumin520 lumin520 0 Nov 20 23:16 test.log
#修改后的test.log文件属性
-rw-rw---x 1 lumin520 lumin520 0 Nov 20 23:16 test.log
```

应用到子目录的文件或文件夹

使用 `-R` 符号。 `R = recursive = 递归` 

``` bash
chgrp -R lumin testDir
chown -R lumin testDir
chmod -R 755 lumin testDir
```

### 权限对于文件和目录的区别

#### 文件的权限

* w(write) 可以编辑、新增、或者修改改文件的内容(但不可以删除该文件)
* x(execute) 可被执行
* r(read) 内容的可见性

> Windows通过"扩展名"来判断文件是否可执行，而在Linux里则和文件名没有任何关系，依靠文件是否具有x权限来判断。

#### 目录的权限

* r(read)  可以查看该目录下的文件名数据。因此，你可以用ls查看目录内容
* w(write)  可以新建、删除、重命名、移动子目录或子文件
* x(execute)  x代表你能够进入到该目录，让它成为你的工作目录，既你的当前目录。简单来说决定了cd命令能否有权执行。所以在架设网站时，文件即便有了read权限，也会出现403错误。所以上级所有目录都必须给予x权限。
