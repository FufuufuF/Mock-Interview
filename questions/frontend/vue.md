# Vue 框架

## 基础题

### Q: 请说明 Vue 2 和 Vue 3 的主要差异有哪些？
**难度**: 基础
**考察点**: Vue 版本演进、框架设计理念
**追问方向**:
- Vue 3 为什么选择用 Proxy 替代 Object.defineProperty？
- Vue 3 在 Tree-shaking 方面做了哪些改进？
- 从 Vue 2 迁移到 Vue 3 的主要成本和注意事项是什么？

### Q: Vue 的生命周期钩子有哪些？请分别说明它们的触发时机和典型使用场景。
**难度**: 基础
**考察点**: 生命周期、组件挂载流程、DOM 操作时机
**追问方向**:
- 父子组件的生命周期执行顺序是怎样的？
- setup() 和 created 的执行时机有什么区别？
- 在哪个钩子中发起异步请求最合适？为什么？

### Q: Vue 中有哪些组件通信方式？分别适用于什么场景？
**难度**: 基础
**考察点**: 组件通信、props/emit、provide/inject、事件总线
**追问方向**:
- props 和 attrs 的区别是什么？
- provide/inject 是响应式的吗？如何让它变成响应式？
- 在大型项目中如何选择组件通信方案？

### Q: computed 和 watch 有什么区别？各自适用什么场景？
**难度**: 基础
**考察点**: 计算属性、侦听器、依赖收集、副作用处理
**追问方向**:
- computed 的缓存机制是如何实现的？
- watchEffect 和 watch 的区别是什么？
- 什么场景下应该用 watch 的 immediate 和 deep 选项？

### Q: Vue Router 中 hash 模式和 history 模式有什么区别？
**难度**: 基础
**考察点**: 前端路由原理、浏览器 History API、hash 变化监听
**追问方向**:
- history 模式在生产环境部署时需要注意什么？
- 路由守卫有哪几种？执行顺序是怎样的？
- 如何实现路由懒加载？原理是什么？

### Q: 请对比 Vuex 和 Pinia 的设计差异，为什么 Vue 3 推荐使用 Pinia？
**难度**: 基础
**考察点**: 状态管理、Vuex 模块化、Pinia 设计理念
**追问方向**:
- Pinia 去掉了 mutations，为什么这样设计是合理的？
- Vuex 中为什么要区分 mutation 和 action？
- 在什么体量的项目中需要引入状态管理？

## 中等题

### Q: 请详细解释 Vue 2 中 Object.defineProperty 和 Vue 3 中 Proxy 实现响应式的原理及差异。
**难度**: 中等
**考察点**: 响应式系统、数据劫持、依赖收集与派发更新
**追问方向**:
- Vue 2 对数组的响应式处理做了哪些特殊处理？为什么不能直接用下标修改数组？
- Proxy 相比 Object.defineProperty 在性能上有哪些优势？
- reactive 和 ref 的区别是什么？为什么需要两种响应式 API？

### Q: 组合式 API（Composition API）和选项式 API（Options API）各有什么优劣？在实际项目中如何选择？
**难度**: 中等
**考察点**: 代码组织方式、逻辑复用、TypeScript 支持
**追问方向**:
- 组合式 API 如何解决 mixins 的缺陷？
- 如何设计一个高复用性的 composable 函数？请举例说明。
- 在团队协作中，组合式 API 带来了哪些新的代码规范挑战？

### Q: 请解释 Vue 中虚拟 DOM 的作用以及 Diff 算法的基本流程。
**难度**: 中等
**考察点**: 虚拟 DOM、Diff 算法、同层比较、key 的作用
**追问方向**:
- 为什么需要虚拟 DOM？直接操作真实 DOM 有什么问题？
- v-for 中的 key 为什么不推荐用 index？对 Diff 算法有什么影响？
- Vue 3 的 Diff 算法相比 Vue 2 有哪些优化？

### Q: 请解释 nextTick 的实现原理及使用场景。
**难度**: 中等
**考察点**: 异步更新队列、微任务/宏任务、事件循环
**追问方向**:
- Vue 为什么要采用异步更新策略而不是同步更新 DOM？
- nextTick 在不同浏览器环境下的降级策略是怎样的？
- 连续多次修改同一个响应式数据，DOM 会更新几次？为什么？

### Q: Vue 3 的编译阶段做了哪些优化？请解释静态提升和 PatchFlag 的概念。
**难度**: 中等
**考察点**: 编译优化、静态提升、PatchFlag、Block Tree
**追问方向**:
- 静态提升（hoistStatic）具体是把什么提升到了哪里？
- PatchFlag 的不同标记分别代表什么含义？
- Block Tree 和传统的 VNode Tree 有什么区别？它解决了什么问题？

### Q: Vue 3 中 ref、reactive、shallowRef、shallowReactive 各自的使用场景是什么？
**难度**: 中等
**考察点**: 响应式 API、深层/浅层响应式、性能优化
**追问方向**:
- toRef 和 toRefs 的作用是什么？解构 reactive 对象时为什么会丢失响应性？
- shallowRef 配合 triggerRef 的使用场景是什么？
- 大型对象或列表数据应该如何选择响应式方案以优化性能？

## 困难题

### Q: 请从源码层面描述 Vue 3 响应式系统中依赖收集与派发更新的完整流程。
**难度**: 困难
**考察点**: effect、track、trigger、依赖关系图、WeakMap/Map/Set 数据结构
**追问方向**:
- targetMap、depsMap、dep 之间的关系是怎样的？为什么使用 WeakMap？
- effect 的调度器（scheduler）是如何工作的？computed 是怎么利用 scheduler 实现懒求值的？
- Vue 3 如何解决循环依赖和无限递归更新的问题？

### Q: 请深入分析 Vue 3 的 Diff 算法中最长递增子序列的应用及其优化原理。
**难度**: 困难
**考察点**: 双端对比、最长递增子序列（LIS）、节点移动最小化
**追问方向**:
- 为什么使用最长递增子序列可以减少 DOM 移动操作？
- 请描述从"头头、尾尾、头尾、尾头"对比到 LIS 求解的完整流程。
- 与 React 的 Fiber Diff 相比，Vue 3 的 Diff 策略有什么优劣？

### Q: 如何设计一个高性能的 Vue 3 大型列表渲染方案？请从虚拟滚动、响应式优化、编译优化等多个层面分析。
**难度**: 困难
**考察点**: 虚拟滚动、shallowRef/shallowReactive、v-memo、组件拆分策略
**追问方向**:
- 虚拟滚动中不定高度元素的处理方案有哪些？各有什么优缺点？
- v-memo 和 v-once 的区别是什么？v-memo 的缓存失效策略是怎样的？
- 在 SSR 场景下大型列表的渲染策略有什么不同？

### Q: Vue 3 的编译器是如何将模板转换为渲染函数的？请描述 parse、transform、codegen 三个阶段的核心逻辑。
**难度**: 困难
**考察点**: 编译器原理、AST 生成、转换插件、代码生成
**追问方向**:
- 模板编译的 AST 和 JavaScript 的 AST 有什么区别？
- transform 阶段的节点转换和指令转换是如何协作的？
- Vue 的编译时优化（如 Block Tree、PatchFlag）是在哪个阶段注入的？
