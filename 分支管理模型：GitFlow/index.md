## 分支管理模型：GitFlow 
### 时序图

[![gitflow](/img/gitflow-layout.png)](/img/gitflow-layout.png)

### 分支组成

主分支(master branch)
开发分支(develop branch)
若干个特性分支(feature branches)
若干个发布分支(release branches)
热修复分支(hotfix branches)

### 分支解读

远端 (origin) 分支只有：`master`和 `develop` 分支。其他分支本地管理，不 push。
master 分支和 develop 分支是长期分支，禁止删除。feature 分支、release 分支、hotfix 分支是辅助分支，有生命周期，可以在完成开发并且合并后可以删除。

#### 主分支(master)

已发布的稳定产品分支。

#### 开发分支(develop)

日常开发分支。

必须从 master分支签出，最后 merge 回到 master 分支。

#### 特性分支(feature branches)

用于开发一些新特性的分支。开发完成，合并 `develop` 分支后，需把 `develop` 分支 `push` 到远端 ( `origin/develop` ) 仓库。

可以从 develop 分支签出；
必须 merge 回到 develop 分支；
分支命名约定：除了 master, develop, release-*, or hotfix-*等分支名之外；

创建特性分支

``` bash
git checkout -b feature/fly develop # 从 develop 分支签出分支名为 feature/fly 的特性分支
Switched to a new branch "feature/fly"
```

开发完成后，merge 回到 develop 分支

``` bash
git checkout develop # 切换回开发分支
Switched to branch 'develop'

git merge --no-ff feature/fly # 不使用fast-forward方式合并，保留分支的commit历史
Updating ea1b82a..05e9557
(Summary of changes)

git branch -d feature/fly # 删除特性分支
Deleted branch feature/fly (was 05e9557).

git push origin develop # 提交到远程仓库
```

#### 发布分支(release branches)

为发布做准备，允许较小的 bug 修复和 meta 数据修改(版本号，构建日期等等)。

可以从 develop 分支签出；
必须 merge 回到 develop 和 master 分支；
分支命名约定：release-*；

创建发布分支

``` bash
git checkout -b release-1.2 develop # 从开发分支签出分支名为 release-1.2 的发布分支
Switched to a new branch "release-1.2"

./bump-version.sh 1.2
Files modified successfully, version bumped to 1.2.

git commit -a -m "Bumped version number to 1.2" # 连同暂存区的文件一起提交。
[release-1.2 74d9424] Bumped version number to 1.2
1 files changed, 1 insertions(+), 1 deletions(-)
```

开发完成后，merge 回到 master 分支

``` bash
git checkout master # 签出主分支
Switched to branch 'master'

git merge --no-ff release-1.2 # 合并
Merge made by recursive.
(Summary of changes)

git tag -a 1.2 # 打上版本标签
```

merge 回到 develop 分支

``` bash
git checkout develop # 签出开发分支
Switched to branch 'develop'

git merge --no-ff release-1.2 # 合并
Merge made by recursive.
(Summary of changes)

git branch -d release-1.2 # 删除发布分支
Deleted branch release-1.2 (was ff452fe).
```

#### 热修复分支(hotfix branches)

用于修复已发布的产品的问题。当需要立即修复已发布的产品 bug 时，签出 master 修复。

可以从 master 分支签出；
必须 merge 回到 develop  和 master 分支；
分支命名约定：hotfix-*；

创建热修复分支

``` bash
git checkout -b hotfix-1.2.1 master # 从主分支创建热修复分支
Switched to a new branch "hotfix-1.2.1"

./bump-version.sh 1.2.1 # 修改版本号
Files modified successfully, version bumped to 1.2.1.

git commit -a -m "Bumped version number to 1.2.1" # 提交
[hotfix-1.2.1 41e61bb] Bumped version number to 1.2.1
1 files changed, 1 insertions(+), 1 deletions(-)
```

修复 bug 后，merge 回到 master 分支

``` bash
git commit -m "Fixed severe production problem" # 提交

git checkout master # 签出主分支
Switched to branch 'master'

git merge --no-ff hotfix-1.2.1 #  合并
Merge made by recursive.
(Summary of changes)

git tag -a 1.2.1 # 打标签
```

修复 bug 后，merge 回到 develop 分支

``` bash
$ git checkout develop #签出开发分支
Switched to branch 'develop'

$ git merge --no-ff hotfix-1.2.1 # 合并
Merge made by recursive.
(Summary of changes)

$ git branch -d hotfix-1.2.1 # 删除
Deleted branch hotfix-1.2.1 (was abbe5d6).
```

release 分支、 hotfix 分支、feature 分支命名可以是：
release-_or release/_ 、 hotfix-_or hotfix/_  、 feature-_or feature/_

参考资料：

\> [https://nvie.com/posts/a-successful-git-branching-model/](https://nvie.com/posts/a-successful-git-branching-model/)

\> [https://yq.aliyun.com/articles/573549?utm_content=m_45538](https://yq.aliyun.com/articles/573549?utm_content=m_45538)
