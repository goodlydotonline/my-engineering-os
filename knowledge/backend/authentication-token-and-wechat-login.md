# JWT、OAuth 2.0、微信登录与应用自有 Token

> 这篇笔记整理几个容易混在一起的问题：JWT 是什么，OAuth 2.0 是什么，微信登录为什么通常属于 OAuth 2.0 场景，应用自己为什么还要签发 Access Token / Refresh Token，以及如果 Token 被盗应该如何防护。

---

## 1. 先给结论

JWT 和 OAuth 2.0 不是同一层面的东西。

- **OAuth 2.0** 是一个授权框架，重点解决“一个应用如何安全地获得访问某些资源的授权”。
- **JWT** 是一种 Token 格式，重点解决“如何把一段可信的身份/权限信息放进一个可验证的字符串里”。

更短地说：

```text
OAuth 2.0 规定流程。
JWT 规定 Token 长什么样。
```

所以它们不是二选一关系，而是经常一起使用：

```text
OAuth 2.0 流程签发 access_token
access_token 的格式可以是 JWT
```

也可以反过来说：

```text
一个系统可以不用 OAuth 2.0，但仍然使用 JWT 做登录态。
一个 OAuth 2.0 系统也可以不用 JWT，而用不透明随机字符串作为 access_token。
```

---

## 2. 从第一性原理理解“登录态”

Web / App 后端最基本的问题是：

```text
用户第一次输入账号密码后，后续每次请求，服务端怎么知道“这还是同一个用户”？
```

HTTP 请求本身是无状态的。用户今天请求一次接口，下一秒再请求一次接口，服务端天然不知道这两个请求是否来自同一个人。

所以系统需要一个“凭证”。用户登录成功后，服务端发给客户端一个凭证。客户端后续每次请求都带上这个凭证，服务端验证凭证后，就知道是谁在请求。

这个凭证可以有很多形式：

```text
Session ID
Cookie
Access Token
JWT
Opaque Token
```

本质都是：

```text
客户端保存一段凭证
请求时带给服务端
服务端验证凭证
验证通过后识别用户身份和权限
```

---

## 3. JWT 是什么

JWT 全称是 JSON Web Token。它是一种自包含 Token 格式，结构通常是：

```text
header.payload.signature
```

例如一个 JWT 看起来像这样：

```text
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiJ1c2VyXzEyMyIsInJvbGUiOiJpbnNwZWN0b3IiLCJleHAiOjE3ODMxNTIwMDB9.signature
```

它分三段：

```text
Header:    使用什么签名算法
Payload:   携带哪些身份和权限信息
Signature: 服务端签名，防止内容被篡改
```

Payload 里可能有：

```json
{
  "sub": "user_123",
  "tenant_id": "tenant_001",
  "role": "inspector",
  "scope": ["feedback:upload"],
  "exp": 1783152000
}
```

这些字段通常叫 claims。

常见 claims：

| 字段 | 含义 |
|---|---|
| `sub` | subject，用户 ID |
| `exp` | 过期时间 |
| `iat` | 签发时间 |
| `iss` | 签发者 |
| `aud` | 接收方/使用方 |
| `role` | 用户角色 |
| `scope` | 权限范围 |
| `tenant_id` | 所属机构/租户 |

JWT 的核心价值是：

```text
服务端可以验证签名，确认 Token 没被篡改。
```

如果有人把 payload 里的 `role` 从 `user` 改成 `admin`，签名就对不上，服务端会拒绝。

但注意一个非常关键的点：

```text
JWT 防篡改，不防泄露。
```

如果别人完整拿到了你的 JWT，他不需要篡改它，也可以拿它请求接口。这就是后面要讲的 Token 被盗问题。

---

## 4. OAuth 2.0 是什么

OAuth 2.0 是一个授权框架。它最经典的问题是：

```text
一个第三方应用，如何在不知道用户密码的情况下，获得访问用户某些资源的权限？
```

比如你下载了一个第三方 App，它支持“微信登录”。这个 App 不应该知道你的微信密码。正确方式是：它把你跳转到微信，由微信确认你的身份，然后微信把一个授权结果交给这个 App。

OAuth 2.0 里有几个核心角色。

| 角色 | 在微信登录里的例子 |
|---|---|
| Resource Owner | 用户本人 |
| Client | 第三方 App，也就是你下载的应用 |
| Authorization Server | 微信授权服务器 |
| Resource Server | 微信用户信息接口，比如获取 OpenID、头像、昵称的接口 |
| Access Token | 微信发给第三方 App 的访问令牌 |
| Authorization Code | 授权码，App 后端用它向微信换 Token |

