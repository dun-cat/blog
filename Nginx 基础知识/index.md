## Nginx 基础知识 
### 前言

Nginx 运行态由一个`主进程`（master process）和多个`工作进程`（worker processes）组成。

`主进程`的工作是读取和评估配置并维护 worker 进程。

真正的请求处理在`worker 进程`上进行，nginx 雇佣基于`事件模型`（event-based model）和`OS-dependent`机制去高效地在 worker 进程之间分发请求。

Worker 进程的`个数`是定义在配置文件里。可以是固定数，也可以是由 CPU 核数来决定。

Nginx 和它的模块（modules）的工作方式取决于配置文件，默认 nginx 的配置文件名叫`nginx.conf`。

默认安装，配置文件在这几个位置之一：`/usr/local/nginx/conf`、`/etc/nginx`或`/usr/local/etc/nginx`。

### 常用命令

#### 应用 conf 文件改动

``` bash
nginx -s reload
```

一旦主进程接收到`信号`（signal）去重载配置，它会去校验新配置的语法正确性并尝试去应用配置。

如果应用成功，主进程会启动新的 worker 进程并且发送消息给老的 worker 进程，请求关闭他们。否则，主进程回滚改变并以老的配置继续工作。

老的 worker 进程接收一个关闭命令，会停止接收新的连接并继续服务当前的请求，直到所有请求被处理完之后退出。

#### 退出\启动\重载

Nginx 支持以下几个信号值：

* `stop`：快速关闭（fast shutdown）；
* `quit`：优雅关闭（graceful shutdown）；
* `reload`：重新加载配置文件；
* `reopen`：重开日志文件。

你可以通过下面的命令关闭 nginx：

``` bash
nginx -s quit
```

发送给 nginx 的一个`信号`（signal）可能来至 Unix 工具，像`kill`套件。

如下面这样直接发送信号给一个给定的进程 ID（process ID）：

``` bash
kill -s QUIT 1628
```

Nginx 主进程 ID 会被写入文件，默认`nginx.pid`在`/usr/local/nginx/logs`或`/var/run`目录。

通过下面的命令，你可以获取所有 nginx 的进程：

``` bash
ps -ax | grep nginx
```



参考文献：

\> [https://nginx.org/en/docs/beginners_guide.html#control](https://nginx.org/en/docs/beginners_guide.html#control)

\> [https://www.lawinsider.com/dictionary/operating-system-dependent](https://www.lawinsider.com/dictionary/operating-system-dependent)
