# React Hooks 底层机制深度学习

> 学习日期：2026-08-11
> 主题：React Hooks 底层机制（存储结构 → Dispatcher → 更新队列 → 调度 → 执行时机）
> 方式：互动式教学（6 个教学步骤 + 多次深入追问）

---

## 零、全景图：一句话模型

```
组件渲染 → fiber.memoizedState 上的 Hook 链表（按调用顺序对号入座）
    ↓
setState → ① update 入队（记账）→ ② lanes 冒泡（报总部）→ ③ 调度去重（决定时机）
    ↓
render（可中断）：bailout 跳过无关子树 → 执行组件函数 → 消费队列 → 生成新树
    ↓
commit（不可中断）：切树 → 更新 DOM → layout 副作用同步执行 → passive 副作用异步执行
```

四个层面完全贯通：**事件循环、调度器、fiber 树、hook 队列**。

---

## 一、Hook 在 Fiber 上的存储结构

### 1.1 两个核心数据结构

```ts
// Fiber 节点（简化）
interface Fiber {
  memoizedState: Hook | null;   // 指向该组件【第一个】Hook 节点
  updateQueue: EffectQueue | null;  // effect 链表（useEffect 用）
  flags: number;                // 副作用标记
  child / sibling / return: Fiber | null;  // 树结构指针
}

// Hook 节点（简化）
interface Hook {
  memoizedState: any;   // 每个 hook 存的东西不一样：
                        // useState → 存 state 值
                        // useRef   → 存 { current: x } 对象
                        // useMemo  → 存 [缓存值, deps]
                        // useEffect → 存 effect 对象
  queue: UpdateQueue | null;  // 更新队列（useState/useReducer 用）
  next: Hook | null;          // 下一个 Hook 节点
}
```

一个组件里所有 Hook 组成**单向链表**，挂在 `fiber.memoizedState` 上。

### 1.2 核心结论：Hook 没有名字，只有"位置"

- **mount（首次渲染）**：每调用一个 Hook 就新建节点、尾部插入
- **update（更新渲染）**：从上次链表上按顺序取节点复用（复制到 workInProgress fiber）
- React 完全不关心 Hook 叫什么，只按"第 N 次调用"对应"链表第 N 个节点"

### 1.3 为什么不能条件调用（严谨回答）

```jsx
function Bad() {
  const [a] = useState(1);
  if (someFlag) {
    const [b] = useState(2);   // 某次渲染不执行！
  }
  const [c] = useState(3);
}
```

- **跳过第 3 个**（尾部）：前两个正常，第 3 个的节点变成孤儿，状态滞留
- **跳过第 2 个**（中间）：**后面的全部错位一格**——第 3 个 useState 读到第 2 个的状态（串台）
- **dev 模式**：渲染结束后 `renderWithHooks` 检查链表是否被完整消费，对不上直接抛错：
  - `Rendered fewer hooks than during the previous render.`（调用少了）
  - `Rendered more hooks than during the previous render.`（调用多了）
- **生产模式**：无此检查，**静默串台**——这就是 ESLint `rules-of-hooks` 是硬性要求的原因

### 1.4 顺带可答的高频题

> 自定义 Hook 的状态是共享的还是隔离的？

**隔离的**。每个组件实例有独立的 fiber 节点，自定义 Hook 只是"打包若干次 Hook 调用"，状态存在调用它的那个组件的 fiber 链表里。

---

## 二、双 Dispatcher 机制

### 2.1 问题

组件代码里写的都是同一个 `useState`，凭什么 mount 时"新建节点"、update 时"遍历队列"？

### 2.2 答案：全局指针 + 两套实现（策略模式）

```js
const HooksDispatcherOnMount = {
  useState: mountState,   // 新建节点、写初始值
  useEffect: mountEffect,
  ...
};
const HooksDispatcherOnUpdate = {
  useState: updateState,  // 遍历队列、算新值
  useEffect: updateEffect,
  ...
};
// 全局指针：ReactCurrentDispatcher.current 指向当前激活的那套
```