OAuth 2.0 的重点不是“Token 长什么样”，而是“如何安全地拿到 Token”。

---

## 5. JWT vs OAuth 2.0

这两者经常被混淆，因为它们都出现在登录和认证系统里。但它们关注的问题不同。

| 对比项 | JWT | OAuth 2.0 |
|---|---|---|
| 本质 | Token 格式 | 授权框架 |
| 解决什么问题 | 如何表达和验证身份/权限信息 | 第三方如何安全获得授权 |
| 是否定义登录流程 | 不定义 | 定义授权流程 |
| 是否一定用于第三方登录 | 不一定 | 经常用于第三方登录 |
| 是否一定包含 access_token | JWT 本身不是流程概念 | OAuth 2.0 有 access_token |
| 是否可以一起用 | 可以 | 可以把 access_token 做成 JWT |

一个简单判断：

```text
你在讨论“Token 字符串里面有什么、怎么验签” -> JWT
你在讨论“跳转授权、授权码、第三方登录、微信登录” -> OAuth 2.0
```

---

## 6. 工业界什么时候用 JWT

JWT 常用于应用自己的登录态。

典型场景：

```text
用户登录你的 App
你的后端验证账号密码/短信/微信身份
你的后端签发自己的 access_token
这个 access_token 可以是 JWT
App 后续访问你的业务接口时带这个 JWT
```

例如你的业务接口：

```http
POST /v1/image-collections
Authorization: Bearer <your_app_access_token>
```

服务端验证这个 JWT：

```text
签名是否有效
是否过期
用户是否存在
用户是否有 feedback:upload 权限
用户是否属于当前租户
```

JWT 的优点：

- 服务端可以快速验证，不一定每次查数据库；
- 可以携带用户 ID、角色、租户、权限范围；
- 适合微服务之间传递身份；
- 对移动端、Web、服务端都通用。

JWT 的缺点：

- 一旦签发，在过期前通常难以立即吊销，除非引入黑名单或版本号；
- Payload 默认只是 Base64URL 编码，不是加密，不能放敏感信息；
- Token 被盗后，攻击者可以在有效期内冒用；
- 如果过期时间设置太长，风险会很大。

工业界常见实践：

```text
Access Token 用短期 JWT
Refresh Token 用长期随机字符串，服务端存储并可吊销
```

---

## 7. 工业界什么时候用 OAuth 2.0

OAuth 2.0 常用于“授权给第三方”或“统一身份登录”。

典型场景：

1. 第三方登录

```text
微信登录
GitHub 登录
Google 登录
企业微信登录
飞书登录
```

2. 第三方应用访问用户资源

```text
一个日历 App 请求访问你的 Google Calendar
一个自动化工具请求访问你的 GitHub 仓库
一个 BI 工具请求访问企业数据接口
```

3. 企业统一登录 SSO

```text
公司内部系统接入统一身份平台
用户在身份平台登录一次
多个内部系统共享登录态
```

OAuth 2.0 的优点：

- 第三方应用不需要知道用户密码；
- 用户可以授权一部分权限，而不是交出全部账号；
- 授权可以撤销；
- 适合跨应用、跨组织、第三方生态。

OAuth 2.0 的复杂点：

- 角色多，流程多；
- 要处理回调、授权码、token 交换；
- 移动端、Web 端、安全要求不同；
- 需要防 CSRF、重放攻击、授权码劫持等问题。

---

## 8. 微信登录是不是 OAuth 2.0 场景

是的，第三方 App 支持微信登录，通常就是 OAuth 2.0 / OpenID Connect 类似的应用场景。微信的具体协议实现有自己的细节，但理解模型可以按 OAuth 2.0 来看。

用 OAuth 2.0 角色解释微信登录：

| OAuth 2.0 角色 | 微信登录里的对象 |
|---|---|
| Resource Owner | 用户本人 |
| Client | 你的第三方 App |
| Authorization Server | 微信授权服务器 |
| Resource Server | 微信用户信息接口 |
| Authorization Code | 微信返回给你的临时授权码 |
| Access Token | 你的后端用授权码向微信换到的微信 access token |
| OpenID / UnionID | 微信侧标识用户身份的 ID |

完整流程大概是：

```text
用户点击“微信登录”
        │
        ▼
App 唤起微信授权页面
        │
        ▼
用户在微信侧确认授权
        │
        ▼
微信返回 authorization code 给 App
        │
        ▼
App 把 code 发给你的后端
        │
        ▼
你的后端拿 code + app_secret 向微信换微信 access_token
        │
        ▼
你的后端用微信 access_token 获取 OpenID / UnionID / 用户资料
        │
        ▼
你的后端查找或创建应用自己的用户
        │
        ▼
你的后端签发应用自己的 Access Token + Refresh Token
        │
        ▼
App 后续访问你的业务接口，只用你应用自己的 Token
```

