## 基于 HTTP 的身份认证 (authentication) 技术 
### 基本认证

基本 (Basic) 认证方式在 [RFC 7617](https://datatracker.ietf.org/doc/html/rfc7617) 标准中被定义。

**原理**：客户端将`用户名`和`密码`以 `username:password` 格式拼接后 `Base64 编码`，通过 `Authorization` 请求头发送。

**示例**：

``` http
GET /api/data HTTP/1.1
Authorization: Basic dXNlcm5hbWU6cGFzc3dvcmQ=
```

**适用场景**：内部系统、低安全需求场景。

**总结**：感觉像是在裸奔。Base64 反编码直接看到账号密码。

### Session 认证

Session 认证是一种基于服务器端状态管理的身份验证技术，常用于`传统 Web 应用`（服务端渲染的网站）。

![sessionid.svg](sessionid.svg)

**特点**：服务器会`保存用户的登录状态`(这里指：Session ID)。那么用户退出登录时，服务器删除 Session ID 便即刻生效。客户端即便还存留 Session ID 也无法继续访问服务。

Session 认证中，服务器会把 Session ID 写入 HTTP 的响应头 `Set-Cookie` 字段里去：

``` http
Set-Cookie: connect.sid=s%3Aabc123; Path=/; HttpOnly; Secure; Expires=Wed, 21 Oct 2024 07:28:00 GMT
```

在 Java Web 应用中设置的默认名称 `JSESSIONID`。

与之相对应客户端(浏览器)技术是 Cookie，浏览器会自动管理 Cookie 数据的存储，并且每次发送请求的时，都会将服务器返回的 Cookie 数据**自动附加**到 HTTP 的请求头 `Cookie` 字段里去，用于服务器身份认证。

``` http
GET /profile HTTP/1.1
Cookie: connect.sid=s%3Aabc123
```

##### 会话有效期

服务器端如果不设置 `Expires` 或 `Max-Age`，那么浏览器在用户关闭当前网站标签页或退出浏览器，Session ID 会自动被浏览器立即删除，也就是所谓的`会话级`对话。 Cookie。

当然服务器端也可以控制失效期，以 Java Servlet 为例：

``` java
HttpSession session = request.getSession(); // 创建 Session
session.setMaxInactiveInterval(30 * 60);    // 服务端 Session 30 分钟无操作后过期
```

不过，Redis 数据库的键值对存储本身包含`过期时间`特性，因此可以为键 (Session ID) 设置生存时间（TTL），Redis 会自动维护删除过期数据，这样就不需要服务端应用单独设置过期时间。

**适用场景**：基于浏览器的服务器端渲染的 Web 应用。

**总结**：在前后端不分家的时代(也是很多后端还能跟得上前端开发)，Session 认证基本就是主流技术。随着移动互联网时代到来，渲染侧发生了历史性转折，前后端分离开发变为主流，该项技术也被其它更宽泛的适合跨用户端认证技术所取代。当然需要说明一点这并不说明这项技术是落后的，只是使用场景变少了。

### JWT 认证

JWT 认证是基于 Token 的认证，它是目前主流的 HTTP API 身法认证方式，即`服务器`端生成一个唯一的`访问令牌` (Access Token)。

`客户端`通过`账户密码直接登录`或者`授权`（例：OAuth2）的方式获取到该 Token，之后每次接口访问都将 Token 附加在 HTTP 请求中提供给服务器以验证 API 调用的合法性。

无论认证方案如何变化，所有的基于 Token 的 HTTP API 身份认证本质就是获取 Access Token 的过程。

JSON Web Token（JWT）是 [RFC 7519](https://datatracker.ietf.org/doc/html/rfc7519) 标准定义的认证方式。

![jwt.svg](jwt.svg)

#### JWT 格式

JWT 由三部分构成，以点（`.`）分隔：

``` text
Header.Payload.Signature
```

**头部** (Header)

- **作用**：描述`令牌类型`（如 JWT）及`签名算法`（如：HMAC SHA256 或 RSA）。
- **示例**：
  ```json
  {
    "alg": "HS256",
    "typ": "JWT"
  }
  ```
- **编码**：Base64Url 编码后形成第一部分。

**荷载** (Payload)

- **作用**：包含声明（Claims），即用户或实体`数据`。
- **声明类型**：
  - **注册声明（Registered Claims）**：预定义字段，如 `iss`（签发者）、`exp`（过期时间）、`sub`（主题）。
  - **公共声明（Public Claims）**：自定义但需避免冲突的字段。
  - **私有声明（Private Claims）**：双方协商的字段。
- **示例**：
  ```json
  {
    "sub": "1234567890",
    "name": "John Doe",
    "admin": true
  }
  ```
- **编码**：Base64Url 编码后形成第二部分。

**签名** (Signature)

- **作用**：验证消息完整性，防止篡改。
- **生成方式**：
  ``` text
  HMACSHA256(
    base64UrlEncode(header) + "." + base64UrlEncode(payload),
    secret
  )
  ```
- **依赖项**：Header 中指定的算法和密钥。

#### JWT 特点

* **包含数据**：JWT 令牌包含 payload，那么就可以把高频查询的用户基本信息（ID、用户名、角色等）放到里面，服务器可直接解析该信息用于大部分 API 请求。
* **无状态**：JWT 免去了向数据库查询用户的登录状态，而是由服务器直接通过 JWT 令牌结构里的`签名部分`验证 JWT 令牌的合法性。

JWT 令牌的合法性校验：对 header 和 payload 进行同样算法的签名计算，并将计算结果与 JWT 中的签名进行对比。如果两者一致，说明签名验证通过，JWT 在传输过程中没有被篡改。

**安全事项**：用于签名的密钥必须小心保管。由于算法都是公开的，一旦密钥泄漏。无需登录，只要 Header 和 Payload 就可伪造 JWT 进行服务访问。

#### 会话有效期

JWT 是无状态的，即所有验证信息都被包含在 JWT 令牌本身，也包括有效期信息，有效期存于 payload 的 `exp` 字段里。这就会导致服务器签发 JWT 令牌后无法被吊销，在有效期内重复登录生成的多个 JWT 令牌也都是合法的。

从另一个方面来说这也是其优点，多个合法 Token 无需额外数据库验证，使其适合分布式服务架构的应用。

![jwt1.svg](jwt1.svg)

#### Token 吊销机制

为了实现 Token 的主动吊销功能，需要引入 Redis 作为验证的存储机制。

在 JWT 认证中，通过 `jti`（JWT ID）标识令牌的唯一性，通过`令牌黑名单或白名单机制`实现`主动注销`功能。将 jti 存入 Redis 可以快速验证令牌是否被撤销，同时利用 Redis 的自动过期特性保持数据清洁。

##### 黑名单机制

<details>
<summary> 时序图 </summary>

![jti_blacklist.svg](jti_blacklist.svg)
</details>


签发 JWT 令牌时，在 payload 里加入 jti：

``` json
{
  "sub": "user123",
  "exp": 1735689600,  // 过期时间戳
  "jti": "abcd1234-5678-90ef-ghijklmnopqr"
}
```

代码示例（Node.js + Redis）:

``` ts
const redis = require("redis");
const client = redis.createClient();

// 用户注销时添加 jti 到黑名单
async function revokeToken(jti, expTimestamp) {
  const currentTime = Math.floor(Date.now() / 1000);
  const ttl = expTimestamp - currentTime;
  if (ttl > 0) {
    await client.set(`jti:${jti}`, "1", { EX: ttl, NX: true }); // NX: true 仅在 Key 不存在时设置
  }
}

// 校验 JWT 时检查黑名单
async function verifyToken(jti) {
  const exists = await client.exists(`jti:${jti}`);
  return exists === 0; // 返回 true 表示令牌有效
}
```

用户注销：从当前 JWT 中提取 jti 和 exp，计算`剩余有效时间`，存入 Redis。此时，JWT 令牌实际被标记为`失效状态`。

##### 白名单机制

在白名单机制中，系统**只允许明确列出的 JWT 访问资源**，与黑名单（记录失效令牌）相反，白名单**主动记录所有有效令牌**。白名单机制的核心流程如下：

<details>
<summary> 时序图 </summary>

![jti_whitelist.svg](jti_whitelist.svg)

</details>

1. **登录时存储令牌**  
   - 用户登录成功后，生成 JWT（含唯一 `jti` 和过期时间 `exp`）  
   - **将 `jti` 存入 Redis**，并设置自动过期时间（与 JWT 的 `exp` 一致）  
   - 示例 Redis 命令：  
     ```bash
     SET jti:123456 1 EX 3600  # 键名 jti:<jti>, 值随意, 过期时间与 JWT 一致
     ```
2. **每次请求校验白名单**  
   - 客户端携带 JWT 请求受保护资源  
   - 服务端解析 JWT 后：  
     1. 验证签名和有效期（基础校验）  
     2. **检查 Redis 中是否存在 `jti`**  
        - **存在** → 允许访问  
        - **不存在** → 拒绝访问（即使 JWT 未过期）  
3. **注销时删除令牌**  
   - 用户注销时，直接从 Redis 中删除对应的 `jti`  
   - 示例 Redis 命令：  
     ```bash
     DEL jti:123456
     ```

##### 白名单 vs 黑名单

| **场景**               | **白名单机制**                          | **黑名单机制**                          |
|-------------------------|---------------------------------------|----------------------------------------|
| **默认逻辑**            | 仅允许明确存在的令牌                   | 允许所有令牌，除非被明确禁止            |
| **存储开销**            | 高（存储所有有效令牌）                 | 低（仅存储已注销的令牌）                |
| **适用场景**            | 高安全场景（如金融系统）               | 常规场景（如社交平台）                  |
| **注销实现**            | 删除 Redis 中的 `jti`                 | 将 `jti` 加入 Redis 黑名单             |
| **令牌自动过期**        | 依赖 Redis 过期清理                    | 黑名单可设置较短过期时间（优化存储）    |

**适用场景**：前后端分离应用、微服务间认证。

**总结**：为了实现主动注销 Token 功能，需要引入 Redis。在黑名单和白名单方案中，**建议选择黑名单**。因为用户登出是个低频事件，可以减少性能开销。

#### 双令牌机制

实现`免登功能`，只要设置一个长期有效的 JWT 令牌即可。但这样会加剧 JWT 被意外泄漏的风险，采用`访问令牌`和`刷新令牌`双令牌机制可以有效避免安全风险。

* `访问令牌` (Access Token)：用于正常的 API 请求，分配较短失效期（例：5 分钟）。失效后向服务器提交 Refresh Token 重新申请一个。
* `刷新令牌` (Refresh Token)：用于重新生成访问令牌。其失效期较长（日、月、年）。失效后用户重新登录。

Refresh Token 由用户本地保管，只有在 Access Token 失效时再提供给服务器，因此降低了中间拦截可能性。而 Access Token 由于其有效期很短，即便被截获也很快失效。

![dual_tokens.svg](dual_tokens.svg)

**关键流程解析：**

1. **失效令牌检测**（步骤 3-5）：
   - 资源服务器收到请求后，**优先查询 Redis** 检查令牌状态（如黑名单、手动吊销）
   - Redis 返回失效状态后直接拒绝请求，**避免无效的数据库查询**
2. **令牌刷新与缓存更新**（步骤 7-9）：
   - 认证服务器生成新 access_token 后：
     - 将旧令牌加入 Redis 黑名单（防止被重复使用）
     - 缓存新令牌元数据（如用户ID、有效期）到 Redis，供后续快速验证
3. **新令牌验证优化**（步骤 10-12）：
   - 资源服务器再次请求时，通过 Redis 快速确认新令牌有效性
   - 避免重复解析 JWT 或查询数据库，**降低系统延迟**

**Redis 核心作用表：**

| 功能                | 数据结构       | 示例 Key-Value               | 过期策略             |
|---------------------|----------------|-----------------------------|----------------------|
| 令牌黑名单          | SET/HASH       | `blocked_tokens:<token>`    | 对齐原令牌过期时间    |
| 有效令牌缓存        | HASH           | `token_meta:<token>` + 用户信息 | 短于令牌实际有效期   |
| 刷新令牌绑定        | STRING         | `refresh_to_access:<refresh_token>` → access_token | 长期有效           |

如果需要补充**时序图交互细节**或**安全设计要点**（如加密存储、令牌绑定设备指纹等），可以进一步讨论。

#### 本地 Token 存储

Token 通常放到 HTTP 的请求头 (Header) 里，那应该采用 `Cookie` 还是 `Authorization` 呢？

##### Cookie

放置在 Cookie 有一个明显的好处是对于`浏览器端`脚本的开发者而言，可以偷懒，不用自己管理 Token。每次请求时，浏览器都会在 HTTP 的请求头上携带 Token 提供给服务器端。

但是在`移动应用`开发中，不会像浏览器一样自动管理 Token。所以如果 API 需要多平台共用，需要使用其它 Header 字段做认证。

同时，也正是因为浏览器会`自动`在 HTTP 请求头携带 Cookie 信息而造成了一些安全性的问题，例如：[CSRF 攻击](https://en.wikipedia.org/wiki/Cross-site_request_forgery)。此时需要额外在服务端设置合适的防御措施来避免此类问题。

虽然 Cookie 有一些安全隐患，但通过适当的设置可以减轻这些风险。例如：

* **HttpOnly**：防止 `JavaScript` 访问 Cookie，减轻 `XSS` 攻击风险。
* **Secure**：确保 Cookie 只通过 `HTTPS` 发送，防止传输中的窃听。
* **SameSite**：防止`跨站`请求伪造（CSRF）攻击。

###### 单点登录（子域名共享）

使用 Cookie 还有一个好处是允许在`多个子域下`共享 Cookie，因为这一点使得 Cookie 成为[单点登录 (SSO)](https://en.wikipedia.org/wiki/Single_sign-on) 最常见的技术选项。

如 `a.example.com` 和 `b.example.com`，可以将 Cookie 的 **Domain 属性**设置为`.example.com`，这样所有 example.com 下的子域名都可以访问该 Cookie。

##### Authorization

这也是比较常见的方式，因为是手动维护 Token，而且不会出现 CSRF 的问题。

可以设置 `Bearer Token` 格式：

``` makefile
Authorization: Bearer <token>
```

`Bearer Token` 是 OAuth2 [RFC 6750](https://datatracker.ietf.org/doc/html/rfc6750) 规范定义的，所以也是一种支持`授权方式`认证的格式。



