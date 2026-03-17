## Nginx 之基本知识 
### 介绍

Nginx 运行态由一个`主进程` (master process) 和多个`工作进程` (worker processes) 组成。

`主进程`的工作是读取和评估配置并维护 worker 进程。

真正的请求处理在 `worker 进程` 上进行，nginx 雇佣基于`事件模型` (event-based model) 和 `OS-dependent` 机制去高效地在 worker 进程之间分发请求。

Worker 进程的`个数`是定义在配置文件里。可以是固定数，也可以是由 CPU 核数来决定。

Nginx 和它的模块 (modules) 的工作方式取决于配置文件，默认 nginx 的配置文件名叫 `nginx.conf` 。

默认安装，配置文件在这几个位置之一：`/usr/local/nginx/conf`、`/etc/nginx`或`/usr/local/etc/nginx`。

### 模块

Nginx 是一个高性能的 HTTP 和反向代理服务器，它支持模块化的架构，可以通过添加不同类型的模块来扩展其功能。

以下是一些常见的 Nginx 模块及其功能的简介：

1. `Core 模块`：Nginx 的核心模块提供了基本的 HTTP 服务器功能，包括请求处理、配置解析、日志记录等。这些模块在 Nginx 中都是必不可少的。
2. `HTTP 模块`：HTTP 模块包括一系列模块，用于处理 HTTP 请求和响应。一些常见的HTTP模块包括：
   * `http_ssl_module`：添加 SSL/TLS 支持，用于加密HTTP连接。
   * `http_gzip_module`：用于压缩HTTP响应以提高性能。
   * `http_rewrite_module`：用于 URL 重写和重定向。
   * `http_auth_basic_module`：支持 HTTP 基本认证，用于访问控制。
3. `Security 模块`：这些模块用于增强 Nginx 的安全性，包括：
   * `http_secure_link_module`：用于生成和验证安全链接，以限制对资源的访问。
   * `http_limit_req_module`：用于限制请求速率以防止滥用。
   * `http_limit_conn_module`：用于限制连接数，以保护服务器免受连接耗尽攻击。
4. `Third-party 模块`：Nginx 支持`第三方模块`，这些模块由社区或其他开发者开发和维护，用于添加各种功能。一个广泛使用的第三方模块是 RTMP 模块，用于支持 RTMP 流媒体。
5. `反向代理和负载均衡模块`：Nginx 可以用作反向代理服务器，这些模块包括：
   * `http_proxy_module`：用于反向代理 HTTP 请求。
   * `http_upstream_module`：用于定义后端服务器组，支持负载均衡。
6. `缓存模块`：这些模块用于缓存 HTTP 响应，提高性能和减少负载：
   * `http_proxy_cache_module`：支持 HTTP 响应缓存。
   * `http_fastcgi_module`：支持 FastCGI 缓存。
7. `Lua 模块`：Nginx 提供了 Lua 模块，允许使用Lua脚本来扩展Nginx 的功能，进行高级的请求处理和响应生成。
8. `WebDAV 模块`：支持 WebDAV（Web-based Distributed Authoring and Versioning）协议，用于创建文件共享和协作环境。

这只是一小部分 Nginx 模块的示例。Nginx 的模块化架构允许根据需要添加和配置模块，以满足各种需求，从基本的 HTTP 服务器功能到反向代理、负载均衡、安全性、缓存等各种需求。根据你的项目需求，可以选择安装并配置适当的模块。

### 常用命令

#### 应用 conf 文件改动

``` bash
nginx -s reload
```

一旦主进程接收到`信号` (signal) 去重载配置，它会去校验新配置的语法正确性并尝试去应用配置。

如果应用配置成功，`主进程`会启动新的 worker 进程并且发送消息给老的 worker 进程，请求关闭他们。否则，主进程回滚改变并以老的配置继续工作。

老的 worker 进程接收一个关闭命令，会停止接收新的连接并继续服务当前的请求，直到所有请求被处理完之后退出。

#### 退出\启动\重载

Nginx 支持以下几个信号值：

* `stop` ：快速关闭 (fast shutdown) ；
* `quit` ：优雅关闭 (graceful shutdown) ；
* `reload` ：重新加载配置文件；
* `reopen` ：重开日志文件。

你可以通过下面的命令关闭 nginx：

``` bash
nginx -s quit
```

发送给 nginx 的一个`信号` (signal) 可能来至 Unix 工具，像 `kill` 套件。

如下面这样直接发送信号给一个给定的进程 ID (process ID) ：

``` bash
kill -s QUIT 1628
```

Nginx 主进程 ID 会被写入文件，默认 `nginx.pid` 在`/usr/local/nginx/logs`或`/var/run`目录。

通过下面的命令，你可以获取所有 nginx 的进程：

``` bash
ps -ax | grep nginx
```


参考资料：

\> [https://nginx.org/en/docs/beginners_guide.html#control](https://nginx.org/en/docs/beginners_guide.html#control)

\> [https://www.lawinsider.com/dictionary/operating-system-dependent](https://www.lawinsider.com/dictionary/operating-system-dependent)
