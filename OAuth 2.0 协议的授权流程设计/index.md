## OAuth 2.0 协议的授权流程设计 
### 简介

OAuth 2.0 是用于授权的行业标准协议，用来`授权第三方应用`，获取用户数据。该协议规范及其扩展正在 [IETF](https://tools.ietf.org/html/rfc6749) OAuth 工作组内开发。

OAuth 代表`开放授权`，把应用的`私有数据`安全得开放给指定的第三方应用。

### 术语

* **资源所有者 (Resource Owner)**：资源所有者是授权应用程序访问其帐户的`用户`。应用程序对用户帐户的访问限于授权范围 (Scope) (例如：读或写访问) 。
* **客户端 (Client)**：使用资源所有者的授权代表资源所有者发起对受保护资源的请求的`应用程序`。术语“客户端”**并非**特指任何特定的实现 (例如：应用程序是否在服务器、台式机或其他设备上执行)。
* **授权服务器 (Authorization Server)**：授权服务器`验证用户凭据 (credentials)`，并使用`授权码` (authorization code) 将用户重定向回客户端。客户端与授权服务器进行通信以`确认其身份`，并将代码交换为访问令牌。
* **资源服务器 (Resource Server)**：资源服务器是用于访问受保护的资源的服务器。它处理来自具有访问令牌的应用程序的经过身份验证的请求。
* **范围 (Scope)**：指定应用程序从客户端请求的访问级别。
* **同意 (Consent)**：`"同意"`屏幕告诉您的用户谁正在请求访问其数据以及您要访问哪种数据。
* **刷新令牌 (Refresh Token)**：当访问令牌失效后，可以通过刷新令牌来重新获取访问令牌。因此，其有效期要比访问令牌长久。

在了解上面的术语之后，就可以进一步探讨 OAuth 的授权类型，各种授权的最终目的是获取`访问令牌` (access token) 来获取资源。

### OAuth 的设计目标

OAuth 是基于下面三种应用程序类型设计的：

* **Web 应用**：Web 应用是在 `Web 服务器`上运行的`受信客户端`。客户端凭据以及颁发给客户端的任何 Access Token 都`存储在 Web 服务器`上，资源所有者`不会公开`或`访问`它们。

  例：运行在`服务器端`的 Web 应用。

* **基于用户代理的应用**：基于用户代理(常见如：浏览器)的应用程序是一个`公共客户端`，其中客户端代码从 Web 服务器下载，并在`资源所有者使用的设备`上的用户代理（例如 Web 浏览器）中执行。

  例：运行在`浏览器`上的网页应用。比如，可以通过已登录的 Github 站点授权登录并获取个人 Github 的基本信息 (昵称、头像等等)。

* **Native 应用**：Native 应用是在资源所有者使用的设备上安装和执行的`公共客户端`。

  例：运行在`系统设备`的上的原生应用。比如，移动 App 应用、桌面端原生程序等等。

![oauth2-3.svg](oauth2-3.svg)

所以，OAuth2 里的`客户端`术语指的也是这三类应用。

### 注册客户端

在启用 OAuth 的授权协议之前，需要客户端提前进行注册。这是一个关键步骤，主要目的是确保客户端应用能够安全地与授权服务器进行交互。

客户端开发人员需要：

1. 指定`客户端类型`；
2. 提供`客户端重定向 URI`，以及包括授权服务器所需的任何其他信息(例如，应用程序名称、网站、描述、徽标图像、法律条款的接受程度等等)。

客户端在授权服务器注册后，需要颁发分配一个唯一的`客户端 ID` 和`客户端密钥`作为凭证。在 OAuth2 里规定了分别用 `client_id` 和 `client_secret` 名称表示。

这些凭据在与授权服务器交互时用于验证客户端的身份，确保只有合法的客户端能够请求访问令牌和受保护资源。

注册客户端的典型流程：

1. **开发者注册**：开发者在授权服务器上注册其应用程序。提供必要的信息，如应用名称、描述、开发者联系方式等。
2. **指定重定向 URI**：开发者提供一个或多个`重定向 URI`，这些 URI 将用于处理授权服务器返回的授权码或令牌。
3. **获取客户端凭据**：授权服务器生成并分配一个唯一的`客户端 ID` 和`客户端密钥`。这些凭据需要安全地存储在客户端应用中。
4. **配置权限范围和其他参数**：开发者可以定义应用程序请求的`权限范围`（scopes）。其他配置选项可能包括访问令牌的生命周期、支持的授权类型等。

#### 客户端的类型

OAuth 协议根据`客户端的机密性`分了两种类型：

**机密的 (confidential)**：这类的客户端运行在安全服务器上的。例如：一个服务器端的 Web 应用程序。

**公共的 (public)**：这类客户端无法对密钥进行保密。例如：一个 Web 应用程序运行在个人电脑上，那么这个 Web 应用程序的密钥安全性就无法保障。

### 四种不同的授权流程

* 授权码授权 (Authorization Code Grant)
* 隐式授权 (Implicit Grant)
* 资源所有者密码凭据授予 (Resource Owner Password Credentials Grant)
* 客户端凭证授权 (Client Credentials Grant)

#### 授权码授权

由于这是一种`基于重定向` (redirection-based) 的流程，因此`客户端`**必须**能够与资源所有者的`用户代理`（通常是 `Web 浏览器`）进行交互。

在 [RFC-6749](https://tools.ietf.org/html/rfc6749#section-4.1) 中，说明了它用于获取`访问令牌` (access token) 和`刷新令牌` (refresh token)。

该模式整体流程：

1. 用户代理请求授权服务器授权，授权服务器返回 Authorization Code；
2. 用户代理重定向到客户端，客户端请求授权服务器获取 Access Token，授权服务器返回 Access Token；

他的授权流程如下：

![oauth_1.svg](oauth_1.svg)

关键参数解释：

* **client_id** 和 **client_secret** ： 由`授权服务器`生成，且需预先在`授权服务器`进行注册。一旦生成基本保持固定值，通过它来确认客户端资质；
* **scope**：授权范围；
* **state**：不可猜测的随机字符串，用于防止`跨站请求伪造 (CSRF)` 攻击。
* **code**：授权码 (authorization code)，用于获取 access_token 的`临时凭证`。为了安全起见，会有`比较短的失效期`和**只能使用一次**的限制，时效时间通常设为`几分钟`。

授权码授权方式的好处是用户可以在授权的过程中参与进来，手动决定是否授权给客户端。如果不信任客户端可以阻断 Access Token 的发放。

##### 实现步骤

1. `重定向到授权端点`。用户代理重定向到授权服务器的端点，客户端指示用户代理使用 TLS 发出以下 HTTP 请求：

    ``` shell
    GET /authorize?response_type=code&client_id=s6BhdRkqt3&state=xyz&redirect_uri=https%3A%2F%2Fclient%2Eabc%2Ecom%2Fcallback HTTP/1.1
    Host: server.abc.com
    ```

2.  `资源所有者身份验证及授予`。如果资源所有者授予访问请求，则授权服务器会发出`授权码`。授权服务器通过发送以下 HTTP 响应来重定向用户代理：

    ``` shell
    HTTP/1.1 302 Found
    Location: https://client.abc.com/callback?code=SplxlOBeZQQYbYS6WxSbIA&state=xyz
    ```

    > Ajax 通过 JavaScript 在客户端发起的异步请求，它**不会**自动处理重定向。

3. `用户代理重定向`。用户代理通过 HTTP Location 重定向 URI 到客户端端点 (endpoint) 上。
4. `客户端请求 Access Token`。客户端发送下面的 `POST` 请求来获取 Access Token。

    ``` shell
    POST /token HTTP/1.1
    Host: server.abc.com
    Authorization: Basic czZCaGRSa3F0MzpnWDFmQmF0M2JW
    Content-Type: application/x-www-form-urlencoded

    client_id=s6BhdRkqt3&
    client_secret=7Fjfp0ZBr1KtDRbnfVdmIw&
    grant_type=authorization_code&
    code=SplxlOBeZQQYbYS6WxSbIA&
    redirect_uri=https%3A%2F%2Fclient%2Eabc%2Ecom%2Fcallback
    ```

    请求响应：

    ``` shell
    HTTP/1.1 200 OK
    Content-Type: application/json;charset=UTF-8
    Cache-Control: no-store
    Pragma: no-cache

    {
      "access_token":"2YotnFZFEjr1zCsicMWpAA",
      "token_type":"Bearer",
      "expires_in":3600,
      "refresh_token":"tGzv3JOkF0XG5Qx2TlKWIA",
      "example_parameter":"example_value"
    }
    ```

##### 分析

授权码授权的`客户端`类型是`机密的` (confidential)，因此被授权的第三方应用需要有自己的后台。也正是如此，可以`更安全`得获取 Access Token。

授权服务器发放临时凭证(授权码)给客户端，来验证用户代理发送给客户端请求的合法性。

这也是目前大部分互联网应用使用的授权方式，通常它们都有自己的后台。

#### 隐式授权

隐式授权的客户端通常是使用`脚本语言` (如 JavaScript) 在`浏览器`中实现的应用。

由于这是基于重定向的流，因此客户端**必须**能够与资源所有者的`用户代理` (通常是 Web 浏览器) 进行交互，并且能够接收来自授权服务器的传入请求（通过重定向）。

他的授权流程如下：
![oauth_2.svg](oauth_2.svg)

这种方式不会颁发中间凭证 (例：授权码)。在隐式授权流期间颁发访问令牌时，授权服务器**不会**对客户端进行`身份验证`，所以它是`公共类型`的客户端。

因为隐式授权被认为`太不安全`，所以授权服务器**不支持**颁发 Refresh Token。

##### 实现步骤

1. `重定向到授权端点`。使用 TLS 通过 Get 请求发送，来获取包含 Access Token 的重定向 URI。

    ```shell
    GET /authorize?response_type=token&client_id=s6BhdRkqt3&state=xyz&redirect_uri=https%3A%2F%2Fclient%2Eabc%2Ecom%2Fcallback HTTP/1.1
    Host: server.abc.com
    ```

    授权服务器验证请求，以确保所有必需的参数都存在且有效。其中，**必须验证** `redirect_uri` 是否与客户端注册的重定向 URI 匹配。

2. `资源所有者身份验证及授予`。如果资源所有者授予访问请求，授权服务器通过发送以下 HTTP 响应来重定向用户代理：
   
    ``` shell
    HTTP/1.1 302 Found
    Location: http://client.abc.com/callback#access_token=2YotnFZFEjr1zCsicMWpAA&state=xyz&token_type=example&expires_in=3600
    ```

3. `用户代理重定向`。用户代理重定向 Location 里的 URI，按照重定向指令向 `Web 托管` (web-hosted) 的客户端资源发出请求。

    开发人员应注意，某些用户代理不支持在 `HTTP Location` 响应标头字段中包含片段。

    此类客户端将需要使用除 3xx 重定向响应之外的其他方法来重定向客户端 - 例如，返回包含`“继续”按钮`的 HTML 页面，其中包含链接到重定向 URI 的操作。

##### 分析

隐式授权的客户端是`公共的`，因此无需对客户端进行身份验证。Access Token 由于出现在重定向 URI 上，所以会向资源所有者和驻留在同一设备上的其他应用程序公开。

#### 资源所有者密码凭证授予

该授权需要`资源所有者`和`客户端`具有信任关系下进行，授权服务器在启用此授权类型时应特别小心，并且仅在其他流不可行时才允许它。

此授权类型适用于能够获取资源所有者凭据（用户名和密码，通常使用交互式表单）的客户端。

该流程如下：

![oauth_4.svg](oauth_4.svg)

##### 实现步骤

1. 通过密码直接向授权服务器请求 Access Token。

    ``` shell
    POST /token HTTP/1.1
    Host: server.abc.com
    Authorization: Basic czZCaGRSa3F0MzpnWDFmQmF0M2JW
    Content-Type: application/x-www-form-urlencoded
    
    grant_type=password&username=johndoe&password=A3ddj3w
    ```

    请求响应：

    ``` shell
    HTTP/1.1 200 OK
    Content-Type: application/json;charset=UTF-8
    Cache-Control: no-store
    Pragma: no-cache
    {
      "access_token":"2YotnFZFEjr1zCsicMWpAA",
      "token_type":"example",
      "expires_in":3600,
      "refresh_token":"tGzv3JOkF0XG5Qx2TlKWIA",
      "example_parameter":"example_value"
    }
    ```

#### 客户端凭证授权

这种授权方式必须对`受信的 (confidential) 客户端`使用。它无需用户代理介入，因此适合`服务器与服务器`之间的授权访问。

`受信任的客户端`通常被颁发（或建立）了一组用于向授权服务器进行身份验证的`客户端凭据`。

它的流程如下：

![oauth_3.svg](oauth_3.svg)

此类型用于**必须**在后台运行的`服务器到服务器` (server to server) 通信交互。

```shell
POST /token HTTP/1.1
Host: authorization-server.com
 
grant_type=client_credentials
&client_id=xxxxxxxxxx
&client_secret=xxxxxxxxxx
```

### 遵循 OAuth 2.0 授权的应用

在这些授权类型中`授权码授权`的方式最为常用，像腾讯开放平台的[授权登录](https://developers.weixin.qq.com/doc/oplatform/Website_App/WeChat_Login/Wechat_Login.html)、钉钉的[免登流程](https://developers.dingtalk.com/document/app/logon-free-process)、Github的[授权登录](https://docs.github.com/cn/developers/apps/authorizing-oauth-apps)等等。

参考资料：

\> [https://oauth.net/2/](https://oauth.net/2/)

\> [https://tools.ietf.org/html/rfc6749](https://tools.ietf.org/html/rfc6749)

\> [https://docs.github.com/en/developers/apps/scopes-for-oauth-apps](https://docs.github.com/en/developers/apps/scopes-for-oauth-apps)

\> [https://www.loginradius.com/blog/async/oauth2/](https://www.loginradius.com/blog/async/oauth2/)

\> [http://www.ruanyifeng.com/blog/2019/04/oauth-grant-types.html](http://www.ruanyifeng.com/blog/2019/04/oauth-grant-types.html)

\> [https://developers.weixin.qq.com/doc/oplatform/Website_App/WeChat_Login/Wechat_Login.html](https://developers.weixin.qq.com/doc/oplatform/Website_App/WeChat_Login/Wechat_Login.html)

\> [https://learn.microsoft.com/zh-cn/entra/identity-platform/v2-oauth2-implicit-grant-flow](https://learn.microsoft.com/zh-cn/entra/identity-platform/v2-oauth2-implicit-grant-flow)

\> [https://www.youtube.com/watch?v=CHzERullHe8](https://www.youtube.com/watch?v=CHzERullHe8)

\> [https://developer.okta.com/blog/2019/05/01/is-the-oauth-implicit-flow-dead](https://developer.okta.com/blog/2019/05/01/is-the-oauth-implicit-flow-dead)