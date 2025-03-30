## Docker 的基本使用 
### 镜像 ( Image )

#### 拉取镜像

从远程仓库拉取镜像。

语法如下：

``` bash
docker pull [OPTIONS] NAME[:TAG|@DIGEST]
```

例如从远程仓库拉取 nginx 镜像：

``` bash
docker pull nginx
```

> 具体使用文档，请查看[这里](https://docs.docker.com/engine/reference/commandline/pull/)。

#### 查看镜像

``` bash
# 查看镜像列表
docker images  
# or
doker image ls
# REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
# hello-world         latest              fce289e99eb9        7 months ago        1.84kB

# 通过镜像 ID 查看单个镜像详情
docker image inspect fce289e99eb9 
```

#### 创建镜像

创建镜像有两种方式：

1. 从已经创建的容器中更新镜像，并且提交这个镜像；
2. 使用 Dockerfile 指令来创建一个新的镜像；

``` bash
# 构建名称为 service-auth 版本为 1.11 的镜像
docker build . -t service-auth:1.11
```

#### 删除镜像

``` bash
# 删除单个镜像
docker image rm service-auth:1.11

# 删除未使用镜像
docker image prune -a
```

### 容器 ( Container )

#### 查看容器

``` bash
# 查看正在运行的容器列表
docker ps 
docker container ls 
docker container ls --all
docker container ls -aq
```

#### 容器的启动、停止、移除

``` bash
# 在新的容器运行``
docker run hello-world # 简单的启动镜像
dokcer run —-name hellowrold -d -p 12345:80 hello-world
# --name hellowrold 指定新容器的名称为 hellowrold。
# -d 后台运行
# -p 12345：80 把容器内部 12345 端口暴露给主机 80 端口。
# hello-world 镜像名称

# 启动 hello-world 镜像的容器(启动已被停止的容器)
docker start hello-world

# 停止 hello-world 镜像的容器
docker stop hello-world 

# 移除 hello-world 镜像的容器
docker rm hello-world 
```

#### 查看启动容器日志

``` bash
docker logs root_commento_1
```

* `root_commento_1` ：容器名称

#### 进入正在运行的容器

``` bash
docker exec -it postgresql /bin/bash
```

* `exec` ：执行
* `-it`：以交互的方式
* `postgresql` ：运行的容器名称或容器ID
* `/bin/bash`：执行的shell

#### 退出容器

``` bash
exit
```

### Docker Compose 工具

Compose是一个用于定义和运行多容器 Docker 应用程序的工具，通过YAML文件配置。

#### Linux 系统下的安装

``` bash
curl -L https://github.com/docker/compose/releases/download/1.24.1/docker-compose- `uname -s` - `uname -m` -o /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
```

> Compose 的版本需要和 Docker 的版本一致。

#### Docker Compose 的运行

``` bash
docker-compose -f docker-compose.demo.yml up -d 
```

* `up` ：启动
* `-d`：在后台运行
* `-f`：指定运行配置文件，无指定时`默认：docker-compose.yml`

### 常见操作

容器文件复制到 host 主机：

``` bash
docker cp yourContainerName:/dir /dir
```

参考资料：

\> [https://docs.docker.com/get-started/](https://docs.docker.com/get-started/)
