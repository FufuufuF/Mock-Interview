# 计算机网络 八股文复习大纲

> 目标岗位：全栈开发（偏前端 / AI Agent）
> 复习模式：**一轮对话讲一章**，讲完后可详细追问
> 题库来源：`questions/frontend/network.md`

---

## 复习节奏

- 每章安排一次会话，讲师主讲核心原理 + 追问方向
- 每章结束用 A/B/C/D 评分复盘（可写入 `records/`）
- 进度在下方表格中勾选 ✅

| 序号 | 章节 | 状态 |
|------|------|------|
| 01 | HTTP 协议演进 | ✅ 2026-08-13 |
| 02 | TCP / UDP | ✅ 2026-08-13 |
| 03 | DNS 与浏览器缓存 | ✅ 2026-08-13 |
| 04 | 跨域与同源策略 | ✅ 2026-08-13 |
| 05 | 身份认证 | ✅ 2026-08-13 |
| 06 | HTTPS 与 TLS | ⬜ |
| 07 | 实时通信：WebSocket / SSE / 长连接 | ⬜ |
| 08 | CDN 与网络性能优化 | ⬜ |
| 09 | 网络安全（XSS / CSRF / 纵深防御） | ⬜ |
| 10 | 综合大题冲刺 | ⬜ |

---

## 第一章：HTTP 协议演进

### 1.1 演进脉络
- HTTP/0.9 → 1.0 → 1.1 → 2.0 → 3.0 各版本核心变化
- 请求方法：GET / POST / PUT / DELETE / PATCH / HEAD / OPTIONS
- 状态码速查：2xx / 3xx（301、302、307、308、304）/ 4xx（401、403、404、429）/ 5xx（500、502、503、504）

### 1.2 HTTP/1.0 → HTTP/1.1
- 短连接 → Keep-Alive 持久连接
- Host 头、管线化（Pipelining）及其缺陷
- HTTP/1.1 队头阻塞（Head-of-Line Blocking）的产生原因

### 1.3 HTTP/2
- 二进制分帧层（Stream / Frame / Message）
- 多路复用：一个 TCP 连接并发多个请求
- 与 Keep-Alive 的本质区别
- 头部压缩 HPACK（静态表 + 动态表 + Huffman）
- Server Push 原理与实际应用
- 残留问题：TCP 层队头阻塞（丢包时所有流全部阻塞）

### 1.4 HTTP/3 与 QUIC
- 为什么抛弃 TCP 改用 UDP
- QUIC 解决的三大问题：队头阻塞、连接建立延迟、连接迁移
- 0-RTT 连接建立与安全风险
- 浏览器支持与落地场景

---

## 第二章：TCP / UDP

### 2.1 TCP 三次握手
- 完整流程（SYN → SYN+ACK → ACK）
- 为什么是三次而不是两次/四次（防止历史失效连接、双方序列号同步）
- 第三次握手丢失会发生什么
- SYN Flood 攻击原理与防御（SYN Cookie、半连接队列）

### 2.2 TCP 四次挥手
- 完整流程（FIN → ACK → FIN → ACK）
- 为什么断开要四次而建立只要三次（半关闭）
- 延迟确认：什么情况下四次可合并成三次
- TIME_WAIT 为什么是 2MSL（保证最后的 ACK 可达、防止旧连接数据串扰）
- 服务端大量 CLOSE_WAIT / 客户端大量 TIME_WAIT 的排查

### 2.3 可靠性与性能机制（了解级）
- 序列号与确认应答、超时重传、快速重传
- 滑动窗口、流量控制
- 拥塞控制：慢启动、拥塞避免、快恢复

### 2.4 TCP vs UDP 对比
- 可靠性 / 有序性 / 连接 / 头部开销 / 适用场景

---

## 第三章：DNS 与浏览器缓存

### 3.1 DNS 解析完整链路
- 浏览器缓存 → 系统 hosts → 本地 DNS 服务器 → 根服务器 → 顶级域服务器 → 权威服务器
- 递归查询 vs 迭代查询
- 各级 DNS 缓存与 TTL