关键点：

```text
微信 Token 是用来和微信交互的。
你应用自己的 Token 是用来访问你自己的后端的。
```

这两个 Token 不是一回事。

---

## 9. 为什么微信登录后应用还要创建自己的 Token

因为微信只能证明：

```text
这个用户在微信体系里是谁。
```

但你的应用还需要知道：

```text
这个用户在你的系统里是谁
属于哪个租户
是什么角色
有没有上传反馈数据的权限
有没有查看后台的权限
是否同意模型优化数据回流
```

这些是你应用自己的业务问题，微信不会替你管理。

所以微信登录成功后，你的后端通常会做一层“账号绑定/账号创建”：

```text
微信 OpenID / UnionID -> 你的 user_id
```

然后签发你应用自己的 Token。

这就是为什么一个 App 用微信登录，但日常访问业务接口时，不是一直拿微信 Token 调你的接口，而是拿你应用自己的 Token。

---

## 10. FireCopilot 当前功能应该怎么理解

你当前的 FireCopilot 图片反馈功能，现在 MVP 里用了一个：

```text
FEEDBACK_APP_API_TOKEN=dev-feedback-token
```

这是临时共享密钥，不是正式用户登录体系。

MVP 请求大概是：

```http
POST /v1/image-collections
Authorization: Bearer dev-feedback-token
```

这只是为了避免上传接口完全裸奔。

正式产品里，FireCopilot 应该改成：

```text
用户登录 FireCopilot
        │
        ▼
FireCopilot 后端签发自己的 Access Token + Refresh Token
        │
        ▼
用户开启“参与模型优化数据回流”
        │
        ▼
App 上传图片反馈时带 FireCopilot 自己的 access_token
        │
        ▼
feedback-service 校验用户、设备、租户、权限、授权同意、hash、幂等
```

也就是说，正式产品中你作为 FireCopilot 开发者，需要维护自己的：

```text
Access Token
Refresh Token
用户表
设备表
同意记录
权限/角色
Token 续签和吊销逻辑
```

如果 FireCopilot 也支持微信登录，那么微信只负责“帮你确认用户是谁”。FireCopilot 仍然要签发自己的 Token 来访问自己的业务接口。

---

## 11. 第一次微信登录后，后面会发生什么

你给出的流程是正确的，可以整理成下面这样：

```text
第一次安装 App
        │
        ▼
微信 OAuth 登录
        │
        ▼
应用创建用户
        │
        ▼
签发 Access Token + Refresh Token
        │
        ▼
日常使用 App（只使用应用自己的 Token）
        │
        ├── Access Token 过期 → 用 Refresh Token 自动续签
        │
        └── Refresh Token 也过期（或用户主动退出登录）
                │
                ▼
再次发起微信 OAuth
                │
                ▼
微信确认你的身份（如果微信本身仍保持登录，通常无需再次输入密码）
                │
                ▼
应用找到原来的用户（根据 OpenID/UnionID）
                │
                ▼
重新签发一套新的 Access Token 和 Refresh Token
```

再拆细一点。

### 11.1 短时间内再次打开 App

如果用户刚登录过，App 本地还保存着有效的 access token，那么它可以直接用自己的 access token 请求业务接口。

```text
App 打开
    │
    ├── access_token 仍有效
    │       └── 直接进入应用
    │
    └── access_token 已过期，但 refresh_token 仍有效
            └── 用 refresh_token 自动换新的 access_token
```

这时通常不需要重新走微信登录。

原因是：

```text
微信已经完成了“初次身份确认”。
日常业务访问由你的应用自己的 Token 负责。
```

### 11.2 应用自己的 Token 和微信 Token 有什么区别

| 对比项 | 微信 Token | 应用自己的 Token |
|---|---|---|
| 谁签发 | 微信 | 你的应用后端 |
| 给谁用 | 你的后端调用微信接口时用 | App 调你的业务接口时用 |
| 表示什么 | 用户在微信体系里的授权 | 用户在你应用里的登录态和权限 |
| 包含你的业务角色吗 | 不包含 | 可以包含 |
| 过期策略谁控制 | 微信 | 你的应用 |
| 能不能访问 FireCopilot 接口 | 不应该直接用 | 应该用它 |

微信 Token 和应用 Token 的联系是：