### 2.3 切换时机：renderWithHooks 的开头

```js
function renderWithHooks(current, workInProgress, Component, props) {
  // ① 执行组件函数【之前】切换 dispatcher
  ReactCurrentDispatcher.current =
    current === null || current.memoizedState === null
      ? HooksDispatcherOnMount     // 首次渲染
      : HooksDispatcherOnUpdate;   // 更新渲染

  // ② 执行组件函数（函数体内所有 Hook 调用经全局指针转发）
  const children = Component(props);

  // ③（dev）校验链表消费完整性（第一节的报错）
  // ④ 渲染结束：current 重置为 ContextOnlyDispatcher（全抛错实现）
}
```

### 2.4 `useState` 本身只是个转发器

```js
export function useState(initialState) {
  const dispatcher = resolveDispatcher();   // 读 ReactCurrentDispatcher.current
  return dispatcher.useState(initialState); // 转发给当前激活的实现
}
```

### 2.5 为什么组件外调用 Hook 会报错

渲染结束后 current 被重置为 `ContextOnlyDispatcher`——**所有方法都是直接抛错的实现**。组件外调用 `useState` 取到的就是它：

```
Invalid hook call. Hooks can only be called inside of the body of a function component.
```

### 2.6 为什么这样设计

1. **API 干净**：用户永远写 `useState(x)`，不感知阶段
2. **切换点集中**：所有行为差异收敛在 renderWithHooks 一个入口
3. **可注入性**：dispatcher 是策略对象，DevTools/测试可换实现

---

## 三、useState 的更新队列

### 3.1 setCount 是什么？——一个"绑死了"的闭包

```js
// mountState 内部（简化）
const dispatch = dispatchSetState.bind(null, currentlyRenderingFiber, queue);
return [hook.memoizedState, dispatch];
```

`setCount(5)` 执行时不需要 React 猜"这 5 要送到哪"——**闭包已写死"哪个组件、哪个队列"**。

### 3.2 数据结构

```ts
interface UpdateQueue {
  pending: Update | null;      // 指向环形链表【最后一个】节点
  lastRenderedState: any;      // eager 优化用
}
interface Update {
  action: any;    // 传给 setState 的值或函数
  lane: Lane;     // 优先级（批处理关键）
  next: Update | null;
}
```

### 3.3 环形链表：pending 的妙用

- `pending` 永远指向**最新**节点
- `pending.next` 就是**最老**节点——取头取尾都是 O(1)
- 插入永远 O(1)：新节点指向最老的（`pending.next`），最新的指向新节点

```
queue.pending ──▶ u6 ──▶ u5 ──┐
                     ▲         │
                     └─────────┘
   遍历：从 pending.next（u5，最老）走到 pending（u6，最新）
```

### 3.4 调用 setCount(5) 的完整流程

```js
function dispatchSetState(fiber, queue, action) {
  // ① 创建 update 对象
  const update = { action, lane: getLane(), next: null };
  // ② 插入环形队列尾部
  if (queue.pending === null) {
    update.next = update;              // 第一个：自己指自己
  } else {
    update.next = queue.pending.next;
    queue.pending.next = update;
  }
  queue.pending = update;
  // ③ eager state 优化：队列为空时预计算，Object.is 相同则跳过调度
  // ④ scheduleUpdateOnFiber(fiber, lane) → 发起调度
}
```

**此刻没有状态计算、没有渲染**——只是"排队"。

### 3.5 render 阶段消费：updateReducer

```js
function updateReducer(hook) {
  // 从 pending.next（最老）遍历整条环：
  //   action 是函数 → state = action(state)   ← 链式计算的来源
  //   action 是值   → state = action          ← 覆盖式
  hook.memoizedState = state;   // 写回
  return [hook.memoizedState, dispatch];
}
```

