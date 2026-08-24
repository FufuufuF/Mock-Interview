# 身份认证深度学习（Cookie / Session / Token / JWT）

> 学习日期：2026-08-13
> 主题：Cookie 属性、Session 与 Token 模型、JWT 结构/签名/双令牌、API 鉴权（AI Agent 加分）
> 方式：慢节奏教学（一个知识点一轮对话，SameSite 与 JWT 加密专项深讲）
> 大纲：`.agents/.session/network-review-outline.md` 第五章

---

## 一、Cookie 属性速查

### 1.1 生命周期与本质
- Cookie = 服务端下发、浏览器按【域名】存储、后续请求自动携带的小数据块
- **下发位置：响应头 Set-Cookie**（不是响应体！）
- 存储：无 Expires/Max-Age = 会话期（关浏览器删，存内存）；有 = 持久（存磁盘）
- 携带：浏览器把符合条件的 Cookie 拼进【请求头】Cookie 字段
- 也可由 JS 设置（document.cookie），但 **HttpOnly 的 Cookie 只能由服务器设置**

### 1.2 重点属性
| 属性 | 作用 | 面试关联 |
|------|------|---------|
| HttpOnly | JS 读不到（document.cookie 拿不到），**不影响自动携带** | 防 XSS 窃取 Cookie（不防 XSS 本身） |
| Secure | 仅 HTTPS 连接发送 | 防中间人抓包截获 |
| SameSite | 限制跨站请求携带 | 防 CSRF 第一道防线 |
| Partitioned | 第三方 Cookie 分区（CHIPS） | 了解即可 |

### 1.3 SameSite 三值（详细场景）
| 值 | 跨站表单/fetch/iframe | 顶层导航 GET（点链接） | 场景 |
|----|:---:|:---:|------|
| Strict | ❌ 不带 | ❌ 不带 | 网银、后台（安全优先，体验差） |
| Lax（默认） | ❌ 不带 | ✅ 带 | 平衡：攻击者请求被拦，正常跳转保留登录态 |
| None（必须配 Secure） | ✅ 都带 | ✅ 带 | 第三方 Cookie：广告跨站追踪、跨域 SSO |

- **Lax 绕过**：放行顶层导航 GET → 若目标站有 GET 改数据的接口，`<a href>` 诱导点击即可触发 → **写操作必须用 POST/PUT**
- **为什么广告/SSO 需要 None**：第三方上下文（news.com 页面里的 ad.com iframe 访问 ad.com 的 Cookie）属于跨站，Lax 不带
- 同主域 SSO（passport.company.com + Domain=.company.com）是 same-site，不需要 None
- Chrome 正在淘汰第三方 Cookie（隐私）；SSO 向 OAuth2/OIDC 迁移

### 1.4 三个「跨」的判定维度（易混）
- 跨**源**（CORS 管）：协议 + 域名 + 端口
- 跨**站**（SameSite 管）：协议 + 可注册域名（不管端口/子域）
- Cookie 携带（浏览器管）：域名（**端口不参与匹配**，localhost 各端口共享）

---

## 二、Session（服务端有状态）

### 2.1 流程
```
① 登录 → 服务器验证 → 创建 session 存服务器（内存/Redis）
② Set-Cookie: sessionId=abc123; HttpOnly; SameSite=Lax 下发
③ 后续请求自动带 Cookie → 服务器查 session → 放行
④ 登出 = 删 session → 立即失效
```

### 2.2 优缺点
- ✅ 可主动失效（踢人/封号/改密码/登出）
- ✅ 敏感数据在服务端，客户端看不到
- ❌ 有状态 → 分布式必须共享 session

### 2.3 分布式 Session 共享三方案
1. 粘性会话（Sticky Session）：固定打到同一台 → 机器挂了 session 全丢
2. **Redis 集中存储（主流）**：服务器无状态化，随便扩容；Redis 需高可用
3. 无状态化（JWT）：不存了，验签即可

---

## 三、Token（服务端无状态）

### 3.1 流程
```
① 登录 → 签发 token（用户信息 + 过期时间 + 签名）返回
② 前端存（localStorage / 内存 / Cookie）
③ 请求带 Authorization: Bearer <token>
④ 服务端验签（不查存储）→ 放行
```

### 3.2 Session vs Token 对比
| 维度 | Session | Token（JWT） |
|------|---------|-------------|
| 状态 | 有状态（必须存） | 无状态（验签即可） |
| 主动失效 | ✅ | ❌ 有效期内无法吊销 |
| 分布式 | 需 Redis 共享 | 天然友好 |
| 携带 | Cookie 自动带 | Authorization 头手动加 |
| CSRF 风险 | 高（自动携带） | 低（表单/图片带不了头） |
| 跨域 | 麻烦 | 方便 |

---

## 四、JWT