```text
应用后端用微信 Token / OpenID / UnionID 确认用户身份
然后映射到自己的 user_id
再签发自己的 Token
```

它们不是同一个 Token，也不应该混用。

### 11.3 应用自己的 Token 会过期吗

会，而且必须过期。

常见策略：

```text
Access Token: 15 分钟到 2 小时
Refresh Token: 7 天到 90 天，视安全级别而定
```

Access Token 短，是为了降低泄露后的风险。

Refresh Token 长，是为了用户不用频繁登录。

但 Refresh Token 需要服务端存储、可吊销、可轮换。

### 11.4 应用侧怎么管理这些 Token

服务端通常维护一张 refresh token 表或 session 表：

```text
id
user_id
device_id
refresh_token_hash
issued_at
expires_at
revoked_at
last_used_at
ip
user_agent
```

注意一般不直接明文存 refresh token，而是存 hash。客户端拿 refresh token 来换新 access token 时，服务端 hash 后比对。

Access Token 如果是 JWT，服务端可以不存每一个 access token，但仍然要有吊销机制，比如：

```text
短过期时间
用户 token_version
黑名单 jti
设备 session 状态
```

---

## 12. 隔很久后再次打开 App，会不会再次触发微信登录

分情况。

### 情况 A：应用 refresh token 还有效

用户打开 App，access token 过期了，但 refresh token 还有效。

```text
App 用 refresh_token 换新的 access_token
用户无需感知
不触发微信登录
```

### 情况 B：应用 refresh token 过期或被吊销

这时 App 无法自己续签。

```text
App 需要重新登录
用户点击微信登录
再次发起微信 OAuth
```

但这不一定意味着用户要重新输入微信账号密码。

如果用户手机上微信仍然登录，通常是：

```text
唤起微信
用户确认授权或快速确认
返回 App
```

如果微信侧也失效，才可能要求用户重新登录微信。

### 情况 C：用户主动退出登录

用户主动退出时，应用应该：

```text
删除本地 access token
删除本地 refresh token
通知服务端吊销当前 refresh token/session
```

下次再进应用，就需要重新登录，包括重新走微信 OAuth。

---

## 13. Token 被盗了怎么办

你的问题非常关键：

```text
如果别人完整拿到了我的 JWT，是不是可以以我的身份访问应用？
```

答案是：

```text
在 JWT 有效期内，通常可以。
```

因为 JWT 防的是“篡改”，不是“复制”。

### 13.1 JWT 防篡改是什么意思

攻击者不能把：

```json
{"role":"user"}
```

改成：

```json
{"role":"admin"}
```

因为签名会失效。

但如果攻击者拿到的是完整原始 JWT：

```text
header.payload.signature
```

他不需要改，直接拿去请求：

```http
Authorization: Bearer <stolen_jwt>
```

服务端只看签名和过期时间，可能会认为这是合法请求。

所以正式系统必须假设：

```text
Token 有可能泄露。
```

然后通过工程手段降低泄露后的影响。

---

## 14. 工业界如何防 Token 被盗

### 14.1 使用 HTTPS

所有请求必须走 HTTPS，避免 Token 在网络中明文泄露。

```text
HTTP:  Token 可能被中间人看到
HTTPS: 传输层加密
```

这是底线。

### 14.2 Access Token 短有效期

Access Token 有效期不能太长。

例如：

```text
15 分钟
30 分钟
2 小时
```

被盗后，即使攻击者能用，也只能用一小段时间。

### 14.3 Refresh Token 服务端可吊销

Refresh Token 通常不做成完全无状态 JWT，而是做成随机字符串，服务端存 hash。

这样可以：

```text
用户退出登录时吊销
发现异常时吊销
设备丢失时吊销
密码变更时全部吊销
管理员封禁用户时吊销
```

### 14.4 Refresh Token 轮换

每次用 refresh token 换新 access token 时，同时签发新的 refresh token，并让旧 refresh token 失效。

```text
旧 refresh token -> 换新 access token + 新 refresh token
旧 refresh token 立即失效
```

如果旧 refresh token 再次出现，说明可能被盗用，服务端可以吊销整个 session。

### 14.5 安全存储 Token

移动端不要随便把 Token 放在普通明文存储里。

常见做法：

| 平台 | 建议存储 |
|---|---|
| iOS | Keychain |
| Android | Keystore / EncryptedSharedPreferences |
| Web | HttpOnly Secure Cookie 更常见 |

Flutter 里常见是用 `flutter_secure_storage` 一类的安全存储，而不是普通 SharedPreferences 保存敏感 Token。