| 写法 | 遍历队列时的行为 | 结果 |
|------|----------------|------|
| `setCount(c => c + 1)` 三次 | 函数式：1→2→3→4，每个函数收到上一个的返回值 | **+3** |
| `setCount(count + 1)` 三次 | 值式：三个值都是同一个快照（调用瞬间已求值），后覆盖前 | **+1** |

---

## 四、setState 完整链路（拆解 5 小步）

### 小步 1：入队（记账）

见第三章。setState 前半段 = 记账。**不计算、不渲染、不改 memoizedState**。

### 小步 2：lanes 标记与冒泡（报总部）

- `lane` = 一个二进制位，表示一种优先级：SyncLane（点击/键盘）< InputContinuousLane（拖拽/滚动）< DefaultLane（setTimeout/Promise）< TransitionLane（startTransition）
- `fiber.lanes` 用**位或**合并（同时有 SyncLane 和 DefaultLane → 两个位都亮）
- `scheduleUpdateOnFiber` 冒泡过程：

```
Button fiber:  fiber.lanes |= SyncLane
  → Panel:     fiber.lanes |= SyncLane
  → App:       fiber.lanes |= SyncLane
  → Root:      root.pendingLanes |= SyncLane
```

- **为什么冒泡到 root**：渲染从 root 开始遍历；fiber.lanes 是遍历时的"路标"（无交集整棵跳过），root.pendingLanes 是"待办看板"（决定要不要调度）
- **为什么每个祖先都要标记**：遍历从 root 往下走时，沿途靠标记判断"要不要进这棵子树"
- **root 不知道哪个 hook 有更新**：位或压扁了信息，细节存在各自的 hook.queue 里，渲染到该组件时才取用

### 小步 3：调度与去重（ensureRootIsScheduled）

```js
function ensureRootIsScheduled(root) {
  const nextLanes = getNextLanes(root);       // 从 pendingLanes 算出当前最重要的事
  const existingTask = root.callbackNode;     // root 上已排定的任务

  if (existingTask !== null) {
    if (优先级没变) {
      return;                    // ★ 批处理的核心：什么都不做
    }
    cancelCallback(existingTask);  // 优先级变了 → 取消旧的，重新排
  }

  root.callbackNode = scheduleCallback(
    priority,                                 // 由 lane 换算
    performConcurrentWorkOnRoot.bind(null, root)  // 干活的人
  );
}
```

任务对象：`{ callback: performConcurrentWorkOnRoot, priorityLevel, expirationTime }`，丢进 Scheduler 的**小顶堆**（按 expirationTime 排序），通过 **MessageChannel 发消息**触发。

**为什么用 MessageChannel 而不当场执行**：当场渲染会破坏批处理（后面的 setState 来不及合并）；宏任务边界让当前同步代码块全部执行完、所有 setState 入队后统一开工。

### 小步 4：render（可中断）—— 从 root 找到组件

1. **深度优先遍历** fiber 树，用 `fiber.lanes` 当路标
2. **bailout（快速路径）**：fiber.lanes 与本次渲染 lanes 无交集 → 整棵子树原样复用（复制到 workInProgress），**不执行组件函数、不 diff**
3. 有活的组件 → `renderWithHooks` 执行组件函数 → useState 走 update 实现 → **消费自己的 hook.queue**（update 的 lane 不在本次渲染中 → 跳过，留 baseQueue）
4. **时间切片**：每处理一个 fiber 检查帧期限，超 5ms → 暂停（保存 workInProgress 指针）→ 让出控制权，下一轮宏任务继续
5. **中断**：更高优先级更新到达 → 丢弃未完成的 workInProgress 树，从头重来

### 小步 5：commit（不可中断）

```
① workInProgress 树切换为 current 树（双缓冲）
② 更新真实 DOM
③ 执行副作用：useLayoutEffect（layout 阶段同步）/ useEffect（passive 异步）
④ 清除 root.pendingLanes 中已消费的位（markRootFinished）
```

