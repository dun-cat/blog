## 非对称加密及 Keytool 工具 
### 介绍

在`密码学`里的非对称加密 (Asymmetric cryptography) 被称为`公开密钥加密` (Public-key cryptography) ，由于`加密`和`解密`需要两个不同的密钥，故被称为非对称加密。

它需要两个密钥:

* 一个是`公开密钥` (public key) ，简称`公钥`。公钥向公众`公开`，谁都可以使用；
* 一个是`私有密钥` (private key) ，简称`私钥`。`私钥不可以公开`，必须由用户自行严格秘密保管，绝不透过任何途径向任何人提供，也不会透露给被信任的要通信的另一方。

### 两种加密方式

1. 若通过`公钥加密`，则需要通过`私钥解密`；
2. 若通过`私钥加密`，则需要通过`公钥解密`。

#### 公钥加密

在网上购物支付或使用网上银行时，输入敏感信息，都会通过`公钥加密`把数据上传至被信任的网站服务器，然后由网站服务器通过`私钥解密`，来获取正确信息。只有被信任的网站才能解密，所以不用担心数据在传输过程中被窃取而发生泄密。

#### 私钥加密

如果某一用户使用他的`私钥加密`明文，任何人都可以用该用户的`公钥解密。

由于私钥只由该用户自己持有，故可以肯定该文件必定出自于该用户。公众可以验证该用户发布的数据或文件`是否完整`、`是否曾被篡改`，接收者可信赖这些数据、文件确实来自于该用户，这被称作`数字签名`。

### 常见用例

#### 应用签名

`应用签名`：通常作为一款 App 的开发者，要防止自己的 App 被别人篡改，就需要`通过私钥对 App 文件做签名`，而后需要把我们的公钥 (证书的形式) 提供给应用市场。那在安装 App 的时候，应用市场就可以通过证书校验 App 是否来自开发者本人以及是否被篡改过。

#### SSH 传输

 `SSH` ：如果你是一位服务器管理员或者你使用 SSH 来作为 Git 仓库的传输方式，`私钥加密传输`就会是家常。

 `ssh-keygen` 是常用的 SSH 客户端工具，你可以通过它生成密钥文件：

``` bash
➜ ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/Users/lumin/.ssh/id_rsa): demo
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in demo.
Your public key has been saved in demo.pub.
The key fingerprint is:
SHA256:+uAtNAB1kl4BECEU7pgfC/pZY5xylZzji35ujTcBVSg lumin@MacBook-Pro-3.local
The key's randomart image is:
+---[RSA 3072]----+
|oo==+oo. o.      |
|... .oE o        |
| . o . o         |
|o.  o..o         |
|+..  .*.S        |
|.o + +oo.        |
|. + B.++ .       |
| . * +==+        |
|  o.o++oo.       |
+----[SHA256]-----+
```

上面会在当前工作目录生成 `demo` 的私钥文件，以及公钥文件 `demo.pub` 。

如果你没有指定密钥文件名，通常在你的用户目录下有一个`.ssh`目录：

``` bash
➜  .ssh ls
config      id_rsa      id_rsa.pub  known_hosts
```

这里保存了你的私钥文件 `id_rsa` 以及公钥文件 `id_rsa.pub`，只需要把本地的公钥文件上传至服务器上，那么就可以不再使用任何口令，而直接访问服务器。

### keytool 工具的使用

个人在工作中开发过 Android、快应用、华为手表、小米手表等平台应用，也做了一些 Linux 的基本运维，发现大部分关于密钥和证书的管理都通过 keytool 工具来完成。所以对于该工具，是有学习的必要性。

keytool 是 Java 工具，意味着你需要 Java 运行环境。如果你安装了 JDK ，那么可以在其安装目录

参考资料：

\> [https://zh.wikipedia.org/wiki/公开密钥加密](https://zh.wikipedia.org/wiki/公开密钥加密)