### 14.6 Token 绑定设备或 Session

服务端维护 session：

```text
user_id
device_id
refresh_token_hash
session_id
status
last_used_at
```

Access Token 里带：

```json
{
  "sub": "user_123",
  "sid": "session_456",
  "device_id": "device_789",
  "exp": 1783152000
}
```

服务端可以检查 session 是否仍有效。

如果设备被禁用，即使 JWT 没过期，也可以拒绝高风险接口。

### 14.7 高风险操作二次验证

比如：

```text
导出数据
删除数据
修改密码
绑定新设备
查看敏感图片
```

这些操作可以要求二次验证：

```text
重新输入密码
短信验证码
企业 SSO 确认
管理员审批
```

### 14.8 异常检测

服务端可以记录：

```text
IP
设备 ID
User-Agent
地理位置
请求频率
失败次数
```

发现异常时：

```text
要求重新登录
吊销 refresh token
冻结账号
提醒用户
```

例如同一个 Token 几分钟内从两个国家访问，就应该触发风控。

### 14.9 权限最小化

Access Token 里不要给过大的权限。

例如 FireCopilot 上传反馈只需要：

```text
feedback:upload
```

不应该顺便带：

```text
admin:export
admin:delete
```

权限越小，被盗后的损失越小。

---

## 15. 如果我是应用开发者，需要做哪些保障

作为应用开发者，你至少要设计以下内容。

### 15.1 登录与 Token 签发

```text
用户登录成功
生成 access token
生成 refresh token
保存 refresh token hash
返回给客户端
```

### 15.2 Token 验证中间件

每个需要登录的接口都走统一认证中间件：

```text
读取 Authorization header
验证 access token 签名
检查 exp 是否过期
解析 user_id / tenant_id / role / scope
加载必要的用户状态
写入 request context
```

### 15.3 Refresh Token 续签接口

```http
POST /auth/refresh
```

逻辑：

```text
校验 refresh token
检查是否过期/吊销
签发新 access token
必要时轮换 refresh token
```

### 15.4 退出登录

```http
POST /auth/logout
```

逻辑：

```text
吊销当前 refresh token/session
客户端删除本地 token
```

### 15.5 权限控制

不是登录了就能做所有事。

接口应该检查 scope 或 role：

```text
feedback:upload
feedback:view
feedback:review
dataset:export
admin:user-manage
```

### 15.6 安全存储

客户端要安全保存 refresh token。

移动端：

```text
Keychain / Keystore / flutter_secure_storage
```

Web：

```text
HttpOnly + Secure + SameSite Cookie
```

### 15.7 日志与审计

重要操作要记录：

```text
谁
什么时候
在哪个设备
做了什么
操作对象是什么
结果成功还是失败
```

对于 FireCopilot，至少包括：

```text
上传反馈图片
查看反馈详情
审核更正
导出数据
删除数据
```

---

## 16. FireCopilot 图片反馈功能的正式认证建议

MVP 现在是：

```text
FEEDBACK_APP_API_TOKEN=dev-feedback-token
```

正式产品建议改为：

```text
用户登录 token + 设备身份 + 用户同意记录 + 租户权限 + 后台角色权限
```

上传图片反馈时：

```http
POST /v1/image-collections
Authorization: Bearer <firecopilot_access_token>
X-Device-Id: <device_id>
Idempotency-Key: <client_record_id>
```

服务端校验：

```text
access_token 是否有效
用户是否存在
用户是否属于当前机构
用户是否有 feedback:upload 权限
设备是否可信
用户是否开启模型优化数据回流
图片 hash 是否匹配
client_record_id 是否重复
请求大小是否超限
```

后台查看时：

```text
管理员登录
校验 role / permission
只允许查看本机构数据
导出需要更高权限
所有导出写审计日志
```

---

## 17. 最后再压缩成一句话

JWT 和 OAuth 2.0 都会出现在登录认证系统里，但它们不是同一个东西。

```text
OAuth 2.0 是授权流程。
JWT 是 Token 格式。
微信登录通常是 OAuth 2.0 场景。
微信确认用户是谁后，你的应用仍然要创建自己的用户和自己的 Token。
日常访问你应用的接口，用的是你应用自己的 Access Token。
Access Token 可以是 JWT，但必须短期有效。
Refresh Token 用来续签，应该服务端可吊销、可轮换。
如果 JWT 被完整盗走，攻击者可以在有效期内冒用，所以工程上要靠 HTTPS、短过期、Refresh Token 轮换、安全存储、设备绑定、权限最小化、异常检测和审计来降低风险。
```
