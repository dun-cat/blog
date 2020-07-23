---
layout: post
title: Docker的基本使用
date: 2020-06-19
tags: ["docker"]
---

### 镜像（ Image ）

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

### 容器（ Container ）

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
dokcer run —name hellowrold -d -p 12345:80 hello-world
# -name hellowrold 指定新容器的名称为 hellowrold。
# -d 后台运行
# -p 12345：80 把容器内部 12345 端口暴露给主机 80 端口。
# nginx:lastest 镜像名称

# 启动 hello-world 镜像的容器(启动已被停止的容器)
docker start hello-world

# 停止 hello-world 镜像的容器
docker stop hello-world 

# 移除 hello-world 镜像的容器
docker rm hello-world 
```

### Docker Compose 工具

Compose是一个用于定义和运行多容器 Docker 应用程序的工具，通过YAML文件配置。

#### Linux 系统下的安装
``` bash
curl -L https://github.com/docker/compose/releases/download/1.24.1/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
```
> Compose 的版本需要和 Docker 的版本一致。

延伸阅读：

\> [https://docs.docker.com/get-started/](https://docs.docker.com/get-started/)