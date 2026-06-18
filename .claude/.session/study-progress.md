# 学习会话进度记录

> 最后更新：2026-06-17
> 主题：JavaScript 异步（从协程角度深入学习）
> 状态：进行中，完成维度 3-4，待继续维度 5-6

---

## 本次学习概况

### 面试检验（2026-06-11 记录已保存）

今天先做了一场面试检验，结果为 D（5 道题，候选人主动结束）。暴露出：
- CSS（BFC）持续黑洞
- 浏览器渲染机制（DOM/CSSOM 构建）完全空白
- async/await 无法作答

随后启动了 JS 异步的系统性学习。

---

## 学习主题：JavaScript 异步

### 学习路线（6 个维度）

| 维度 | 内容 | 状态 |
|------|------|------|
| 1 | 异步的根源（JS 单线程） | ✅ 跳过（已掌握） |
| 2 | 三代演进（回调→Promise→async/await） | ✅ 跳过（已掌握） |
| 3 | **Promise 链式调用机制 + 手写 then** | ✅ 已完成 |
| 4 | **Generator + 协程 + async/await 本质** | ❌ 未开始 |
| 5 | **事件循环精确模型** | ❌ 未开始 |
| 6 | **代码输出题实战** | ❌ 未开始 |

---

## 维度 3 完成内容（Promise）

### 已讲解并理解的知识点：

1. **then 方法的两条核心规则**
   - then 必须返回一个**新的** Promise（不能返回 this）
   - 新 Promise 的状态由 then 回调的返回值决定（普通值→直接 resolve / Promise→跟随 / throw→reject）

2. **为什么需要 callbacks 列表**
   - 同一个 Promise 可以被多次 .then（分叉调用）
   - 每次 then 注册一个独立的回调，状态切换时一起触发
   - 类比：事件订阅模式（只能触发一次的事件）

3. **完整的 Promise 手写代码（带 reject + error 处理）**
   - 构造函数：state/value/reason/两个 callbacks 数组/resolve&reject 函数/executor try-catch
   - then 方法：参数容错（透传）/ 返回新 Promise / handleFulfilled&handleRejected / queueMicrotask / resolvePromise
   - catch：本质是 `this.then(undefined, onRejected)`
   - resolvePromise 辅助函数：处理返回值是 Promise 时的"跟随"

4. **关键细节**
   - 状态不可逆（resolve/reject 里先判断 state !== 'pending'）
   - then 回调必须异步（queueMicrotask 包裹）
   - onRejected 正常执行（没抛错）会让新 Promise 变 fulfilled → 这是 .catch 后能继续 .then 的原因
   - executor 是同步执行的

5. **then 方法中的 this.value**
   - 指的是"调用 then 的那个 Promise"（老 Promise）的值
   - 作为输入喂给回调函数
   - 回调的返回值决定新 Promise 的状态
   - "输入来自老 Promise，输出决定新 Promise" — then 方法的灵魂

6. **为什么 then 必须返回新 Promise**
   - Promise 状态不可逆，返回 this 则回调返回值无法传递下游
   - 保持不可变性（类似数组 map）
   - 支持分叉调用（同一个 Promise 多次 then 互不影响）

7. **静态方法速览**（all/race/allSettled/any 对比 + all 手撕核心思路）

### 学生表现：
- callbacks 列表的必要性：提出了"一个 promise 不是只有一个回调吗"的疑问，讲解后理解
- this.value 归属：提出了精确的疑问（是老 Promise 还是新 Promise 的），讲解后理解
- 为什么返回新 Promise：能独立推理出"如果返回 this 则回调返回值不生效"

---

## 维度 4 完成内容（协程 + Generator + async/await）

### 已讲解并理解的知识点：

1. **进程/线程/协程的层次关系**
   - 协程是用户态调度、极低切换成本、协作式让出
   - 协程 = 可以暂停和恢复的函数

2. **协程核心思想**
   - 暂停时保存局部变量和执行位置
   - 恢复时从精确位置继续
   - 解决异步代码的状态保留问题