### 3.2 DNS 的前端优化
- dns-prefetch / preconnect / preload / prefetch 的区别与用法
- 域名收敛：减少 DNS 查询、cookie 域隔离

### 3.3 浏览器缓存
- 强缓存：Cache-Control（max-age、no-cache、no-store、public/private、immutable）、Expires
- 协商缓存：ETag / If-None-Match、Last-Modified / If-Modified-Since
- 优先级与对比：强缓存 → 协商缓存 → 发请求
- 缓存位置：Memory Cache / Disk Cache / Service Worker / Push Cache
- 304 与 200（from cache）的区别

### 3.4 缓存策略设计
- HTML 不缓存 or no-cache，JS/CSS 带 hash 长缓存
- 发版更新如何及时生效

---

## 第四章：跨域与同源策略

### 4.1 同源策略
- 源 = 协议 + 域名 + 端口
- 限制了什么：DOM 访问、Cookie 读取、XHR/Fetch、iframe
- 不限制什么：img / script / link 标签、表单提交

### 4.2 CORS
- 简单请求 vs 预检请求（Preflight）
- 触发预检的条件：非简单方法、自定义头、非简单 Content-Type
- 相关响应头：Access-Control-Allow-Origin / Methods / Headers / Credentials / Max-Age
- 预检请求的性能开销与优化（合并请求、Max-Age 缓存）

### 4.3 JSONP
- 原理（script 标签 + 回调函数）
- 局限性：只支持 GET、安全性差

### 4.4 跨域方案全家桶
- CORS / JSONP / Nginx 反向代理 / devServer 代理 / postMessage / document.domain / window.name / WebSocket
- 各方案优缺点与选型

---

## 第五章：身份认证

### 5.1 Cookie
- 属性速查：Name/Value/Domain/Path/Expires/Max-Age/HttpOnly/Secure/SameSite/Partitioned
- SameSite：Strict / Lax / None 的区别
- Cookie 的跨域携带规则（第三方 Cookie）

### 5.2 Session 与 Token
- Session 流程：服务端存储 + sessionId 下发
- 分布式环境下 Session 共享：粘性会话 / Redis 集中存储 / JWT 无状态
- Token 认证流程
- 三者对比：存储位置、扩展性、安全性、CSRF 相关性

### 5.3 JWT
- 结构：Header.Payload.Signature（base64url）
- 签名与防篡改原理（HMAC / RSA）
- 优缺点：无状态、跨域友好 vs 无法主动失效、payload 可解
- 刷新机制：access token + refresh token

### 5.4 AI Agent 加分项：API 鉴权
- API Key 与 OAuth2.0 流程（授权码模式）
- 短期 token + 刷新令牌模式在 LLM API 中的应用

---

## 第六章：HTTPS 与 TLS

### 6.1 为什么需要 HTTPS
- HTTP 明文：窃听、篡改、伪装
- 对称加密 vs 非对称加密的取舍 → 混合加密

### 6.2 TLS 1.2 握手流程
- ClientHello / ServerHello → 证书 → 密钥交换 → 双方生成会话密钥 → Finished
- 数字证书与 CA：证书链验证、签名原理
- 中间人攻击（MITM）如何实现，证书如何防住

### 6.3 TLS 1.3 优化
- 握手从 2-RTT → 1-RTT，恢复 0-RTT
- 废弃 RSA 密钥交换（前向保密）、精简密码套件
- 0-RTT 的重放攻击风险

### 6.4 相关机制
- HSTS 与 SSL 剥离攻击
- 证书类型：DV / OV / EV
- 全链路 HTTPS：HTTP 请求在代理/网关处的解密

---

## 第七章：实时通信：WebSocket / SSE / 长连接

### 7.1 WebSocket
- 握手过程：HTTP Upgrade 请求 + 101 响应，Sec-WebSocket-Key/Accept
- 全双工、帧格式（Text / Binary / Ping / Pong / Close）
- 与 HTTP 的本质区别：连接建立后协议切换

### 7.2 长连接可靠性
- 心跳机制：Ping/Pong、业务心跳，为什么需要
- 断线重连：指数退避 + 抖动、重连上限
- 消息可靠性：ACK 确认、消息序号、去重、离线消息补偿

