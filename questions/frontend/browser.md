# 浏览器渲染机制

## 基础题

### Q: 请描述从用户在地址栏输入 URL 到页面完整显示的全过程
**难度**: 基础
**考察点**: 网络请求流程、DNS 解析、TCP 连接、浏览器渲染流水线
**追问方向**:
- DNS 解析过程中有哪些缓存层级？
- HTTPS 握手与 HTTP 握手有何不同？多了哪些步骤？
- 浏览器在收到 HTML 响应后，解析和渲染的具体阶段有哪些？

### Q: 浏览器中 localStorage、sessionStorage 和 Cookie 各有什么区别？它们的使用场景分别是什么？
**难度**: 基础
**考察点**: 浏览器存储机制、数据持久化、存储容量限制
**追问方向**:
- localStorage 存储的数据量上限是多少？超出后会怎样？
- sessionStorage 在多标签页场景下的表现是怎样的？
- IndexedDB 相比 localStorage 有哪些优势？适合存储什么类型的数据？

### Q: 什么是 DOM 树和 CSSOM 树？它们是如何构建的？
**难度**: 基础
**考察点**: DOM 解析、CSSOM 构建、渲染树生成
**追问方向**:
- HTML 解析过程中遇到 `<script>` 标签会发生什么？`async` 和 `defer` 有何区别？
- CSS 的加载会阻塞 DOM 的解析吗？会阻塞渲染吗？
- 渲染树（Render Tree）和 DOM 树有什么区别？`display: none` 的元素会出现在渲染树中吗？

### Q: 请解释浏览器事件循环（Event Loop）的运行机制，并说明宏任务和微任务的区别
**难度**: 基础
**考察点**: 事件循环、宏任务、微任务、异步执行顺序
**追问方向**:
- `Promise.then`、`setTimeout`、`requestAnimationFrame` 的执行顺序是怎样的？
- `MutationObserver` 属于宏任务还是微任务？
- Node.js 中的事件循环和浏览器中的有什么区别？

### Q: 请说明浏览器的多进程架构，每个进程分别负责什么？
**难度**: 基础
**考察点**: 浏览器进程模型、渲染进程、GPU 进程、网络进程
**追问方向**:
- 为什么 Chrome 要采用多进程而不是多线程架构？
- 渲染进程中包含哪些线程？它们各自的职责是什么？
- 一个标签页崩溃会影响其他标签页吗？什么情况下多个标签页会共享同一个渲染进程？

## 中等题

### Q: 什么是重排（回流）和重绘？哪些操作会触发重排？如何减少重排次数？
**难度**: 中等
**考察点**: 重排、重绘、性能优化、批量 DOM 操作
**追问方向**:
- 读取 `offsetWidth`、`clientHeight` 等属性为什么会强制触发重排？
- `DocumentFragment` 和虚拟 DOM 在减少重排方面的作用有何不同？
- `transform` 和 `opacity` 的修改为什么不会触发重排？

### Q: 什么是合成层（Compositing Layer）？哪些属性会创建新的合成层？GPU 加速的原理是什么？
**难度**: 中等
**考察点**: 合成层、GPU 加速、will-change、图层管理
**追问方向**:
- 过多的合成层会带来什么问题？什么是层爆炸（Layer Explosion）？
- `will-change` 属性应该怎样正确使用？滥用会有什么后果？
- 如何通过 Chrome DevTools 的 Layers 面板来调试合成层问题？

### Q: Web Worker 是什么？它与主线程之间如何通信？有哪些使用场景？
**难度**: 中等
**考察点**: Web Worker、多线程、postMessage、SharedArrayBuffer
**追问方向**:
- Web Worker 中能否操作 DOM？为什么？
- Dedicated Worker、Shared Worker 和 Service Worker 之间有什么区别？
- 主线程与 Worker 之间传递数据时，结构化克隆算法（Structured Clone）和 Transferable Objects 有什么区别？