**为什么 commit 不可中断**：DOM 已动，中途停手会导致"改了一半"的脏状态，无法恢复。

---

## 五、调度深层机制（追问沉淀）

### 5.1 标记（lanes）vs 任务（callbackNode）——两种生命周期

| | 标记（lanes） | 任务（callbackNode） |
|---|---|---|
| 是什么 | root/fiber 上的待办位集合 | Scheduler 里的一个执行载体 |
| 生命周期 | **只增不减**，直到被渲染消费 | 随时可**取消重排** |
| 产生 | 每次 setState 冒泡时位或 | ensureRootIsScheduled 排定时 |
| 消失 | render 处理完对应更新后清除 | 执行完，或被取消 |

**标记不用取消、也取消不了**（它就是待办集合表，新活只是叠加）；**可取消的只有任务**（它有且只有一个优先级，来了更急的活就得换一个更高优先级的任务顶上）。

### 5.2 取消重排为什么不会延迟渲染

- **任务还没执行**：取消一个没干过活的任务零损失；新任务优先级更高 → expirationTime 更早 → 排更前 → **比原任务更早执行**（效果 = 高优先级插队）
- **任务正在执行**：不"取消"，而是**中断当前渲染 + 丢弃未完成部分 + 从头按新优先级重来**。宁可重来也不让低优先级阻塞高优先级；且时间切片保证丢弃的工作量最多 5ms

### 5.3 为什么"删除重发"= "修改优先级"的标准实现

- 任务执行顺序完全由 expirationTime（由优先级推导）决定
- 优先级变了 → expirationTime 变了 → **小顶堆里的位置必须移动**
- 堆的"移动"没有便捷 API，标准做法：**标记删除 + 重新插入**（O(log n)）
- `cancelCallback` 是**惰性删除**：只把 task.callback 置为 null，堆里还躺着，弹出时跳过（O(1)）
- **绝大多数调用走不到这里**：同优先级直接 return（批处理主路径）

### 5.4 两层调度模型

```
浏览器事件循环：同时只有一个 React 宏任务（MessageChannel 消息）
  内容固定 = "从小顶堆里取到期任务执行"（performWorkUntilDeadline）
    ↓
React Scheduler 内部：
  小顶堆 [T2(最急) → T3 → T1]  ← 顺序完全由 React 自己管
  弹堆顶 → 执行 5ms → 到期 → 再发 MessageChannel 消息让出
```

**一个例外**：SyncLane（点击/键盘等离散交互）**不经过小顶堆**——排进同步回调队列，用**微任务**立即 flush（保证绘制前完成）。DefaultLane / TransitionLane 才走 Scheduler 的堆。

### 5.5 全链路时间线对比

**场景 A：点击 onClick 里 setCount(5)（SyncLane）**

```
[宏任务] click 事件 → onClick → setCount(5)
  ① update 入队 ② 冒泡标记 ③ ensureRootIsScheduled: SyncLane → 同步回调队列 + 微任务 flush
[微任务] flushSyncCallbacks → renderRootSync（不切片）→ commit → DOM 更新
[vsync] 渲染机会 → paint（点击到可见，同一帧内完成）
```

**场景 B：setTimeout 里 setCount(6)（DefaultLane）**

```
[宏任务] setTimeout 回调 → setCount(6) → 入队/冒泡/排任务进堆 → MessageChannel 发消息
[宏任务] MessageChannel 消息 → 弹堆顶 → workLoopConcurrent（可切片可中断）→ commit
```

---

## 六、React 18 自动批处理精确定位

### 6.1 常见误解纠正

**❌ "React 18 用微任务实现批处理"**
批处理 = **调度层的任务去重**（ensureRootIsScheduled：已有任务同优先级 → 直接 return），与调用位置无关。微任务只出现在 SyncLane 的**执行**环节（flush），那是执行层面的事，不是批处理层面的机制。

