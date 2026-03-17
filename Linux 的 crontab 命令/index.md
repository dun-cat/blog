## Linux 的 crontab 命令 
### 简介

crontab (cron table) 用于计划任务的工具，运行在类 Unix 系统上。

任务时间表 (crontab) 文件储存的指令被 crond 守护进程激活，守护进程在后台运行，并`每一分钟`检查是否有定期的作业需要执行。这类作业一般称为 `cron jobs` 。

### 使用

 `crontab` 系统默认处于运行状态，通过设置 cron 文件来支持计划任务。

首先，创建 cron 文件：

``` sh
crontab -d
```

在设定时间格式上，它有比较通用的约定：

``` sh
# ┌───────────── minute (0 - 59)
# │ ┌───────────── hour (0 - 23)
# │ │ ┌───────────── day of the month (1 - 31)
# │ │ │ ┌───────────── month (1 - 12)
# │ │ │ │ ┌───────────── day of the week (0 - 6) (Sunday to Saturday;
# │ │ │ │ │                                   7 is also Sunday on some systems)
# │ │ │ │ │
# │ │ │ │ │
# * * * * * <command to execute>
```

例如如果我们要每分钟执行 `echo` 命令可以如下写法：

``` sh
 * * * * * echo hi
```

### 表达式

* 逗号 (`,`) 表示列举，例如：`1,3,4,7 * * * * echo hello world` 表示，在每小时的 1、3、4、7 分时，打印"hello world"。
* 连词符 (`-`) 表示范围，例如：`1-6 * * * * echo hello world`，表示，每小时的 1 到 6 分钟内，每分钟都会打印"hello world"。
* 星号 (`*`) 代表任何可能的值。例如：在“小时域”里的星号等于是“每一个小时”。
* 百分号(`%`) 表示“每"。例如：`*%10 * * * * echo hello world` 表示，每 10 分钟打印一回"hello world"。
