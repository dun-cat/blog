## Linux 的 APT 命令 
APT (Advanced Package Tool) 是 Linux 上高级包管理工具，用于 Ubuntu 或 Debian Linux 系统上的软件包。

### apt 和 apt-get

`apt` 和 `apt-get` 都是用于管理 Debian 和 Ubuntu 系统上软件包的命令行工具。区别是：

1. apt：

* apt 它整合了多个命令，如 `apt-get`、`apt-cache` 和其他一些工具的功能，使其更统一。
* apt 支持`自动解决依赖关系`，`安装`、`升级`和`删除软件包`时通常不需要额外的选项。
* 使用 apt 可以更容易地搜索软件包、显示软件包信息、查看可用更新等。

2. apt-get：

* apt-get 是传统的 Debian 包管理工具，已经存在较长时间。它提供了精确的控制和更多选项。
* apt-get 命令通常需要额外的选项来执行特定的任务，如`更新软件包列表`、`安装软件包`、`升级系统`等。

以下是一些常见的 apt-get 操作：

* 安装软件包：`sudo apt-get install package-name`
* 升级已安装的软件包：`sudo apt-get upgrade`
* 升级系统中的所有软件包（包括内核）：`sudo apt-get dist-upgrade`
* 移除软件包：`sudo apt-get remove package-name`
* 移除软件包及其配置文件：`sudo apt-get purge package-name`
* 更新软件包列表：`sudo apt-get update`
* 搜索可用软件包：`apt-cache search search-keyword`

### 配置文件

APT 的配置文件路径在 `/etc/apt/apt.conf` 或 `/etc/apt/apt.conf.d/` 目录中。

### sources.list 文件

`sources.list` 文件用于配置软件包管理系统（APT）的源列表配置文件。这个文件包含了用于下载和安装软件包的`软件源`（repositories）的信息。在 sources.list 文件中，可以指定从哪些镜像源获取软件包以及哪些版本的软件包应该可用。

sources.list 文件通常位于 `/etc/apt/` 目录下。以下是一个典型的 `sources.list` 文件的示例：

``` text
deb http://archive.ubuntu.com/ubuntu/ bionic main restricted
deb-src http://archive.ubuntu.com/ubuntu/ bionic main restricted

deb http://archive.ubuntu.com/ubuntu/ bionic-updates main restricted
deb-src http://archive.ubuntu.com/ubuntu/ bionic-updates main restricted

deb http://archive.ubuntu.com/ubuntu/ bionic universe
deb-src http://archive.ubuntu.com/ubuntu/ bionic universe

deb http://archive.ubuntu.com/ubuntu/ bionic-updates universe
deb-src http://archive.ubuntu.com/ubuntu/ bionic-updates universe

```

上述示例包括了一些 `deb` 和 `deb-src` 条目，每个条目都指定了一个`软件源`和`软件包`组件。这些条目的含义如下：

* `deb`：表示要从指定的源下载`二进制软件包`。
* `deb-src`：表示要下载`源码软件包`，这些源码软件包可用于开发和构建软件包。

每个条目的结构如下：

* `http://archive.ubuntu.com/ubuntu/`：软件源的基本 URL。
* `bionic`：发行版本的代号，这里是 `Ubuntu 18.04` 的代号。
* `main`、`restricted`、`universe` 等：`软件包组件`，这些组件包含了不同类型的软件包。

您可以根据您的需要编辑 sources.list 文件，以包含所需的`软件源和组件`。然后，使用 `apt update` 命令来更新软件包列表，以便能够安装来自这些软件源的软件包。

> 请注意，对 sources.list 文件的编辑需要`超级用户权限`。在修改前，**确保备份文件**或小心操作，以避免损坏系统的软件包管理。

#### 组件

在 Linux 软件仓库（例如 Debian、Ubuntu 等）中，`组件`（components）是一种用于`组织软件包`的概念。

组件通常用于将软件包按照其自由软件许可证、版权、法律约束或其他特定要求进行分类和分组。每个组件代表一组软件包，而这些软件包在某种程度上共享相似的特征或限制。以下是一些常见的组件：

* Main（主要）：Main 组件包含了操作系统的核心组件和自由软件，这些软件遵守自由软件许可证（如GPL）并受到积极的维护和支持。这些软件包通常由Linux发行版的官方维护团队提供，并是系统的关键部分。

* Contrib（贡献）：Contrib 组件包含了一些自由软件，但它们可能依赖于非自由软件或受到一些限制。这些软件包通常不是官方维护的一部分，而是由社区或其他个人维护的。

* Non-Free（非自由）：Non-Free 组件包含了不遵守自由软件许可证的软件，可能包括专有或受限制的软件。这些软件包通常不建议使用，因为它们可能不符合自由软件原则。

* Backports（后端口）：Backports 组件包含了从较新版本的发行版中提取并适应较旧发行版的软件包。这允许用户在不升级整个系统的情况下使用较新的软件包。

* Multiverse（多元宇宙）：Multiverse 组件包含了非常不常见或专有的软件包，通常包含专有的代码和二进制文件。

不同的 Linux 发行版可能有不同的组件，而组件的名称和含义可能会有所不同。通过将软件包分成不同的组件，Linux 发行版能够满足各种用户需求，包括那些关注自由软件原则、法律要求或其他特殊需求的用户。用户可以根据他们的需求选择性地启用或禁用这些组件。

#### 多个源的优先级

每个软件源都有一个优先级，表示其重要性。APT 使用源的优先级来确定从哪个源获取软件包。更高优先级的源将优先考虑。源的优先级通常由 `/etc/apt/preferences` 文件中的配置规则来设置，或者由 APT 的内置规则来确定。

### 使用

#### 列出已安装的本地包

``` shell
apt list --installed
```

#### 搜索包

``` shell
apt search package-name
```

``` shell
apt-cache search package-name
```

### 包的公钥验证

包的公钥验证是一种确保下载和安装的软件包来自`受信任的源`并且`没有被篡改`的重要安全措施。这是通过 `GPG` (GNU Privacy Guard) 密钥进行的，`每个软件源`都有一个相关的公钥用于签署软件包。

{{% notice tip %}}
GPG 是一个用于`加密`、`解密`、`数字签名`和`管理加密密钥`的工具。GPG 通常用于`电子邮件加密`、`数字签名文件`、`保护文档`等安全通信任务。
{{% /notice %}}

以下是如何进行包的公钥验证的一般步骤：

1. `获取公钥`：要验证软件包的公钥，首先需要`获取软件源的公钥`。这通常可以通过官方网站或软件源提供的方式获取。
2. `导入公钥`：一旦获得了软件源的公钥，使用 `gpg` 命令将其导入系统的 GPG 密钥环中。通常，您可以使用以下命令导入公钥：
  
    ``` shell
    gpg --import /path/to/public-key.asc
    ```

    其中，`/path/to/public-key.asc` 是您下载的公钥文件的路径。

3. `验证软件包`：每当使用 `apt`、`apt-get` 或其他包管理工具安装软件包时，系统会自动使用已导入的公钥来验证软件包的签名。

    如果软件包的签名与相应的公钥匹配且未被篡改，安装将继续。

    如果软件包的签名无法验证或与公钥`不匹配，系统将阻止软件包的安装`，并显示错误消息。

4. `更新公钥`：定期更新已导入的公钥以确保安全性。您可以使用以下命令从密钥服务器获取并导入新的公钥：

   ``` shell
   gpg --recv-keys KEY-ID
   ```