3. **Generator = JS 的"半协程"实现**
   - 半协程：只能在 yield 处让出，不能跨函数让出
   - function* / yield / gen() 返回迭代器 / .next() 唤醒
   - 双向通信：yield 输出值，next(arg) 输入值

4. **从 Generator 到 async/await 的完整推导**
   - 手写了 asyncToGenerator 自动执行器
   - 核心逻辑：step 递归，每次 next 拿到 yield 出的 Promise，等它 resolved 后传结果回 next 唤醒
   - **async 函数 = Generator 函数 + 内置自动调度器**
   - **await = 被自动驱动的 yield**

5. **为什么 await 之后的代码是微任务（协程视角）**
   - await 时协程让出控制权 + 让出一个 Promise
   - 调度器注册 .then(v => step('next', v))
   - .then 回调进入微任务队列
   - 微任务被调度时，调度器唤醒协程
   - 协程从 await 处恢复执行

6. **JS vs Python 协程对比**
   - 两边演进路径几乎相同：Generator → 自动调度器 → async/await
   - 差异：事件循环暴露程度（Python 显式 / JS 隐式）、await 的东西（Future vs Promise）
   - 学生有 Python asyncio 背景，类比理解很快

7. **加分细节**
   - async 函数返回值永远是 Promise
   - await 任何值都会被包成 Promise
   - await 前是同步，await 后是微任务
   - 串行 await vs 并行 Promise.all 的性能差距

### 学生表现：
- 主动提出从协程角度学习（非常好的学习直觉）
- 有 Python asyncio 背景，能类比理解
- 核心概念都已讲解完毕，但尚未做题验证

---

## 下次继续的内容

### 维度 5：事件循环精确模型

需要讲的：
- 修正学生之前的描述（不是"宏任务→微任务→渲染"严格循环）
- 精确模型：每跑完一个宏任务 → 清空所有微任务（包括过程中新增的）→ 可能渲染
- 宏任务来源：setTimeout/setInterval、I/O、UI rendering、MessageChannel、requestAnimationFrame
- 微任务来源：Promise.then/catch/finally、queueMicrotask、MutationObserver
- requestAnimationFrame 的位置（渲染前，不算宏也不算微）
- Node.js 事件循环与浏览器的差异（如果时间允许）

### 维度 6：代码输出题实战

需要做的：
- 综合题：setTimeout + Promise.then + async/await + console.log 混合
- 逐步分析执行顺序的方法论
- 至少 3-4 道题，从简到难
- 包含上次面试没答的那道题（async function foo/bar 嵌套）

---

## 学生知识状态总结

### 已牢固掌握：
- JS 单线程 + 异步动机
- 回调地狱 + 控制反转问题
- Promise 三状态 + 状态不可逆
- Promise 链式调用机制（then 返回新 Promise、resolvePromise）
- callbacks 列表的必要性
- 完整 Promise 手写（含 error 处理）
- 协程概念 + Generator 是半协程
- async/await = Generator + 自动执行器
- await 之后代码是微任务的原因（协程视角）

### 需要在维度 5-6 巩固：
- 事件循环的精确执行模型
- 代码输出题的系统性分析方法
- 实际做题验证理解

### 整体学习之外仍需补齐的模块（按优先级）：
1. 🔥 CSS（BFC、Flex、盒模型）— 连续两次面试 D
2. 🔥 浏览器渲染流水线（DOM/CSSOM 构建、重排重绘深度、合成层）
3. 原型与继承 follow-up（instanceof 手写、Mixin）

---

## 面试记录已保存

- `records/markdown/2026-06-11-frontend.md`
- `records/json/2026-06-11-frontend.json`

---

## 恢复指令

下次新对话开始时，读取此文件后：
1. 直接进入维度 5（事件循环精确模型）
2. 保持面试辅导老师角色，互动式教学
3. 学生有 Python asyncio 背景，可以类比
4. 维度 5 讲完后直接进入维度 6 做题
5. 全部完成后保存学习记录到 records/study/ 下
