# 跨域与同源策略深度学习

> 学习日期：2026-08-13
> 主题：同源策略、CORS（简单/预检请求）、JSONP、跨域方案选型
> 方式：慢节奏教学（一个知识点一轮对话，含多次追问与纠正）
> 大纲：`.agents/.session/network-review-outline.md` 第四章

---

## 一、同源策略

### 1.1 同源定义
**源 = 协议 + 域名 + 端口**，三者完全一致才同源。
- 协议不同 / 域名不同 / 端口不同 → 均不同源（localhost:3000 vs localhost:8080 已跨域）

### 1.2 限制什么
- 跨源读取 DOM（iframe）
- 跨源读取 Cookie / localStorage
- 跨源 XHR / Fetch：**请求发出去了、服务器执行了、响应回来了，是浏览器拿到响应后拦截，不交给 JS**

### 1.3 不限制什么（核心边界）
> **「加载 / 引用 / 导航」不受限，「读取」才受限。**

| 不受限行为 | 用途 |
|-----------|------|
| `<img src>` | 跨源显示图片 |
| `<script src>` | 跨源加载并执行 JS（JSONP 原理） |
| `<link>` | 跨源 CSS |
| `<form action>` | 跨源提交表单（CSRF 根源） |
| `<a>` / location | 跨源导航 |

**「能加载 ≠ 能读取」**：
- script 能执行但拿不到响应源码（别人家厨师来炒菜，但拿不到菜谱）
- img 能显示但读不到像素 → **canvas 污染**：跨源图片画上 canvas 后 `toDataURL()`/`getImageData()` 抛 SecurityError
- 防什么：防恶意网页通过 img + canvas 读取并窃取其他网站需登录才可见的私密图片（加载不受限 + 浏览器自动带 Cookie → 图片能显示；读像素则被禁）
- 解除：图片加 `crossorigin="anonymous"` + 服务器返回 CORS 头（= 服务器显式授权）

### 1.4 表单提交不受限 → CSRF 根源（伏笔）
`form 提交不受限` + `浏览器自动带 Cookie` = 服务器无法区分请求是否用户本意。
攻击场景：攻击者页面藏隐藏表单 `action="https://bank.com/transfer"`，页面加载自动提交 → 浏览器带用户银行 Cookie → 银行以为是用户本人操作。
防御（第九章）：SameSite、CSRF Token、Origin 校验。

---

## 二、CORS（W3C 标准）

### 2.1 请求头 vs 响应头（易混，重点）
| 方向 | 头 | 作用 |
|------|-----|------|
| 请求（浏览器自动加） | `Origin: http://localhost:3000` | 告诉服务器请求来自哪个源 |
| 请求（仅预检） | `Access-Control-Request-Method` | 询问「我打算用这个方法」 |
| 请求（仅预检） | `Access-Control-Request-Headers` | 询问「我打算带这些头」 |
| 响应 | `Access-Control-Allow-Origin` | 允许哪些源读取 |
| 响应 | `Access-Control-Allow-Methods` | 允许哪些方法 |
| 响应 | `Access-Control-Allow-Headers` | 允许哪些请求头 |
| 响应 | `Access-Control-Allow-Credentials` | 是否允许携带 Cookie |
| 响应 | `Access-Control-Max-Age` | 预检结果缓存时长 |

**记忆**：`Origin` 是问句（请求侧）；`Access-Control-Allow-*` 是答句（响应侧）。浏览器检查的是**响应头** Allow-Origin。

### 2.2 简单请求 vs 预检请求
**简单请求 = 以下全部满足**：
1. 方法：仅 GET / HEAD / POST
2. Content-Type：仅 `application/x-www-form-urlencoded` / `multipart/form-data` / `text/plain`
3. 自定义头：仅安全名单（Accept、Accept-Language、Content-Language、Content-Type、Range）
4. 无 ReadableStream

**触发预检的常见元凶**：`Content-Type: application/json`、`Authorization` 头（fetch 带 token 必中）。

**设计理由**：简单请求 = 传统表单能发出的请求（表单一直不受限，拦也白拦）；预检针对「表单干不了的事」——服务器同意才发。

### 2.3 预检完整流程
```
OPTIONS /api/login  + Origin + Access-Control-Request-Method/Headers
→ 服务器回 204 + Allow-Origin + Allow-Methods + Allow-Headers + Max-Age
→ 通过后才发真实请求
→ 真实响应也必须带 Allow-Origin（+ Allow-Credentials）才把数据交给 JS
```
**两道关卡**：预检通过 ≠ 读取放行；真实响应也要带 Allow-Origin。

### 2.4 细节
- `Allow-Origin: *` 与 `Allow-Credentials: true` **不能同时用**（带凭证必须回显具体源）
- 跨域 fetch 默认不带 Cookie（credentials 默认 same-origin），要带：`credentials: 'include'` + 后端 Allow-Credentials: true + Cookie 本身条件
- **CORS 是读保护不是写保护**：请求发出即执行，副作用已发生，浏览器只拦「把响应交给 JS」——服务端不能把 CORS 当安全机制

