## OAuth 2.0 的授权设计 
### 简介

OAuth 2.0 是用于授权的行业标准协议，用来`授权第三方应用`，获取用户数据。该协议规范及其扩展正在 [IETF](https://tools.ietf.org/html/rfc6749) OAuth工作组内开发。

OAuth 代表`开放授权`，把应用的`私有数据`安全得开放给指定的第三方应用。

### 术语

* `资源拥有者（Resource Owner）`：资源所有者是授权应用程序访问其帐户的`用户`。应用程序对用户帐户的访问限于授权范围（Scope）（例如，读或写访问）。
* `客户端（Client）`：客户端是尝试访问用户帐户的应用程序。在访问帐户之前，它`需要获得用户的许可`。客户端可以向用户显示`登录页面`或`已登录账户的授权页`，以获取用于访问特定资源的`访问令牌`（access token）。
* `授权服务器（Authorization Server）`：授权服务器`验证用户凭据（credentials）`，并使用`授权码`（authorization code）将用户重定向回客户端。客户端与授权服务器进行通信以`确认其身份`，并将代码交换为访问令牌。
* `资源服务器（Resource Server）`：资源服务器是用于访问受保护的资源的服务器。它处理来自具有访问令牌的应用程序的经过身份验证的请求。
* `范围（Scope）`：指定应用程序从客户端请求的访问级别。
* `同意（Consent）`：`"同意"`屏幕告诉您的用户谁正在请求访问其数据以及您要访问哪种数据。

在了解上面的术语之后，就可以进一步探讨 OAuth 的授权类型。

### 5种不同的授权流程

* 授权码授权（Authorization Code Grant）
* 隐式授权（Implicit Grant）
* 资源所有者证书授权（Resource Owner Credentials Grant）
* 客户端证书授权（Client Credentials Grant）
* 刷新令牌授权（Refresh Token Grant）

#### 授权码授权

`故事`：用户尝试登录abc.com，但他忘记了密码，并且发现了一种使用google登录的选项，通过单击此选项，该用户将很容易使用google帐户登录。

##### 授权流程

[![oauth_1](oauth_1.svg)](oauth_1.svg)

延伸阅读：

\> [https://oauth.net/2/](https://oauth.net/2/)

\> [https://tools.ietf.org/html/rfc6749](https://tools.ietf.org/html/rfc6749)

\> [https://docs.github.com/en/developers/apps/scopes-for-oauth-apps](https://docs.github.com/en/developers/apps/scopes-for-oauth-apps)

\> [https://www.loginradius.com/blog/async/oauth2/](https://www.loginradius.com/blog/async/oauth2/)