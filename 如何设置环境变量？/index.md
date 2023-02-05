## 如何设置环境变量？ 
在日常工作中，我们经常需要为一些通过命令行运行的应用程序设置环境变量。通过环境的设置，让 Shell 能够在任何工作目录下都能找到他们并通过命令直接启动。

### 查看环境变量

#### 打印所有变量

我们可以通过 `printenv` 命令来查看当前的所有环境变量列表：

``` bash
TERM_SESSION_ID=w0t0p0:20776E1B-31FF-4533-8303-06EB31B1461D
SSH_AUTH_SOCK=/private/tmp/com.apple.launchd.nAalHY9CCE/Listeners
LC_TERMINAL_VERSION=3.4.16
COLORFGBG=7;0
ITERM_PROFILE=Default
XPC_FLAGS=0x0
PWD=/Users/lumin
SHELL=/bin/zsh
...
```

#### 打印指定变量

通过 `echo` 回显命令查看指定环境变量值：

``` bash
echo $PWD
```

也可以使用 `printenv` ：

``` bash
printenv PWD
```

### 设置临时环境变量

通过 `export` 命令可以分配一个临时环境变量，仅限于当前 shell 会话，退出登录就没了。

``` bash
export LUMIN=123
```

``` bash
echo $LUMIN
# 123
```

### 持久化环境变量

我们要永久保存该变量，需要写入配置文件。不同的操作系统，配置文件名称不同。

#### Mac 配置文件

在早期 mac 系统版本使用`用户目录`的`.bash_profile`或`.profile`文件。从 macOS `Catalina` 开始，Mac 使用 [zsh](https://support.apple.com/zh-cn/HT208050) 作为默认登录 Shell 和交互式 Shell。

所以它的配置文件是用户目录的`.zshrc`文件。

例如：如果我们要将 flutter 二进制执行程序目录路径添加到 PATH 变量中去。

可以修改该文件：

``` bash
# If you come from bash you might have to change your $PATH.
export PATH=$HOME/bin:/usr/local/bin:$HOME/flutter/bin:$PATH

# more ...
```

> 注意的是目录路径之间用英文冒号 (`:`) 隔开。

为了快速在当前 shell 生效，使用 `source` 命令重新读取文件配置：

``` bash
source ~/.zshrc
```

#### Liunx 配置文件

Liunx 的配置文件为用户目录的`.bashrc`文件或`.bash_profile`文件。在设置变量命令和方式与 mac 是一样的。

如果要对所有登录用户生效，你需要修改`/etc/bashrc`文件、`/etc/profile`文件或`/etc/environment`文件。它需要你有管理员权限来执行对它们的修改。

### 系统变量 vs 用户变量

环境变量分为`系统环境变量`和`用户环境变量`。系统环境变量**所有用户**都可以访问，用户环境变量**只有当前用户**可以访问。

系统环境通常有一些内置变量：`HOME`、`PATH`、`USER` 等。这些变量都有其对应的值，并不建议用户去直接替换他们。

例如：通常我们需要让所有登录用户能命令行运行应用程序，可以在 PATH 系统变量前面添加其执行路径：

``` bash
export PATH=/Users/app_folder:$PATH
```

如果设置了一个和系统环境变量同名的用户变量，那么`优先使用用户变量`。