**❌ "React 18 之前 setState 同步，18 之后异步"**
批处理边界由**调用场景决定的 lane** 决定：

| 场景 | React 17 | React 18 |
|------|---------|---------|
| React 事件处理器内 | 批处理 | 批处理 |
| setTimeout / Promise / 原生事件 | **不批处理**（同步多次渲染） | 批处理 |

**❌ "常规 setState 都是紧急同步微任务"**
只有**离散交互事件**（click/keydown）→ SyncLane → 微任务 flush；setTimeout/Promise 里的 setState → **DefaultLane** → 走 Scheduler 并发调度（宏任务）。"紧急与否由触发事件的类型决定，不由 setState 本身决定"。

### 6.2 时间切片由 lane 决定，不由 API 决定

| 场景 | lane | 渲染入口 | 时间切片 |
|------|------|---------|---------|
| 点击/键盘里的 setState | SyncLane | renderRootSync（微任务 flush） | ❌ |
| setTimeout/Promise 里的 setState | DefaultLane | renderRootConcurrent | ✅ 可切片 |
| startTransition / useDeferredValue | TransitionLane | renderRootConcurrent | ✅ 可切片 |

时间切片是"能力"不是"保证"：渲染 5ms 内能做完就一口气跑完，观察不到切片；只有超过帧预算才表现。

### 6.3 setState vs startTransition 的真正区别（不是"是否切片"）

| | DefaultLane（普通 setState） | TransitionLane（startTransition） |
|---|---|---|
| 优先级 | 中 | **最低** |
| 被插队 | 会被 SyncLane 插队重来 | 同样会被，且更频繁 |
| 可被无限推迟 | 有过期时间兜底 | **无硬性过期**（用户一直点就一直等） |
| 配合 Suspense | 新内容未就绪会阻塞 | 保留旧 UI（pending 状态） |

### 6.4 并发重放（baseState / baseQueue）

- 队列里低优先级 update（如 DefaultLane）被高优先级渲染**跳过**时：**不丢失**，留在 `baseQueue`
- 下次渲染**从 baseState（跳过前的状态）重放**，保证所有更新最终按序生效
- **函数式更新在重放下依然正确**：action 的执行推迟到遍历那一刻，重放时重新执行
- 值式更新的数值在创建时已固定

---

## 七、useEffect 全流程

### 7.1 两段式设计

**调用 useEffect 的那一刻，create 不会执行。** 调用只是"登记"：

```js
function mountEffect(create, deps) {
  // ① 创建 effect 对象（存 create、deps 引用快照）
  // ② 挂到自己的 hook 节点（hook.memoizedState = effect）
  // ③ 追加到 fiber.updateQueue 的 effect 链表（环形，lastEffect 指向尾部）
  // ④ 给 fiber.flags 打上 PassiveEffect 标记
}
```

```ts
interface Effect {
  tag: number;         // Passive(useEffect) / Layout(useLayoutEffect)
  create: () => void;  // 副作用函数（此刻只是被存着）
  destroy: () => void; // 清理函数（首次渲染时 undefined）
  deps: Array | null;  // 依赖数组快照
  next: Effect | null;
}
```

### 7.2 关键：deps 只影响"是否执行"，不影响"是否登记"

**每次渲染都会创建全新的 effect 对象并追加到链表，不管 deps 变没变。** 原因：

1. **快照机制**：判断"变没变"需要本次 deps vs 上次 deps，上次的存在旧 effect 对象里；不创建新对象，比较链条就断了
2. **链表完整性**：链表也是 destroy（清理函数）的载体，destroy/create 严格配对，顺序不能乱；用"标记"表示是否执行，而不是"不在册"
3. **两段式一致性**：render 可中断可丢弃，副作用不能在这里执行，只能"登记"；执行决策统一在 commit

**实现细节**：deps 比较发生在 **render 阶段**（updateEffect 内部），结果编码成 `HookHasEffect` 标记打在 effect 对象上；commit 遍历链表**只看标记**，有标记的执行，没标记的跳过。

