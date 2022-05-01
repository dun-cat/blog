## Linux 的关机重启命令 
### shutdown 命令

`-k：发出警告消息，不会关机。`

``` bash
shutdown -k now '各位在线的用户，本机子要关机了。'
```

在线的用户会收到如下消息：

``` bash
Broadcast message from root@iZ233h0dt5tZ
    (/dev/pts/0) at 16:04 ...

The system is going down for maintenance NOW!
各位在线的用户，本机子要关机了。
```

`-h：服务器把系统的服务停掉之后就立即关机。`

``` bash
shutdown -h now
```

在线的用户会收到如下消息：

``` bash
Broadcast message from root@iZ233h0dt5tZ
    (/dev/pts/0) at 16:15 ...

The system is going down for halt NOW!
```

指定关机时间

``` bash
# 固定时间关机
shutdown -h 14:00
# 5分钟后关机
shutdown -h +5
```

`-r：将系统服务关掉后立即重启。`

``` bash
# 立即重启
shutdown -r now

# 5分钟之后重启，并提前发消息给用户
shutdown -r +5 '系统即将重启'

# 固定时间重启
shutdown -r 20:00 '用户您好，系统即将关机。'
```

### halt 命令

halt 会先调用 shutdown，而 shutdown 最后会调用halt。不过 shutdown 会逐个关闭所有服务，然后关机。至于 halt -k 可以不用理会目前系统情况，进行硬件关机的特殊功能。

### init 命令

``` bash
init 0
```

* 0：关机
* 3：纯命令行模式
* 5：含有图形界面模式
* 6：重启

### poweroff 命令

``` bash
# 关机
poweroff -f
```

### reboot 命令

``` bash
# 重启
reboot
```
