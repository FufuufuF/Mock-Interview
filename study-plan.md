# 前端八股文 32 天学习大纲

> 目标：系统覆盖前端秋招面试核心知识点（不含算法）
> 节奏：每天 1 个主题，每个主题对应一次 `/study`
> 建议：每周末用 `/interview frontend` 做一次完整面试验证

---

## Week 1：JavaScript 核心（Day 1-7）

JS 是面试重中之重，占比最高，必须打牢。

| Day | 主题 | 关键词 |
|-----|------|--------|
| 1 | 作用域与闭包 | 词法作用域、闭包原理、经典闭包题、内存泄漏 |
| 2 | this 指向 | 默认/隐式/显式/new 绑定、箭头函数、优先级 |
| 3 | 原型与继承 | 原型链、`__proto__` vs `prototype`、各种继承方式、class 语法糖 |
| 4 | 异步编程（上） | 回调、Promise 原理、链式调用、错误处理、微任务 |
| 5 | 异步编程（下） | async/await 原理、Generator、并发控制、Promise 组合方法 |
| 6 | Event Loop | 宏任务/微任务、执行顺序、Node vs 浏览器差异、经典输出题 |
| 7 | ES6+ 核心特性 | let/const/var、解构、Proxy/Reflect、Map/Set/WeakMap、Symbol、模块化 |

---

## Week 2：CSS + 网络 + 浏览器（Day 8-14）

三个高频模块穿插安排，避免连续啃同一方向疲劳。

| Day | 主题 | 关键词 |
|-----|------|--------|
| 8 | CSS 盒模型与 BFC | 标准/IE 盒模型、margin 塌陷、BFC 触发条件与应用 |
| 9 | HTTP 协议 | HTTP/1.1 vs 2 vs 3、请求方法、状态码、头部字段、Keep-Alive |
| 10 | Flex 与 Grid 布局 | Flex 容器/项目属性、Grid 布局、常见布局实战（圣杯/双飞翼/居中） |
| 11 | HTTPS 与安全 | TLS 握手、证书链、对称/非对称加密、XSS/CSRF/CSP 防御 |
| 12 | 浏览器渲染流程 | 构建 DOM/CSSOM、渲染树、Layout/Paint/Composite、重排重绘优化 |
| 13 | HTTP 缓存与 DNS | 强缓存/协商缓存、Cache-Control、ETag、DNS 解析流程、CDN 原理 |
| 14 | 浏览器存储与跨域 | Cookie/Storage/IndexedDB、同源策略、CORS、JSONP、代理 |

---

## Week 3：React + Vue + 工程化（Day 15-23）

两大框架原理 + 现代前端工程化。

| Day | 主题 | 关键词 |
|-----|------|--------|
| 15 | React Hooks 原理 | useState/useEffect 实现、闭包陷阱、Hooks 规则、自定义 Hook |
| 16 | 虚拟 DOM 与 Diff | VDOM 结构、Diff 算法策略（tree/component/element）、key 的作用 |
| 17 | React Fiber 架构 | Fiber 节点结构、时间切片、调度优先级、可中断渲染、并发模式 |
| 18 | React 状态管理与性能优化 | Context/Redux/Zustand、memo/useMemo/useCallback、Suspense |
| 19 | Vue 响应式原理 | Object.defineProperty vs Proxy、依赖收集、派发更新、ref/reactive |
| 20 | Vue 组件机制 | 生命周期、组合式 API vs 选项式、nextTick 原理、组件通信方式 |
| 21 | Vue 生态与进阶 | Vue Router 原理、Pinia 状态管理、keep-alive、Vue3 编译优化 |
| 22 | Webpack 与 Vite | 构建流程、Loader/Plugin、Tree Shaking、HMR、Vite 为什么快 |
| 23 | 模块化与包管理 | ESM vs CJS vs UMD、循环依赖、monorepo、pnpm 原理 |

---

## Week 4：手写题 + 性能 + TypeScript + 综合（Day 24-32）

手写代码是面试硬指标，性能优化是进阶加分项。

| Day | 主题 | 关键词 |
|-----|------|--------|
| 24 | 手写 Promise | Promise/A+ 规范、then/catch/finally、resolve/reject/all/race |
| 25 | 手写工具函数 | 防抖/节流、深拷贝、柯里化、compose/pipe、flat |
| 26 | 手写 DOM/事件相关 | 事件委托、发布订阅、EventEmitter、虚拟列表原理 |
| 27 | 手写网络/异步相关 | Ajax 封装、并发请求限制、请求重试、取消请求（AbortController） |
| 28 | TypeScript 核心 | 泛型、类型体操、工具类型、type vs interface、类型收窄 |
| 29 | 性能优化（加载） | 首屏优化、资源压缩、懒加载、预加载、SSR/SSG |
| 30 | 性能优化（运行时） | 长列表虚拟滚动、Web Worker、requestAnimationFrame、内存泄漏排查 |
| 31 | 性能指标与监控 | Core Web Vitals（LCP/INP/CLS）、Performance API、埋点方案 |
| 32 | 综合串联 | 从输入 URL 到页面渲染全流程、项目难点表述、场景设计题练习 |

---

## 使用方式

每天按主题执行：

```
/study 作用域与闭包
/study Event Loop
/study React Fiber 架构
...
```

建议每周末做一次完整面试验证当周学习效果：

```
/interview frontend
```

## 节奏建议

- 每天投入 2-3 小时
- 如果某个主题学不完，允许延伸到第二天上午，但不要打乱整体进度
- Week 3 的 Vue 部分如果你日常用 React 更多，可以适当压缩；反之亦然
- Week 4 的手写题建议自己先写再对答案，不要直接看讲解