### Q: 请写出以下代码的输出顺序并解释原因：`setTimeout(() => console.log(1), 0); Promise.resolve().then(() => console.log(2)); requestAnimationFrame(() => console.log(3)); console.log(4);`
**难度**: 中等
**考察点**: 事件循环执行顺序、微任务优先级、requestAnimationFrame 时机
**追问方向**:
- 如果在 `Promise.then` 回调中再添加一个 `Promise.then`，新的微任务会在什么时候执行？
- `requestAnimationFrame` 的回调在事件循环的哪个阶段执行？
- `queueMicrotask` 和 `Promise.then` 在微任务队列中的优先级一样吗？

### Q: IndexedDB 的事务机制是怎样的？在大数据量前端存储场景下，如何设计高效的读写方案？
**难度**: 中等
**考察点**: IndexedDB、事务、索引、游标、大数据存储
**追问方向**:
- IndexedDB 的事务有哪些类型？`readwrite` 和 `readonly` 事务在并发时的行为是怎样的？
- 如何为 IndexedDB 创建复合索引？什么时候需要用到游标（Cursor）？
- 在离线优先（Offline First）的应用中，如何设计 IndexedDB 与服务端的数据同步策略？

## 困难题

### Q: V8 引擎的垃圾回收机制是怎样的？请详细描述新生代和老生代的回收策略
**难度**: 困难
**考察点**: V8 垃圾回收、Scavenge 算法、Mark-Sweep、Mark-Compact、增量标记
**追问方向**:
- 新生代为什么使用 Scavenge（Cheney）算法而不是标记清除？空间换时间的代价如何权衡？
- 什么条件下对象会从新生代晋升到老生代？
- 增量标记（Incremental Marking）和并发标记（Concurrent Marking）是如何减少 GC 停顿时间的？三色标记法的原理是什么？

### Q: V8 引擎中 JIT（即时编译）的工作流程是怎样的？Ignition 解释器和 TurboFan 编译器如何协作？
**难度**: 困难
**考察点**: JIT 编译、Ignition、TurboFan、隐藏类（Hidden Class）、内联缓存（Inline Cache）
**追问方向**:
- 什么是反优化（Deoptimization）？哪些代码模式会导致 TurboFan 编译的代码被反优化？
- V8 的隐藏类（Hidden Class / Map）机制是什么？为什么对象属性的添加顺序会影响性能？
- 内联缓存（Inline Cache）有哪几种状态（Uninitialized、Monomorphic、Polymorphic、Megamorphic）？它们分别意味着什么？

### Q: 浏览器渲染流水线中，从 Layout 到 Paint 再到 Composite 的完整流程是怎样的？如何利用这些知识做极致的渲染性能优化？
**难度**: 困难
**考察点**: 渲染流水线、Layout、Paint、Composite、RAIL 模型、性能优化
**追问方向**:
- 什么是渲染流水线的「跳过」优化？哪些 CSS 属性的修改可以跳过 Layout 和 Paint 阶段？
- `contain` 属性（CSS Containment）如何帮助浏览器优化渲染性能？`content-visibility: auto` 的原理是什么？
- 在 60fps 的要求下，每帧留给 JavaScript 的执行时间大约是多少？如何利用 `requestIdleCallback` 做任务调度？

### Q: 请设计一个方案，实现一个包含 10 万条数据的虚拟滚动列表，要求滚动流畅且内存占用可控，请从浏览器渲染原理的角度分析你的优化策略
**难度**: 困难
**考察点**: 虚拟滚动、DOM 回收、合成层优化、IntersectionObserver、内存管理
**追问方向**:
- 如何处理列表项高度不固定的场景？动态高度计算会带来哪些额外的性能挑战？
- `IntersectionObserver` 在虚拟滚动中可以扮演什么角色？它与 `scroll` 事件监听相比有什么优势？
- 如何利用 `will-change`、`transform` 和合成层来优化滚动时的渲染性能？如何避免层爆炸？
