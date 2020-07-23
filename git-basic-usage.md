---
layout: post
title: Git基本使用
date: 2017-11-20
tags: ["git","其它","命令"]
---

### 基本流程

![git_base](git_base.png)

### 基本命令

#### pull

从服务器更新
``` bash
git pull
```
#### add

这是个多功能命令，根据目标文件的状态不同，此命令的效果也不同：

*   可以用它开始跟踪新文件
*   把已跟踪的文件放到暂存区
*   还能用于合并时把有冲突的文件标记为已解决状态
``` bash
git add . # 新建文件的时候，用于跟踪当前目录下的所有文件（除去.gitignore）
```
#### commit

提交暂存区里的文件到本地仓库。
``` bash
git commit -m '我是commit附带的消息'
git commit --amend --no-edit  #把这次改动合并到上一次commit，并且不修改提交信息。
```
#### push

*   提交本地仓库到远程服务器
*   提交分支到远程服务器
*   删除远程分支

``` bash
git push

git push origin dev # 提交本地分支dev到远程分支dev，假如远程分支dev不存在就会创建远程分支dev

git push origin dev:dev 一样的效果，[本地分支]:[远程分支] ，远程分支可重命名。 # 上面的写法是简写，全写如下

git push origin --delete hotfix # 删除远程hotfix分支
```

#### status

查看当前文件状态。
``` bash
    git status
```

#### checkout

多功能命令：

*   切换分支
*   切换HEAD到指定的commit
``` bash
git checkout somebranch

git checkout -b somebranch  # 创建 somebranch 分支并切换到 somebranch 分支

git checkout HEAD^ #切换HEAD到上一个commit。
```
#### branch

查看分支、创建分支、删除分支
``` bash
git branch # 查看本地分支

git branch -r # 查看远程分支

git branch -a # 查看本地和远程分支

git branch dev # 创建本地dev分支

git branch -d feature-001 # 删除本地feature-001分支
```
#### tag

给当前分支打标签
``` bash
git tag v1.0
```
合并分支
``` bash
#### merge
git merge hotfix # 合并hotfix 分支到当前分支去
```

### 命令的简写
* gco = git checkout
* gcmsg = git commit -m
* gp = git push
* gl = git pull
* gm = git merge
* ga =  git add
* gb = git branch
    