## 如何在 Github Actions 里通过 SSH 向其它仓库进行推送？ 
### How

整个工作流程：在`源仓库` 的 action 里向`目标仓库` 推送源仓库内容。

在 Github 仓库访问的场景中，通常是把`公钥`提供给远程服务站点。而本地持有`私钥`来做身份验证，在这里源仓库的 action 环境就是本地。

所以，实际整个工作原理就是把 `action` 执行环境作为`本地主机`和`目标仓库`作为`远程主机`，然后正常按 SSH 方式进行访问即可。

### 步骤

#### 1.生成密钥对

那么先用 `ssh-keygen` 生成密钥对。

``` bash
ssh-keygen -t ed25519 -C "xxx@qq.com"  # 生成密钥对（公钥和私钥）
```

``` bash
Generating public/private ed25519 key pair.
Enter file in which to save the key (/Users/lumin/.ssh/id_ed25519): github_action
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in github_action
Your public key has been saved in github_action.pub
The key fingerprint is:
SHA256:32jjXgNWjYXbKLUXt0h82vrtyhDePP5l2AdGrF6iido xxx@qq.com
The key's randomart image is:
+--[ED25519 256]--+
|            ...  |
|            oB...|
|           .+=Oo.|
|          ..+=oo |
|        S o.+.=  |
|         + O O + |
|        . B B * *|
|       o o o = ++|
|      . E.o   ++o|
+----[SHA256]-----+
```

这里设置文件名时，输入 `github_action`，因此会在`当前目录`生成私钥文件 `github_action` 和公钥文件 `github_action.pub`。如果默认回车则存储到 `~/.ssh` 目录下。

#### 2.目标仓库配置 Deploy Key

1. 进入目标仓库的 **Settings** → **Deploy keys** → **Add deploy key**。
2. 填写：
   - **Title**: `ACTIONS_DEPLOY_KEY`
   - **Key**: 粘贴生成的公钥（`github_action.pub` 文件内容）
   - 勾选 **Allow write access**（否则无法提交）。

#### 3.源仓库配置 Secrets

1. 进入源仓库的 **Settings** → **Secrets and variables** → **Actions** → **New repository secret**。
2. 添加两个 Secrets：
   - **Name**: `SSH_PRIVATE_KEY` → **Value**: 私钥内容（`github_action` 文件内容）。
   - **Name**: `TARGET_REPO_SSH` → **Value**: 目标仓库的 SSH 地址（如 `git@github.com:user/target-repo.git`）。

#### 4.编写 GitHub Actions 工作流

```yaml
name: Push to Another Repository via SSH

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout source code
        uses: actions/checkout@v4

      - name: Generate files
        run: |
          mkdir -p output
          echo "Hello, Target Repo!" > output/file.txt

      - name: Configure SSH
        run: |
          mkdir -p ~/.ssh
          echo "${{ secrets.SSH_PRIVATE_KEY }}" > ~/.ssh/github_action
          chmod 600 ~/.ssh/github_action
          ssh-keyscan github.com >> ~/.ssh/known_hosts

      - name: Configure Git
        run: |
          git config --global user.name "GitHub Actions"
          git config --global user.email "xxx@qq.com"

      - name: Push to target repository
        run: |
          cd output
          git init
          git remote add origin ${{ secrets.TARGET_REPO_SSH }}
          git add -A
          git commit -m "Update files via GitHub Actions"
          git push -f origin main  # 强制推送（根据需求调整分支）
```