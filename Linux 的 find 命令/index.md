## Linux 的 find 命令 
### 语法

find [path] [expression]

查看详情请看 man 手册

``` bash
man find
```

### 默认

 `path` 路径默认从当前目录向其子目录查找。
 `expression` 表达式默认值 -print 。打印输出。

> mac上必须指定 path。不同 linux 发行版，指令参数也有所不同，但大同小异，可自行 man。

``` shell
find
# 等于
find . -print
```

### 组成

 `expression` 可以由操作符`(operators)`、选项`(options)`、测试集`(tests)`、动作`(actions)`几部分组成。

#### 操作符( `operators` )

你可以对 `expression` 通过简单逻辑操作符进行条件操作。
以下都是允许：

* `!` EXPR = `-not` EXPR
* EXPR1 `-a` EXPR2 =  EXPR1 `-and` EXPR2
* EXPR1 `-o` EXPR2 =  EXPR1 `-or` EXPR2  = EXPR1 `,` EXPR2

#### 选项( `options` )

-E ：对测试项  [-&#91;i&#93;regex](#regex)，采扩展正则表达式。

> POSIX 定义了两种正则表达式语法基本正则表达式 (BRE) 和扩展正则表 达式 (ERE) 。

#### 测试集( `tests` )

测试集就是过滤条件。

变量解释：

1. PATTERN：匹配表达式。可以使用通配符：*、?、[]。通配符与正则表达式语句有些相似，但它与正则表达式不同。
2. N：数字，可以是大于N (+N) 或 小于N(-N) 或 等于 (N)
3. FILE：可以是多种类型的文件，目录也算其中一种。
4. NAME：名称
5. i：可以忽略大小写

过滤条件如下：

* `-[i]name PATTERN`：模式匹配文件名称
默认填写文件名称，只会匹配目录。所以可以通过匹配模式，匹配所想要的文件或目录。例如：a&#42;、b.&#42;、&#42;.&#42;等
* `-type [bcdpflsD]`：匹配对应文件类型的文件
b: 块设备文件(block device file)
c: 字符设备文件(character device file)
d: 目录文件(directory)
p: 命名管道文件(named pipe)
f: 一般文件(regular file)
l: 符号链接文件(symbolic link) (指向另一个文件,类似于windows下的快捷方式)
s: 套接字文件(local socket file)
D：door类型文件 (Solaris 系统专有类型)
* `-size N[bcwkMG]`：匹配N[bcwkMG]单位大小的文件
b: 512 bytes block
c: bytes(B)
w: two bytes word
k: kilobytes
M: megabytes
G: gigabytes
* <a name = "regex">`-[i]regex PATTERN`</a>： 匹配正则表达式的文件
* `-amin N`：匹配N分钟访问的文件
* `-anewer FILE`: 匹配最后一次访问超出最近一次其修改时间的文件
* `-cmin N`：匹配N分钟修改的文件
* `-empty`：匹配空文件
* `-path PATTERN`：匹配模式路径的文件
* `-readable`：匹配可读文件
* `-writable`：匹配可写文件
* `-executable`：匹配可执行文件
* `-group NAME`：匹配组名为NAME的文件
* `-user NAME`：匹配文件所有者为NAME的文件

##### 示例

``` bash
# 以当前目录为起点，找到名称包含 foo 却不以.bar为结尾的目录。最后打印出来.
find . -name '*foo*' ! -name '*.bar' -type d -print # -print可以省略
# 找到 大于 1G 的文件
find . -size +1G
```

还有一些过滤条件雷同，不在叙述。

#### 动作( `actions` )

在查找到文件后，可以对他们执行一些操作。例如：-print 就是默认动作。

变量解释：

1. FORMAT：格式化输出，具体参考man手册
%p：输出文件名，包括路径名
%f：输出文件名，不包括路径名
%m：以8进制方式输出文件的权限
%g：输出文件所属的组
%h：输出文件所在的目录名
%u：输出文件的属主名
... ...
2. `COMMAND` ：命令

动作参数如下：

* `-delete`： 删除查找到的文件；
* `-print0`：
* `-printf FORMAT`：格式化打印
* `-fprintf FILE FORMAT`：文件格式化打印
* `-print`：默认打印
* `-fprint0 FILE`：
* `-fprint FILE`：
* `-ls`：以 `ls -l` 命令方式的显示结果。
* `-fls FILE`：
* `-prune`：
* `-quit`：
* `-exec COMMAND \;`：执行外部命令
 `\;`：`;`元字符为命令做分割，表示忽略结果继续执行下一条命令。 `\` ：负责转义。[详情](https://www.cnblogs.com/cynchanpin/p/7399164.html)
* `-exec COMMAND {} \;`：对每一个输出文件结果执行 COMMAND。
`{}`：{}元字符将命令置于non-named function中运行。
* `-ok COMMAND {} \;`：对每一个结果执行 COMMAND；每次执行由用户进行确认；
* `-execdir COMMAND \;`：从文件所在的目录运行指定的命令
* `-execdir COMMAND {} \;`：
* `-okdir COMMAND \;`：

##### 示例

``` bash
# 查找名字为 foo 的文件，并对每一个输出的文件执行 ls 命令。
find . -name foo -exec ls  -l \;

# 删除文件名为a的任意格式文件。
find . -name "a.*" -exec rm {} \;
# 建议使用 -ok 动作，执行命令前确认一下。
find . -name "a.*" -ok rm {} \;
```

参考资料：

\> [https://blog.csdn.net/MrDongShiYi/article/details/81625172](https://blog.csdn.net/MrDongShiYi/article/details/81625172)

\> [https://www.lifewire.com/uses-of-linux-command-find-2201100](https://www.lifewire.com/uses-of-linux-command-find-2201100)

\> [http://man7.org/linux/man-pages/man1/find.1.html](http://man7.org/linux/man-pages/man1/find.1.html)

\> [https://www.cnblogs.com/davidwang456/p/3753707.html](https://www.cnblogs.com/davidwang456/p/3753707.html)