### 2.5 预检性能优化
1. `Access-Control-Max-Age: 86400` 缓存预检结果（缓存 key = 方法 + 头组合）
2. **简单请求化**：Content-Type 降级 / token 走 Cookie——但简单请求 CSRF 面大，敏感接口别降级
3. **Nginx 同域反代**：彻底无 CORS（生产首选）
4. 排查：Network 面板看 OPTIONS 是否 2xx；常见失败 = Allow-Headers 没配 / 网关拦 OPTIONS

---

## 三、JSONP

### 3.1 原理（三步）
```
① 前端定义全局回调函数 handleUser（挂 window 上）
② <script src="https://api.example.com/user?callback=handleUser">
   └─ callback 参数 = 告诉服务器「把数据包进对这个函数的调用里」
③ 服务器读 callback 参数拼字符串：handleUser({"name":"张三","age":25})
④ 浏览器把响应当 script 执行 → 全局找到 handleUser → 调用 → 数据经参数到达
```
**为什么能绕过**：script 加载不受限 + 执行允许；数据不是「被读取」而是「执行代码时经函数参数送出」。执行效果里携带数据 ≠ 读取响应文本。

**类比**：CORS = 允许拆包裹（读取）；JSONP = 派传话的人当面念出内容（执行代码调用你的函数）——没拆包裹但内容已到手。

### 3.2 局限
- 只支持 GET（script 只能发 GET）
- 信任模型弱：服务器返回的是**可执行代码**，被黑/恶意 = 任意代码在你页面执行（XSS 载体）
- 错误处理差：无状态码、难分错误原因、超时需自己模拟
- 无标准协议（callback 参数名靠协商）
- 回调函数污染全局

### 3.3 与 CORS 对比
CORS：W3C 标准、全部方法、服务器显式声明、错误处理完善、当前首选。
JSONP：民间 hack、仅 GET、信任执行代码、已基本淘汰（老接口兼容）。

---

## 四、跨域方案全家桶（选型）

| 方案 | 原理 | 优点 | 缺点 | 适用 |
|------|------|------|------|------|
| CORS | 后端响应头声明 | 标准、灵活 | 预检开销、依赖后端 | 首选 |
| JSONP | script 回调 | 无需后端配置 | 仅 GET、不安全 | 老接口 |
| Nginx 反代 | 同域转发 | 浏览器零感知、无 CORS、可顺带鉴权/限流/HTTPS | 多一层运维 | **生产首选** |
| devServer 代理 | 开发期转发 | 零配置 | 仅开发环境 | 本地开发 |
| postMessage | 跨窗口发消息 | 安全可控 | 需双方配合 | iframe/微前端 |
| document.domain | 降级同源 | 简单 | 仅同主域子域、已废弃 | 淘汰 |
| window.name | 窗口名传数据 | 无需服务器 | 容量小 | 淘汰 |
| WebSocket | 协议不受限 | 全双工 | 非 HTTP | 实时通信 |

### 关键细节
- **postMessage 安全关键**：`targetOrigin` 参数 + 接收方 `event.origin` 校验，缺一个任意页面都能发/读消息
- **跨域带 Cookie 三条件**：前端 `credentials: 'include'` + 后端 `Allow-Credentials: true`（Allow-Origin 不能是 *）+ Cookie 的 SameSite 允许
- 面试标准回答：开发用 devServer 代理 → 生产用 Nginx 反代 → 第三方接口后端配 CORS 白名单

---

## 五、本章自查清单

1. 同源策略边界一句话？（加载 vs 读取）
2. canvas 污染防什么？怎么解除？
3. 简单请求的三个条件？触发预检的两大元凶？
4. Origin 和 Access-Control-Allow-Origin 各是什么头、谁加、检查谁？
5. 预检两道关卡？预检失败常见原因？
6. 为什么 Allow-Origin: * 不能配 Allow-Credentials: true？
7. JSONP 闭环怎么讲？（callback 参数 → 服务器拼代码 → 执行 → 参数到达）
8. CORS 为什么管读不管写？副作用请求靠什么防御？
9. 跨域方案选型标准答案？
10. postMessage 两个安全检查是什么？

## 六、误区纠正记录

| # | 错误理解 | 正确模型 |
|---|---------|---------|
| 1 | 同源限制加载、不限制读取 | 反了：不限制加载/引用，限制读取 |
| 2 | 检查响应头里的 Origin 字段 | Origin 是请求头；浏览器检查的是响应头 Access-Control-Allow-Origin |
| 3 | Cookie 按「源」存储携带 | 按「域名」存储携带，**端口不参与匹配**（localhost 各端口共享） |
| 4 | localStorage 和 Cookie 隔离单位相同 | Cookie 按域名；localStorage 按源（含端口）严格隔离 |
| 5 | 预检通过 = 放行读取 | 真实响应也必须带 Allow-Origin / Allow-Credentials |
| 6 | JSONP 是「读取」跨域数据 | 是「执行代码，数据经函数参数送出」，从未读取 |
| 7 | 跨域 fetch 默认带 Cookie（只是响应被拦） | 跨域 fetch 默认请求本身就不带 Cookie（credentials 默认 same-origin） |
| 8 | CORS 能防住副作用请求 | CORS 只保护读取；写操作防护靠 CSRF Token / SameSite / Origin 校验 |
| 9 | 简单请求的判断只看方法 | 方法 + Content-Type + 自定义头三条件同时满足 |
