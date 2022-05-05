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

在了解上面的术语之后，就可以进一步探讨 OAuth 的授权类型，各种授权的最终目的是获取`访问令牌`（access token）来获取资源。

### 五种不同的授权流程

* 授权码授权（Authorization Code Grant）
* 隐式授权（Implicit Grant）
* 资源所有者证书授权（Resource Owner Credentials Grant）
* 客户端证书授权（Client Credentials Grant）
* 刷新令牌授权（Refresh Token Grant）

#### 授权码授权

这种是较为完备的授权方式，在[RFC-6749](https://tools.ietf.org/html/rfc6749#section-4.1)中，解释了它用于同时获取 `访问令牌`（access token）和`刷新令牌`（refresh token）。在`访问令牌`失效后，通过`刷新令牌`重新获取`访问令牌`。

该模式整体流程：

1. 客户端发起授权请求，用户允许授权客户端后，会拉起客户端应用或重定向到客户端网站，并且带上授权临时票据 code（授权码）参数；
2. 通过 code 参数加上 client_id 和 client_secret 等，通过 API 换取 access_token；
3. 通过 access_token 进行接口调用，获取用户基本数据资源或帮助用户实现基本操作；

他的授权流程如下：

![oauth_1.svg](oauth_1.svg)

> 这里的客户端（Client）并非指的浏览器，指代基于浏览器的 web 应用，通常把一个 server 端（后台）作为代理和授权服务器进行授权认证。

关键参数解释：

* `client_id` 和 `client_secret`： 由`授权服务器`生成，并且需要预先在`授权服务器`做备案。一旦生成基本保持固定值，通过它来确认客户端资质；
* `scope`：确定授权范围；
* `state`：用于保持请求和回调的状态，授权请求后原样带回给客户端。该参数可用于`防止csrf攻击`（跨站请求伪造攻击），建议用户端（例如：浏览器）带上该参数，可设置为简单的随机数加 session 进行校验；
* `code`：授权码（authorization code），用于获取 access_token 的临时凭证。为了安全起见，会有`比较短的失效期`和`只能使用一次`的限制，时效时间通常设为几分钟。

`client_id` 定义为客户端标识（Client Identifier），授权服务器颁发给已注册客户端的唯一身份标识；它表示了客户端提供的身份信息并且无需加密；它暴露给资源拥有者，并且**不允许**单独被用于客户端授权。

#### 隐式授权

`故事`：在此流程中，客户端直接获得 token，而无需执行额外的授权代码交换步骤，就能访问资源服务器上的资源。

他的授权流程如下：
![oauth_2.svg](oauth_2.svg)

这种方式省去了通过 code 交换 access token 的流程。

#### 客户端证书授权

`故事`：客户端应用程序将其客户端凭据（客户端标识符和客户端密钥）提供给授权服务器，以请求批准访问资源服务器上的受保护资源（由客户端应用程序拥有）。授权服务器对客户端凭据进行身份验证并颁发 token。

它的流程如下：

![oauth_3.svg](oauth_3.svg)

此流程中,客户端可以仅使用其客户端凭据（或其他受支持的身份验证方法）获取 token。

#### 资源所有者证书授予

该授权需要`资源所有者`和`客户端`具有信任关系下进行，启动此类型要格外小心，仅在其他流程不可行时才允许它。

客户端将要求用户提供其授权凭证（通常是用户名和密码），该流程如下：

![oauth_4.svg](oauth_4.svg)

#### 刷新令牌授权

令牌如果过期，没必要重新走流程，OAuth 允许直接刷新令牌即可获得有效令牌。

它的具体流程如下：

![oauth_5.svg](oauth_5.svg)


### 遵循OAuth 2.0授权的应用

在这些授权类型中`授权码授权`的方式最为常用，像腾讯开放平台的[授权登录](https://developers.weixin.qq.com/doc/oplatform/Website_App/WeChat_Login/Wechat_Login.html)、钉钉的[免登流程](https://developers.dingtalk.com/document/app/logon-free-process)、Github的[授权登录](https://docs.github.com/cn/developers/apps/authorizing-oauth-apps)等等。

参考文献：

\> [https://oauth.net/2/](https://oauth.net/2/)

\> [https://tools.ietf.org/html/rfc6749](https://tools.ietf.org/html/rfc6749)

\> [https://docs.github.com/en/developers/apps/scopes-for-oauth-apps](https://docs.github.com/en/developers/apps/scopes-for-oauth-apps)

\> [https://www.loginradius.com/blog/async/oauth2/](https://www.loginradius.com/blog/async/oauth2/)

\> [http://www.ruanyifeng.com/blog/2019/04/oauth-grant-types.html](http://www.ruanyifeng.com/blog/2019/04/oauth-grant-types.html)

\> [https://developers.weixin.qq.com/doc/oplatform/Website_App/WeChat_Login/Wechat_Login.html](https://developers.weixin.qq.com/doc/oplatform/Website_App/WeChat_Login/Wechat_Login.html)