### 7.3 选型对比
- 轮询 / 长轮询 / SSE / WebSocket：延迟、开销、单向/双向
- SSE：text/event-stream 格式、自动重连、EventSource API、单向服务端推送

### 7.4 AI Agent 高频考点：SSE 流式输出
- LLM 打字机效果的实现：stream: true + 流式解析
- SSE 数据格式（data: / event: / id:）、UTF-8
- 中断/取消（AbortController）、断流重连、前后端配合的缓存策略
- 为什么 LLM 场景选 SSE 而不是 WebSocket（单向、HTTP 兼容、代理友好）

---

## 第八章：CDN 与网络性能优化

### 8.1 CDN 原理
- 核心组件：边缘节点、中心节点、回源、缓存
- DNS 调度（CNAME 指向 CDN 智能 DNS，返回最优节点 IP）
- DNS 调度 vs HTTP 302 调度对比
- 回源流程与回源优化：分级缓存、回源协议、Range 回源

### 8.2 静态资源缓存策略
- HTML 不缓存 / no-cache；带 hash 的 JS/CSS 长缓存（CDN 上）
- 资源更新时 URL 变化 → 命中新文件
- 动态内容不缓存或按参数缓存

### 8.3 全链路优化串联（面试综合）
- DNS 预解析 → preconnect → TLS 会话复用 → HTTP/2 多路复用 → CDN 就近 → 缓存命中
- 前端工程化配合：代码分割、按需加载、资源压缩

---

## 第九章：网络安全（XSS / CSRF / 纵深防御）

### 9.1 XSS
- 三种类型：存储型 / 反射型 / DOM 型（区别、触发链路）
- 防御：输出编码、输入校验、HttpOnly、CSP
- 框架层面的防御：React/Vue 默认转义，dangerouslySetInnerHTML / v-html 的风险
- 前端渲染（SPA）为何能缓解存储型/反射型，但 DOM 型依然存在

### 9.2 CSRF
- 原理：借用浏览器自动携带 Cookie 的特性
- 防御：SameSite Cookie、CSRF Token（生成/校验流程）、Referer/Origin 校验、自定义请求头
- CSRF 与 XSS 的关系：XSS 可绕过 CSRF 防御（读取 token / 发起请求）

### 9.3 纵深防御体系（困难题）
- CSP 配置：default-src / script-src / connect-src 等
- SRI：integrity 属性防 CDN 资源篡改
- 供应链安全：npm 包锁定、依赖审计、私有源
- 微前端安全隔离：沙箱、子应用数据隔离
- 安全响应头清单：X-Frame-Options、X-Content-Type-Options、Referrer-Policy

---

## 第十章：综合大题冲刺

### 10.1 从输入 URL 到页面展示（背诵链路）
- DNS 解析 → TCP 连接 → TLS 握手 → HTTP 请求/响应 → 浏览器解析渲染
- 网络层优化点：DNS 预解析、连接复用、HTTP/2、缓存、CDN
- HTTP/2 相比 1.1 在此链路的差异
- Preload Scanner 的角色

### 10.2 百万级在线长连接方案设计（系统设计视角）
- 架构：网关层（连接保持）、消息队列、推送服务
- 心跳与超时踢除、ACK 与重推、消息去重（幂等）
- 横向扩容、连接迁移（QUIC 特性）、移动端弱网优化
- 降级：WebSocket → 长轮询 → 轮询的切换策略

### 10.3 AI Agent 综合：LLM API 调用链路
- 超时分级与重试策略（指数退避 + 幂等键）
- 流式 vs 非流式、SSE 中断恢复
- 限流（429）与配额处理
- 网关层：鉴权、灰度、熔断、降级
- 流式响应的浏览器端处理：解析缓冲、渲染节流

---

## 每章复习流程（讲师执行）

1. 主讲核心原理，覆盖上表全部知识点
2. 结合 `questions/frontend/network.md` 的追问方向进行拷问
3. 学员复述关键机制（讲给讲师听）
4. 讲师按 A/B/C/D 评分并给出提升点
5. 可选：记录到 `records/`（markdown + json），供 `/review` 复盘
