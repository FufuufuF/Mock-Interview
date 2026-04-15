# 性能优化

## 基础题

### Q1: 请介绍前端核心性能指标（Core Web Vitals），并说明每个指标的含义和合格标准。
**难度**: 基础
**考察点**: FCP、LCP、TTI、CLS、FID、性能指标体系
**追问方向**:
- FCP 和 LCP 的区别是什么？LCP 通常由页面中的哪些元素决定？
- CLS 是如何计算的？哪些操作容易导致布局偏移？
- FID 即将被 INP（Interaction to Next Paint）取代，你了解 INP 吗？它解决了 FID 的什么不足？

### Q2: 什么是懒加载（Lazy Loading）？在前端中有哪些常见的懒加载实现方式？
**难度**: 基础
**考察点**: 懒加载、Intersection Observer、动态 import、加载优化
**追问方向**:
- 图片懒加载使用 Intersection Observer 和传统的 scroll 监听方案各有什么优缺点？
- React.lazy 和 Vue 的异步组件在实现懒加载时有什么区别？
- 原生 HTML 的 `loading="lazy"` 属性有什么局限性？

### Q3: 请解释浏览器的重排（Reflow）和重绘（Repaint），以及如何减少它们的发生？
**难度**: 基础
**考察点**: 重排、重绘、渲染机制、DOM 操作优化
**追问方向**:
- 哪些 CSS 属性的修改会触发重排，哪些只触发重绘？
- 为什么读取 offsetWidth、clientHeight 等属性会强制触发同步布局（Forced Synchronous Layout）？
- 使用 CSS 的 `transform` 和 `opacity` 做动画为什么性能更好？它们是如何利用 GPU 合成层的？

### Q4: 常见的图片优化策略有哪些？如何根据场景选择合适的图片格式？
**难度**: 基础
**考察点**: 图片格式、图片压缩、响应式图片、WebP、AVIF
**追问方向**:
- WebP 和 AVIF 相比传统的 JPEG/PNG 有什么优势？兼容性如何处理？
- `<picture>` 标签和 `srcset` 属性分别适用于什么场景？
- 如何实现图片的渐进式加载（Progressive Loading）？低质量图片占位（LQIP）方案是怎么做的？

### Q5: 请介绍浏览器缓存策略，强缓存和协商缓存分别是如何工作的？
**难度**: 基础
**考察点**: HTTP 缓存、Cache-Control、ETag、Last-Modified、CDN 缓存
**追问方向**:
- `Cache-Control` 中 `no-cache` 和 `no-store` 的区别是什么？
- 在使用 Webpack 打包时，文件名 hash 策略（contenthash/chunkhash/hash）是如何配合缓存的？
- Service Worker 缓存和 HTTP 缓存的优先级关系是什么？

## 中等题

### Q6: 请详细说明 Tree Shaking 的原理，以及在实际项目中如何确保它正常工作？
**难度**: 中等
**考察点**: Tree Shaking、ES Module 静态分析、sideEffects、打包优化
**追问方向**:
- 为什么 Tree Shaking 依赖 ES Module 而不能用于 CommonJS？
- `package.json` 中的 `sideEffects` 字段有什么作用？如果配置不当会导致什么问题？
- 如何判断某个第三方库是否支持 Tree Shaking？遇到不支持的库该怎么处理？

### Q7: 请对比 Webpack 和 Vite 在构建优化方面的差异，各自有哪些关键的性能优化手段？
**难度**: 中等
**考察点**: Webpack 配置优化、Vite 原理、构建工具、开发体验
**追问方向**:
- Vite 在开发环境使用原生 ESM 的方式有什么优势和局限？
- Webpack 的持久化缓存（Persistent Caching）是如何提升二次构建速度的？
- 如何优化 Webpack 的打包体积？请至少说出五种具体做法。

### Q8: 请设计一个完整的首屏加载优化方案，从网络请求到页面渲染的全链路进行分析。
**难度**: 中等
**考察点**: 首屏优化、关键渲染路径、SSR/SSG、资源优先级、预加载
**追问方向**:
- SSR、SSG、ISR 三种渲染模式各自适用于什么场景？
- `<link rel="preload">` 和 `<link rel="prefetch">` 的区别和使用场景是什么？
- 如何利用 HTTP/2 Server Push 优化首屏？它存在什么问题导致 Chrome 后来移除了支持？

