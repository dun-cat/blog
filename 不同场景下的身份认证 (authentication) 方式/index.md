## 不同场景下的身份认证 (authentication) 方式 
### HTTP API 的鉴权

HTTP API 的鉴权，通常是`服务器`端生成一个唯一的`访问令牌` (token) 。

`客户端`通过`登录`或者`授权`（例：OAuth2）的方式获取到该 Token，之后每次接口访问都将 token 提供给服务器以验证 API 的访问权限。

#### 签名方式

Token 的`签名` (signature) 使用`对称加密`方式，通常使用 Hash 算法，保证加密的不可逆性质，可以选择 `HS256` 和 `HS384` 等等。

#### 放置位置

Token 通常放到 HTTP 的 Header 里，那应该采用 `Cookie` 还是 `Authorization` 字段呢？

###### 放置 Cookie 里

放置在 Cookie 有一个明显的好处是对于`浏览器端`脚本的开发者而言，可以偷懒，不用自己管理 Token。每次请求时，浏览器都会在 HTTP 的请求头上携带 Token 提供给服务器端。

但是在`移动应用`开发中，通常不会像浏览器一样自动管理 Token。所以如果 API 需要多平台共用。

同时，也正是因为浏览器会`自动`在 HTTP 请求头携带 Cookie 信息而造成了一些安全性的问题，例如：[CSRF 攻击](https://en.wikipedia.org/wiki/Cross-site_request_forgery)。此时需要额外在服务端设置合适的防御措施来避免此类问题。

虽然 Cookie 有一些安全隐患，但通过适当的设置可以减轻这些风险。例如：

* **HttpOnly**：防止 `JavaScript` 访问 Cookie，减轻 `XSS` 攻击风险。
* **Secure**：确保 Cookie 只通过 `HTTPS` 发送，防止传输中的窃听。
* **SameSite**：防止`跨站`请求伪造（CSRF）攻击。

另外，使用 Cookie 还有一个好处是允许在`多个子域下`共享 Cookie，因为这一点使得其成为[单点登录 (SSO)](https://en.wikipedia.org/wiki/Single_sign-on) 最常见的方式。

用户在使用同一个公司下的其它站点时，无需重新登录，对于使用者来说是无感知的。

###### 放置 Authorization 里

这也是比较常见的方式，因为是手动维护 Token，而且不会出现 CSRF 的问题。

可以设置 `Bearer Token` 格式：

``` makefile
Authorization: Bearer <token>
```

`Bearer Token` 是 OAuth2 [RFC 6750](https://datatracker.ietf.org/doc/html/rfc6750) 规范定义的，所以也是一种支持`授权方式`认证的格式。

除此之外，`Authorization` 支持 HTTP/1.0 [RFC 2617](https://datatracker.ietf.org/doc/html/rfc2617) 规范里定义的 `Basic` 和 `Digest` 两种认证格式。

#### 实现方案



### App 应用签名