### 7.3 术语澄清：React 渲染 ≠ 浏览器渲染

| | React 的 render 阶段 | 浏览器的渲染 |
|---|---|---|
| 本质 | 纯 JS 代码执行（协调阶段） | style/layout/paint（浏览器引擎） |
| 发生位置 | 主线程，React 排定的任务里 | 主线程，vsync 驱动的渲染机会 |
| 谁执行 | JavaScript 引擎 | 浏览器渲染引擎 |

"渲染期间执行 JS"不矛盾——**React 的 render 阶段本身就是一段 JS**。浏览器接管的是最后的 paint。

### 7.4 commit 的三个子阶段

```
commit（不可中断，全部同步 JS）
├─ ① before mutation: 快照、安排 passive 的清理
├─ ② mutation: 更新真实 DOM
│     └─ 同时执行【上一次】useLayoutEffect 的 destroy
├─ ③ layout: 绑定 ref、执行【本次】useLayoutEffect 的 create
└─ （结束后）→ 排宏任务 → paint 之后 → useEffect 的 destroy + create
```

### 7.5 useLayoutEffect vs useEffect

| | useLayoutEffect | useEffect |
|---|---|---|
| deps 比较 | render 阶段（打标记） | render 阶段（打标记） |
| destroy | commit ② mutation（同步） | paint 后 passive（异步） |
| create | commit ③ layout（同步，paint 前） | paint 后 passive（异步） |
| 发布方式 | **不发布，在 commit 调用栈里直接执行** | Scheduler 宏任务（MessageChannel） |
| 能否阻塞绘制 | 能（故意为之） | 不能 |

### 7.6 useEffect 怎么保证在 paint 之后执行

```js
// commitRoot 末尾（简化）
if (有 passive effects 待执行) {
  scheduleCallback(NormalSchedulerPriority, flushPassiveEffects);
}
```

**通过 Scheduler 的宏任务（MessageChannel）发布**，机制与渲染任务相同。严格说：宏任务只能保证"当前任务之后"，不能从规范层面绝对保证"一定在 paint 后"（vsync 存在竞速）；但：

- React 文档对 useEffect 的语义定位就是"浏览器绘制后运行的副作用"
- 实践上 commit 任务与 passive 任务之间隔了一个宏任务边界，60Hz 下几乎总能在中间完成绘制
- **React 不依赖这个精确时序**（非阻塞副作用）；需要精确时序的场景用 useLayoutEffect（规范级保证）

---

## 八、useMemo / useCallback / useRef

### 8.1 useRef：最特殊——连"更新"都没有

```js
function mountRef(initialValue) {
  const hook = mountWorkInProgressHook();
  const ref = { current: initialValue };
  hook.memoizedState = ref;
  return ref;
}
function updateRef() {
  const hook = updateWorkInProgressHook();
  return hook.memoizedState;   // 同一个对象，从 mount 起再没换过
}
```

**`ref.current = x` 只是普通属性赋值：没有入队、没有冒泡、没有调度 → 永不触发渲染。** 反向验证全链路模型：触发渲染的唯一入口是"更新入队 + 冒泡标记"，ref 不走这条路。

### 8.2 useMemo：存 [计算结果, deps]

- mount：**立即执行** nextCreate，存 `[值, deps]`
- update：deps 逐个 Object.is 相同 → 返回缓存值（跳过计算）；不同 → 重新计算
- **计算在 render 阶段同步执行 → useMemo 里不能写副作用**

### 8.3 useCallback：useMemo 的语法糖

```js
useCallback(fn, deps)  ≡  useMemo(() => fn, deps)
```

缓存函数引用，配合 React.memo 浅比较做渲染优化。

### 8.4 三者共同点