### Q9: 如何利用 Code Splitting 优化单页应用的加载性能？请结合路由级和组件级两个层面进行说明。
**难度**: 中等
**考察点**: 代码分割、动态 import、路由懒加载、按需加载、chunk 策略
**追问方向**:
- Webpack 中 `splitChunks` 的配置策略应该如何设计？`chunks: 'all'` 和 `chunks: 'async'` 有什么区别？
- 如何避免代码分割后产生过多小 chunk？合理的 chunk 粒度应该怎么把握？
- 动态 import 配合 Magic Comment（如 `webpackChunkName`、`webpackPrefetch`）能做哪些优化？

### Q10: 请介绍 requestAnimationFrame 的工作机制，以及它在性能优化中有哪些实际应用？
**难度**: 中等
**考察点**: requestAnimationFrame、事件循环、动画性能、帧率优化
**追问方向**:
- requestAnimationFrame 和 setTimeout/setInterval 做动画的本质区别是什么？
- 如何用 requestAnimationFrame 实现节流（throttle）？相比传统的时间戳节流有什么优势？
- 在大量 DOM 操作的场景下，如何使用 requestAnimationFrame 进行分帧渲染避免页面卡顿？

## 困难题

### Q11: 如何系统性地排查前端内存泄漏问题？请结合具体场景和工具进行说明。
**难度**: 困难
**考察点**: 内存泄漏、Chrome DevTools Memory 面板、堆快照、垃圾回收机制
**追问方向**:
- 前端常见的内存泄漏场景有哪些（闭包、事件监听、定时器、DOM 引用、全局变量）？请逐一分析原因和解决方案。
- 如何使用 Chrome DevTools 的 Heap Snapshot 和 Allocation Timeline 定位内存泄漏？
- 在 SPA 应用中，路由切换后组件未正确销毁导致的内存泄漏应该如何排查和修复？WeakRef 和 FinalizationRegistry 在处理内存问题时有什么作用？

### Q12: 请设计一套完整的前端性能监控方案，覆盖数据采集、上报、分析和告警的全流程。
**难度**: 困难
**考察点**: Performance API、PerformanceObserver、性能监控、Lighthouse、数据分析
**追问方向**:
- 如何使用 PerformanceObserver 采集 LCP、FID、CLS 等指标？与 `performance.timing` 相比有什么优势？
- 性能数据上报时如何设计采样策略和上报方式（sendBeacon vs XMLHttpRequest）？
- 如何设定性能基线和告警阈值？如何区分实验室数据（Lab Data）和真实用户数据（Field Data）？

### Q13: 在一个大型电商网站中，列表页有上万条商品数据需要展示，请设计一套完整的性能优化方案。
**难度**: 困难
**考察点**: 虚拟滚动、长列表优化、分页策略、Web Worker、渲染性能
**追问方向**:
- 虚拟滚动（Virtual Scrolling）的核心实现原理是什么？如何处理不定高度的列表项？
- 如何利用 Web Worker 将数据处理和排序等计算密集型任务从主线程中剥离？
- 在低端设备上还可以做哪些额外的降级优化？如何实现自适应的性能策略（Adaptive Loading）？

### Q14: 请深入分析 HTTP/2 对前端性能优化的影响，以及在 HTTP/2 环境下传统优化策略需要做哪些调整？
**难度**: 困难
**考察点**: HTTP/2 多路复用、头部压缩、CDN 优化、网络协议、HTTP/3
**追问方向**:
- HTTP/2 的多路复用如何解决 HTTP/1.1 的队头阻塞问题？为什么在 TCP 层面仍然存在队头阻塞？
- HTTP/2 环境下，域名分片、雪碧图、文件合并等传统优化是否还有必要？为什么？
- HTTP/3（QUIC）相比 HTTP/2 在性能方面有哪些改进？对前端开发有什么实际影响？

### Q15: 请从浏览器渲染管线的角度，解释一个复杂动画页面出现掉帧卡顿时的排查和优化思路。
**难度**: 困难
**考察点**: 渲染管线、合成层、GPU 加速、Performance 面板、帧率分析
**追问方向**:
- 浏览器渲染管线的五个阶段（Style -> Layout -> Paint -> Composite -> Display）中，哪些阶段最耗时？如何跳过不必要的阶段？
- 什么是层爆炸（Layer Explosion）？如何使用 `will-change` 属性提升合成层，同时避免滥用？
- 如何使用 Chrome DevTools 的 Performance 面板和 Layers 面板定位渲染瓶颈？如何识别长任务（Long Task）并进行优化？