### 4.1 三段结构
```
eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiIxMjMifQ.x7YqQ...
└─ Header（算法）─┘└─ Payload（信息）─┘└─ Signature（签名）─┘
```
- Header/Payload 是 **base64url 编码，不是加密**——任何人可解码看到内容
- 防篡改靠签名：Signature = HMAC(header.payload, secret)，重算对不上即拒绝
- **签名 vs 加密**：加密保密封（可逆）；签名防篡改/认证来源（不可逆）。JWT 贴防伪标签，不锁保险柜

### 4.2 信息泄漏防护四层
1. payload 最小化：只放 userId/role/exp，密码/手机号/余额绝不能放（需要就查库）
2. 传输靠 HTTPS（TLS 加密）
3. 存储靠 HttpOnly + Secure
4. 短期有效期缩小损失窗口

### 4.3 签名算法
| | HS256（HMAC） | RS256（RSA） |
|--|--------------|--------------|
| 密钥 | 对称（签发=校验同一密钥） | 非对称（私钥签、公钥验） |
| 适用 | 单体 | 认证服务独立 + 多业务服务 |
| 风险 | 密钥泄露 = 可伪造 | 业务服务只有公钥，被攻破也伪造不了 |

### 4.4 缺点（必背）
1. 无法主动失效
2. payload 明文
3. 体积大（比 sessionId 大一个数量级）
4. 密钥泄漏全盘皆输

### 4.5 双令牌机制（Access + Refresh）
```
登录 → access token（15min~1h，调 API）+ refresh token（7~30 天，只换新）
access 过期 → 401 → refresh 调 /refresh → 新 access（+ 轮换旧 refresh 作废）
refresh 过期 → 重新登录
```
- 解决：access 泄漏损失窗口小；refresh 存服务端**可主动吊销**；轮换防重放
- 存储：access 存内存（防 XSS 偷）；refresh 存 HttpOnly Cookie
- 本质：**用少量有状态（refresh 可吊销）换安全与体验的平衡**

---

## 五、API 鉴权（AI Agent 加分）

### 5.1 API Key 模式（LLM API 主流）
- 长期静态字符串，`Authorization: Bearer sk-xxx`，服务端校验 + 计费 + 限流
- 场景：服务端到服务端（Agent 后端 → LLM 平台）
- 风险：泄漏被盗刷 → 吊销/轮换/额度上限
- **key 绝不能放前端**（浏览器里任何字符串都能被爬走）→ Agent 应用：前端 → 自己的后端（带 key）→ LLM

### 5.2 OAuth2.0 授权码模式
- 四角色：用户（资源所有者）/ 客户端应用 / 授权服务器 / 资源服务器
- 五步流程：① 点第三方登录 → ② 重定向授权页 → ③ 用户登录授权 → ④ 重定向回带一次性 code → ⑤ 服务端拿 code + client_secret 换 access token → ⑥ 调 API
- **为什么 code 中转**：重定向经过浏览器，直接给 token 会暴露；code 一次性短期；换 token 需 client_secret（只有服务端有）
- **state 参数防 CSRF**：回调校验是否自己发起的授权

### 5.3 LLM 场景鉴权层次
| 层 | 凭证 |
|----|------|
| 用户 → 你的应用 | 双 token 会话 / OAuth2 登录 |
| 你的后端 → LLM | API Key（只放服务端） |
| 第三方 → 你的 API | API Key 或 OAuth2 client 模式 |

---

## 六、本章自查清单

1. Cookie 在哪下发、存哪、怎么携带？
2. HttpOnly 防什么不防什么？Secure 呢？
3. SameSite 三值对比？Lax 的绕过场景？
4. 跨源 / 跨站 / Cookie 携带三个判定维度？
5. Session 为什么有状态？分布式三方案？
6. Token 为什么无状态？最大缺点？
7. JWT 三段结构？签名 vs 加密？
8. HS256 vs RS256？
9. 双令牌机制解决什么？refresh 为什么能吊销？
10. API Key 为什么不能放前端？OAuth2 为什么需要 code 中转？

## 七、误区纠正记录

| # | 错误理解 | 正确模型 |
|---|---------|---------|
| 1 | SameSite 是「允许跨站请求」 | 是「限制跨站携带」：Strict 不带 / Lax 仅顶层导航 GET 带 / None 全带 |
| 2 | Cookie 在响应体中下发 | 在【响应头】Set-Cookie 中下发，请求时经【请求头】Cookie 携带 |
| 3 | HttpOnly = 只有 HTTP 请求能读 | HttpOnly = JS 读不到但照样自动携带；只能由服务器设置 |
| 4 | JWT 有加密所以信息安全 | base64url 是编码不是加密，payload 明文；防篡改靠签名 |
| 5 | 加密 = 签名 | 加密保密（可逆），签名防篡改（不可逆） |
| 6 | Token 完全无状态是纯优点 | 无法主动失效是硬伤，业界都用双令牌混合方案 |
| 7 | 第三方授权直接给 token | 授权码模式用一次性 code 中转，token 换发在服务端 |
