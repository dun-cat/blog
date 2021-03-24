## Linux的SCP命令 
### 简介

`SCP` --secure copy （远程文件拷贝程序）是用于主机之间的文件拷贝，使用 `SSH`（Secure Shell）来作为数据传输协议（[IETF](https://tools.ietf.org/html/rfc4253)制定），这意味你需要通过`key`或者`密码`来做与远程主机认证。

> 在没有特殊端口指定的情况下。SSH 使用 22 号端口，所以在使用的时候需要开放该端口，该端口已经在[IANA](https://www.iana.org/)里注册，并且官方分配给了 SSH。

SCP的命令大纲：

``` shell
scp [-346BCpqrTv] [-c cipher] [-F ssh_config] [-i identity_file]
    [-J destination] [-l limit] [-o ssh_option] [-P port] [-S program]
    source ... target
```

`source` 和 `target` 可以指定一个本地绝对或相对路径，也可以使用[URI](https://en.wikipedia.org/wiki/Uniform_Resource_Identifier)格式。
URI的形式是`scp://[user@]host[:port][/path]`。

> 通用的 URI 格式是：`[协议名]://[用户名]:[密码]@[主机名]:[端口]/[路径]?[查询参数]#[片段ID]`。

简化来看：

``` shell
scp [OPTION] [user@]SOURCE_HOST:]file1 [user@]TARGET_HOST:]file2
```

* `[user@]SOURCE_HOST:]file1`：源文件
* `[user@]TARGET_HOST:]file2`：目标文件

### 使用

拷贝本地当前路径下的所有文件到远程主机地址为 101.125.45.5 并用户为 root 的 remote_dir 目录里去。

``` shell
scp -r -C ./public/* root@101.125.45.5:/remote_dir/
```

* `-C`：开启SSH压缩传输，只是针对每个文件进行压缩；
* `-r`：递归遍历整个目录；

在本地主机直接使用路径名称即可，远程使用 URI 格式，默认会弹出密码提示框让你执行拷贝动作。

如果拷贝指定目录下所有内容，记得加`*`号，不然会连同`public`拷贝到服务器。

### 使用 key 来传输

使用 key 来传输，即把本地的公钥提供给远程主机，使其能信任复制源，从而`避免手动密码输入`，这需要有一下几个步骤。

**1**.生成秘钥对：

查看本地是否已存在公钥。

``` shell
ls ~/.ssh/id_*
```

通常 ssh 的`秘钥对`存储位置在用户目录下的`.ssh`目录，如果没有该文件，你需要使用`ssh-keygen`生成：

``` shell
ssh-keygen -t rsa -b 4096 -C "your_email@domain.com"
```

生成的过程会提示输入密码，不过通常开发者或系统管理员`不会设置密码`，因为 scp 经常被用于`自动化`操作。

**2**.复制公钥给远程主机：

``` shell
ssh-copy-id remote_username@host
```

你会被要求输入远程用户的密码做认证。

如果你没有`ssh-copy-id`工具，可以手动拷贝公钥文件内容到远程用户`~/.ssh/authorized_keys`目录。

**3**.验证连接是否可用：

``` shell
ssh -T remote_username@host
```

这样就完成了 key 的配置。

### 自动化环境

由于 SCP 是一个接一个（one by one）的上传方式，在上传效率方面会比较差。所以通常我们需要进行整体`打包压缩传输`。

这里会分几个操作：

1. 压缩打包源文件到一个临时目录
2. 上传压缩包到服务器临时目录
3. 移除老的 target 目录
4. 创建 target 目录
5. 解压压缩包到 target 目录
6. 删除压缩包

于是我们会有一个这样的 shell 脚本：

``` shell
#! /bin/sh

tar -cf /tmp/upload.tar -C ./
scp -r -C /tmp/upload.tar root@101.125.45.5:/temp_dir/
ssh root@101.125.45.5 "rm -rf /target_dir"
ssh root@101.125.45.5 "mkdir -p /target_dir"
tar -xf /temp_dir/upload.tar -C /target_dir
rm -rf /temp_dir/upload.tar
```

扩展阅读：

\> [https://linuxize.com/post/how-to-use-scp-command-to-securely-transfer-files/](https://linuxize.com/post/how-to-use-scp-command-to-securely-transfer-files/)

\> [https://en.wikipedia.org/wiki/Uniform_Resource_Identifier](https://en.wikipedia.org/wiki/Uniform_Resource_Identifier)

\> [https://linuxize.com/post/how-to-setup-passwordless-ssh-login/](https://linuxize.com/post/how-to-setup-passwordless-ssh-login/)

\> [https://docs.github.com/en/github/authenticating-to-github](https://docs.github.com/en/github/authenticating-to-github)