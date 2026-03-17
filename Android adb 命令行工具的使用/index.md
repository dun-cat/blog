## Android adb 命令行工具的使用 
### 介绍

Android 调试桥 (adb) 是一种功能多样的命令行工具，可与设备进行通信。

adb 命令可用于执行各种设备操作 (例如安装和调试应用) ，并提供对 Unix shell (可用来在设备上运行各种命令) 的访问权限。它是一种 C/S 程序，包括以下三个组件：

* 客户端：用于发送命令。客户端在开发计算机上运行。你可以通过发出 adb 命令从命令行终端调用客户端。
* 守护程序 (adbd)：用于在设备上运行命令。守护程序在每个设备上作为后台进程运行。
* 服务器：用于管理客户端与守护程序之间的通信。服务器在开发机器上作为后台进程运行。

在 Android studio 里安装了 SDK 后，可以在平台工具 (platform-tools) 对应的目录里找到 adb 工具。

工具的位置为 `android_sdk/platform-tools/adb` 里，在我的 mac 本地默认的位置为`/Users/lumin/Library/Android/sdk/platform-tools`。

当然你可以手动配置系统环境变量，然后直接执行 adb 命令，这样一劳永逸。

### 列出已连接设备

``` bash
➜  platform-tools ./adb devices -l
List of devices attached
8FD0218A25001582       device usb:338821120X product:CLT-AL00l model:CLT_AL00l device:HWCLT transport_id:6
192.168.43.57:5555     device product:GLL-AL00CN model:GLL_AL00 device:HWGLL transport_id:5
```

上面连接了两个设备，第一个设备是 USB 数据线直连，第二个设备是在同一局域网下，通过设定 ip 地址连接的。

> 第一项为设备的标识 (序列号) ，后续命令指定设备都将通过该标识来操作。

通常在多设备情况下你需要指定序列号：

``` bash
adb -s 8FD0218A25001582 command
```

adb 会针对每个设备输出以下状态信息：

* 序列号：由 adb 创建的字符串，用于通过端口号唯一标识设备。
* 状态：设备的连接状态可以是以下几项之一：
  * offline：设备未连接到 adb 或没有响应。
  * device：设备现已连接到 adb 服务器。请注意，此状态并不表示 Android 系统已完全启动并可正常运行，因为在设备连接到 adb 时系统仍在启动。不过，在启动后，这将是设备的正常运行状态。
  * no device：未连接任何设备。
  
* 说明：如果命令包含 -l 选项，devices 命令会告知你设备是什么。当你连接了多个设备时，此信息很有用，可帮你将它们区分开来。

### 连接远程设备

``` bash
➜  platform-tools ./adb connect 192.168.43.57:5555
already connected to 192.168.43.57:5555
```

### 文件互传

你可以使用 `pull` 和 `push` 命令将文件复制到设备或从设备复制文件，使用 `pull` 和 `push` 命令可将任意目录和文件复制到设备中的任何位置。

从设备中复制某个文件或目录 (及其子目录) ：

``` bash  
adb pull remote local
```

将某个文件或目录 (及其子目录) 复制到设备：

``` bash
adb push local remote
```

复制一个文件到`/sdcard/haps` 目录

``` bash
➜  platform-tools ./adb -s 8FD0218A25001582 push /Users/lumin/build/outputs/hap/release/entry-release-rich-signed.hap /sdcard/haps 
/Users/lumin/build/output...ap: 1 file pushed, 0 skipped. 97.9 MB/s (176627 bytes in 0.002s)
```

### 停止 adb 服务器

在某些情况下，你可能需要终止 adb 服务器进程，然后重启以解决问题 (例如，如果 adb 不响应命令) 。

``` bash
adb kill-server
```

### 向设备发送 shell 命令

在发送命令前，需要指定哪台设备。如果只有一台连接设备，则不需要指定，系统默认发送命令到该设备。

``` bash
adb [-d | -e | -s serial_number] command
```

#### 单命令操作

列出设备根目录

``` bash
➜  platform-tools ./adb -s 8FD0218A25001582 shell ls -l / 
total 28
drwxr-xr-x   2 root   root      0 2018-08-08 00:01 3rdmodem
drwxr-xr-x   2 root   root      0 2018-08-08 00:01 3rdmodemnvm
drwxr-xr-x   2 root   root      0 2018-08-08 00:01 3rdmodemnvmbkp
dr-xr-xr-x 110 root   root      0 2021-05-17 10:19 acct
lrw-r--r--   1 root   root     11 2018-08-08 00:01 bin -> /system/bin
lrw-r--r--   1 root   root     50 2018-08-08 00:01 bugreports -> /data/user_de/0/com.android.shell/files/bugreports
drwxrwx---   7 system cache  4096 2019-08-29 02:56 cache
lrw-r--r--   1 root   root     13 2018-08-08 00:01 charger -> /sbin/charger
drwxr-xr-x   4 root   root      0 1970-01-01 08:00 config
drwxr-xr-x   7 root   root     79 2018-08-08 00:01 cust
lrw-r--r--   1 root   root     17 2018-08-08 00:01 d -> /sys/kernel/debug

```

#### 交互式 shell 命令

``` bash
adb [-d | -e | -s serial_number] shell
```

退出交互式命令：

``` bash
exit
```

### 查看设备支持的工具

Android 提供了大多数常见的 Unix 命令行工具。如需查看可用工具的列表，请使用以下命令：

``` bash
adb shell ls /system/bin
```

### 安装应用

``` bash
adb install path_to_apk
```

参考资料：

\> [https://developer.android.google.cn/studio/command-line/adb](https://developer.android.google.cn/studio/command-line/adb)
