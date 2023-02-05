## Git 分支管理之清除 
### 简介

对于分布式的 git，远程分支多了，不利于接管的开发人员立即投入开发。而在公司自动化不是很完善的时候，手动处理分支变得家常。

### 基本命令

#### 删除远程分支

``` bash
git push origin --delete hotfix # 删除远程hotfix分支
```

> 本地对应的分支还是会存在的。

#### 删除本地分支

``` bash
git branch -d hotfix # 删除本地hotfix分支
```

### 日常操作

#### 查看远程已删除的分支

``` bash
git remote prune origin --dry-run

# 输出
Pruning origin
URL: http://gi.xxx.com/xxx.git
 * [would prune] origin/demo
```

#### 清理已删除的远程分支的引用

``` bash
git remote prune origin

# 输出
Pruning origin
URL: http://gi.xxx.com/xxx.git
 * [pruned] origin/demo
```

> 并不会清除本地已存在的该分支

#### 查看已无法追踪的本地分支

``` bash
git branch -vv

# 输出
... ...
demo 92e944d [origin/demo: gone] Merge branch 'xxx/1.0.0' into 'master'
... ...
```

> 已失效的远程追踪分支会有 `gone` 标识

#### 删除无法追踪的本地分支

这里我们会配合本地分支删除命令，做多个无效的本地分支删除。

会分为三步：

1. `git branch --merged` ：列出已经合并到`当前分支`的分支列表，确保删除安全；
2. `egrep -v "(^\*|master|dev)"` ：排除不想删除的关键分支；
3. `xargs git branch -d` ：执行多个删除操作；

``` bash
git branch --merged | egrep -v "(^\*|master|rollback)" | xargs git branch -d

# 输出
Deleted branch demo (was 92e944d).
```