| | 存储位置 | deps 比较 | 是否触发渲染 |
|---|---|---|---|
| useRef | hook 节点的 {current} 对象 | 无 | 永不 |
| useMemo | [值, deps] 二元组 | Object.is 浅比较 | 不 |
| useCallback | [函数, deps] 二元组 | Object.is 浅比较 | 不 |

**全部复用同一套机制：hook 链表节点 + memoizedState 存储 + Object.is 浅比较。** 没有特殊实现。

---

## 九、完整模型 + 高频面试题

```
┌─ 组件渲染时 ─────────────────────────────────────────────┐
│ fiber.memoizedState: hook0 → hook1 → hook2（链表）        │
│ 每个 hook 节点: { memoizedState, queue, next }            │
└─────────────────────────────────────────────────────────┘
┌─ setState 时 ───────────────────────────────────────────┐
│ update 入队（记账）→ lanes 冒泡（报总部）→ 调度去重         │
│ → render（bailout 跳过 → 消费队列 → 生成新树）            │
│ → commit（DOM + layout 同步 + passive 异步）              │
└─────────────────────────────────────────────────────────┘
```

**现在能完整回答的高频面试题：**

1. 为什么 hooks 不能条件调用？→ 链表按位置匹配，顺序变化导致错位/报错
2. hooks 状态存在哪？→ fiber.memoizedState 的单向链表
3. 为什么自定义 hook 状态隔离？→ 每个组件实例有独立 fiber
4. setState 为什么"异步"？→ 入队 + 调度合并，渲染在任务边界执行
5. 为什么多次 setState 只渲染一次？→ ensureRootIsScheduled 任务去重
6. 函数式更新 vs 值式更新？→ 执行推迟到遍历时 vs 创建时固定
7. useEffect 为什么异步？→ passive 阶段通过 Scheduler 宏任务发布
8. useLayoutEffect 为什么同步？→ 必须在 paint 前完成，夹在 commit 里
9. 修改 ref.current 为什么不渲染？→ 没有入队、没有标记、没有调度
10. 为什么 React 18 所有场景都批处理？→ 调度层统一去重，不再依赖调用位置

---

## 十、误区纠正记录（从错误到正确）

| # | 错误理解 | 正确模型 |
|---|---------|---------|
| 1 | rAF 在 DOM 更新后执行 | rAF 在渲染步骤最前（style/layout/paint 之前） |
| 2 | setState：18 之前同步、18 之后异步 | 批处理边界由调用场景决定的 lane 决定 |
| 3 | React 18 用微任务实现批处理 | 批处理 = 调度任务去重；微任务只用于 SyncLane flush |
| 4 | 常规 setState 都是紧急同步 | 只有离散交互 SyncLane；setTimeout/Promise 是 DefaultLane |
| 5 | setState 不开启时间切片，只有 transition 会 | DefaultLane 也可切片；"是否切片"由 lane 决定 |
| 6 | 标记可以被取消 | lanes 只增不减直到消费；可取消的只有任务 |
| 7 | 取消重排会延迟渲染 | 未执行 → 插队提前；执行中 → 中断重来（最多丢 5ms） |
| 8 | React 渲染 = 浏览器渲染 | React render 是 JS 执行；浏览器接管的是 paint |
| 9 | deps 没变就不需要新 effect 对象 | 每次渲染都新建（快照 + 链表完整性 + 标记机制） |
| 10 | useEffect 严格保证在 paint 后 | 宏任务发布 + 语义定位非阻塞；精确时序用 useLayoutEffect |
| 11 | useMemo/useCallback 有特殊实现 | 同一套链表机制 + Object.is 浅比较 |

---

## 十一、学习方法总结

本次采用"拆小步 + 即时确认"的教学模式：
- 先建全景模型（一句话模型），再逐层深入
- 每讲一小部分配 1-2 个确认问题，答错当场补讲
- 追问内容（标记 vs 任务、取消重排、两层调度等）全部沉淀进笔记
- 误区纠正记录表可作复习时的对照清单
