# React 框架

## 基础题

### Q: 请解释 JSX 的本质是什么？浏览器能直接执行 JSX 吗？
**难度**: 基础
**考察点**: JSX 编译原理、React.createElement、Babel 转译
**追问方向**:
- JSX 经过 Babel 编译后会变成什么样的代码？
- React 17 之后的新 JSX Transform 和旧版有什么区别？
- 为什么在 React 17 之前每个使用 JSX 的文件都需要引入 React？

### Q: 请描述虚拟 DOM 是什么，以及 React 为什么要引入虚拟 DOM？
**难度**: 基础
**考察点**: 虚拟 DOM 概念、性能优化、跨平台渲染
**追问方向**:
- 虚拟 DOM 一定比直接操作真实 DOM 快吗？在什么场景下可能更慢？
- 虚拟 DOM 的结构是什么样的？包含哪些关键属性？
- 除了性能之外，虚拟 DOM 还带来了哪些好处？

### Q: React 组件的生命周期有哪些阶段？类组件和函数组件在生命周期处理上有何不同？
**难度**: 基础
**考察点**: 生命周期方法、useEffect 模拟生命周期、挂载/更新/卸载阶段
**追问方向**:
- getDerivedStateFromProps 和 componentWillReceiveProps 有什么区别？为什么后者被废弃？
- useEffect 的依赖数组为空和不传依赖数组分别对应类组件的哪个生命周期？
- getSnapshotBeforeUpdate 的使用场景是什么？

### Q: 请解释 useState 的工作原理。为什么 useState 不能在条件语句或循环中调用？
**难度**: 基础
**考察点**: Hooks 调用规则、链表结构、闭包
**追问方向**:
- useState 的初始值如果传入一个函数和传入一个值有什么区别？
- setState 是同步的还是异步的？React 18 之后有什么变化？
- 连续调用多次 setState 会触发几次渲染？为什么？

### Q: useRef 的作用是什么？它和 useState 有什么本质区别？
**难度**: 基础
**考察点**: useRef 原理、可变引用、DOM 引用、跨渲染周期持久化
**追问方向**:
- 修改 useRef 的 current 值会触发组件重新渲染吗？为什么？
- forwardRef 的作用是什么？什么时候需要用到它？
- useRef 除了获取 DOM 引用之外还有哪些常见用途？

### Q: React 中 key 的作用是什么？为什么不推荐使用数组的 index 作为 key？
**难度**: 基础
**考察点**: Diff 算法、列表渲染、DOM 复用
**追问方向**:
- 在什么情况下使用 index 作为 key 不会有问题？
- 如果不提供 key，React 会怎么处理？
- key 值改变时，React 对该组件做了什么？

## 中等题

### Q: 请详细说明 React 的 Diff 算法的策略和时间复杂度优化。
**难度**: 中等
**考察点**: 三层 Diff 策略、同层比较、类型比较、key 匹配
**追问方向**:
- React 的 Diff 算法是如何将传统树对比的 O(n^3) 降低到 O(n) 的？
- 单节点 Diff 和多节点 Diff 的处理流程有什么不同？
- Vue 的双端对比算法和 React 的 Diff 有什么区别？

### Q: 请解释 useEffect 和 useLayoutEffect 的区别，以及它们各自的执行时机。
**难度**: 中等
**考察点**: 副作用执行时机、浏览器绘制流程、同步与异步
**追问方向**:
- useLayoutEffect 在什么具体场景下是必须使用的？
- useEffect 的清除函数在什么时候执行？执行顺序是怎样的？
- 在 SSR 环境下使用 useLayoutEffect 会有什么问题？如何解决？

### Q: useMemo 和 useCallback 分别解决什么问题？请结合 React.memo 说明它们在性能优化中的配合使用。
**难度**: 中等
**考察点**: 引用稳定性、记忆化、渲染优化、React.memo 浅比较
**追问方向**:
- 是否应该给每个函数都包裹 useCallback、每个计算都包裹 useMemo？为什么？
- React.memo 的第二个参数有什么作用？和 shouldComponentUpdate 有什么关系？
- React Compiler（React Forget）出来之后，手动记忆化还有意义吗？

