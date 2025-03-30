## Linux 的用户和组 
### 简介

Linux 上做事要用到用户。初始系统默认只有一个 `root` 用户，该用户拥有`最高权限`。运维上，通常会开通不同的用户服务于不同的服务软件，在对待root用户及密码都应该极为谨慎。
通过查看 `/etc/passwd` 文件可以`列出所有用户`，通过查看 `/etc/group`文件可以`列出所有组`

### 查看用户

``` bash
# 查看主机用户列表
cat /etc/passwd 
# -- output --
# root:x:0:0:root:/root:/bin/bash
# daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
# bin:x:2:2:bin:/bin:/usr/sbin/nologin
# sys:x:3:3:sys:/dev:/usr/sbin/nologin
# sync:x:4:65534:sync:/bin:/bin/sync
# games:x:5:60:games:/usr/games:/usr/sbin/nologin
# man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
# lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
# ...
```

### 查看组

``` bash
# 查看主机组列表
cat /etc/group
# -- output --
# root:x:0:
# daemon:x:1:
# bin:x:2:
# sys:x:3:
# adm:x:4:syslog
# tty:x:5:
# disk:x:6:
# lp:x:7:
# ...
```

### 添加用户

添加用户会用到两个命令: `adduser` 和 `useradd`，这两个会有一些区别：

1. `useradd` 不会在`/home`目录创建用户目录，不会自动选择shell版本，也不会提示设置密码。所以也就不能直接登录。可以使用 `passwd` 命令修改密码。
2. `adduser` 则会创建用户目录，选择shell版本，提示设置密码。完成步骤之后可直接登录。

> 在 CentOS 系统下 useradd 与 adduser 一样。

``` bash
# 添加用户
adduser demo

# Adding user `demo' ...
# Adding new group `demo' (1004) ...
# Adding new user `demo' (1003) with group ` demo' ...
# Creating home directory `/home/demo' ...
# Copying files from `/etc/skel' ...
# Enter new UNIX password: 
# Retype new UNIX password: 
# passwd: password updated successfully
# Changing the user information for demo
# Enter the new value, or press ENTER for the default
#       Full Name []: 
#        Room Number []: 
#        Work Phone []: 
#        Home Phone []: 
#        Other []: 
# Is the information correct? [Y/n] 
```

### 查看在线的用户

``` bash
root@iZm5eehitkhg98sinqalalZ: w
 11:29:07 up 351 days, 23:44,  1 user,  load average: 0.13, 0.14, 0.10
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
root     pts/0    115.236.67.42    10:17    0.00s  0.17s  0.17s -bash
```