### Q: 请对比 Redux、Zustand、Jotai 这三种状态管理方案的设计理念和适用场景。
**难度**: 中等
**考察点**: 状态管理架构、单一数据源、原子化状态、发布订阅模式
**追问方向**:
- Redux 中间件的实现原理是什么？洋葱模型是如何工作的？
- Zustand 是如何实现在 React 外部也能访问和修改状态的？
- 在大型项目中如何选择合适的状态管理方案？选型依据有哪些？

### Q: React Router 的前端路由原理是什么？hash 模式和 history 模式有什么区别？
**难度**: 中等
**考察点**: 前端路由原理、History API、hash 变化监听、路由匹配
**追问方向**:
- React Router v6 相比 v5 做了哪些重大变化？
- 嵌套路由和 Outlet 的工作机制是什么？
- 如何实现路由级别的权限控制和路由守卫？

### Q: React.lazy 和 Suspense 是如何实现组件懒加载的？请描述其内部机制。
**难度**: 中等
**考察点**: 代码分割、动态 import、Suspense 捕获机制、加载状态
**追问方向**:
- Suspense 是如何知道子组件正在加载中的？底层是什么机制？
- 如何对懒加载的组件进行预加载（prefetch）？
- 在路由级别做代码分割时有哪些最佳实践？

## 困难题

### Q: 请深入解释 React Fiber 架构的设计动机和工作原理。Fiber 如何实现可中断的渲染？
**难度**: 困难
**考察点**: Fiber 节点结构、双缓冲树、工作循环、时间切片、任务优先级
**追问方向**:
- Fiber 树的 child、sibling、return 指针分别指向什么？为什么要用链表而非树结构遍历？
- requestIdleCallback 和 React 实际使用的调度机制有什么区别？为什么 React 没有直接使用 requestIdleCallback？
- Fiber 的协调阶段（Reconciliation）和提交阶段（Commit）分别做了什么？为什么提交阶段不能中断？

### Q: React 的 Concurrent Mode 解决了什么问题？请解释 useTransition 和 useDeferredValue 的工作机制和使用场景。
**难度**: 困难
**考察点**: 并发渲染、优先级调度、可中断更新、用户体验优化
**追问方向**:
- startTransition 标记的更新和普通更新在调度优先级上有什么区别？
- useDeferredValue 和防抖（debounce）/节流（throttle）有什么本质区别？
- Concurrent Mode 下可能出现的 tearing（撕裂）问题是什么？useSyncExternalStore 是如何解决这个问题的？

### Q: 请从架构层面详细说明 React SSR 的完整流程。Next.js 的 App Router 中 Server Components 和 Client Components 是如何协作的？
**难度**: 困难
**考察点**: SSR 渲染流程、Hydration、RSC 协议、流式渲染、Selective Hydration
**追问方向**:
- Hydration 的过程是什么？为什么会出现 Hydration Mismatch？如何排查和解决？
- React Server Components 和传统 SSR 有什么本质区别？RSC 的序列化协议是什么？
- Streaming SSR 和传统 SSR 在性能指标（TTFB、FCP、TTI）上各有什么优势？

### Q: 请从源码角度分析 Hooks 的实现原理。为什么 Hooks 必须在组件顶层调用？它们在 Fiber 节点上是如何存储的？
**难度**: 困难
**考察点**: Hooks 链表结构、memoizedState、dispatcher 切换、mount 与 update 阶段
**追问方向**:
- mount 阶段和 update 阶段的 dispatcher 有什么不同？React 是如何切换的？
- useEffect 的依赖比较使用的是什么算法？它和深比较有什么区别？存在什么潜在问题？
- 自定义 Hooks 的状态是共享的还是隔离的？请从 Fiber 节点的角度解释原因。
