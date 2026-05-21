# this 指向 — 面试要点笔记

> 学习日期：2026-05-20 | 分类：js-basics

## 一句话总结

this 由调用方式决定（非箭头函数），四种规则优先级：new > 显式绑定 > 隐式绑定 > 默认绑定；箭头函数无自己的 this，继承外层词法作用域。

## 知识要点

### 1. 四大绑定规则与优先级

**面试问法**: 请详细解释 this 的指向规则，并按优先级排列。

**核心回答**:
- 默认绑定：`fn()` 独立调用，this 为 window（严格模式 undefined）
- 隐式绑定：`obj.fn()` 方法调用，this 为点号前的对象
- 显式绑定：`call/apply/bind` 指定 this
- new 绑定：`new Fn()` 创建新对象作为 this
- 优先级：new > 显式 > 隐式 > 默认

**加分细节**:
- new 可以覆盖 bind：`new (fn.bind(obj))()` 中 this 指向新对象而非 obj
- 箭头函数独立于四大规则之外，无法被任何方式改变 this

### 2. 箭头函数的 this

**面试问法**: 箭头函数和普通函数的 this 有什么区别？

**核心回答**:
- 箭头函数没有自己的 this，继承外层第一个普通函数的 this
- call/apply/bind/new 对箭头函数均无效
- 箭头函数还缺少：arguments、prototype、不能做构造函数、不能做 Generator

**加分细节**:
- 对象字面量 `{}` 不创建作用域，箭头函数作为对象属性时 this 不是该对象
- class 属性中的箭头函数有效，因为属性初始化器在 constructor 内执行，捕获到实例 this

### 3. 隐式绑定丢失

**面试问法**: 什么是隐式绑定丢失？在什么场景下会发生？

**核心回答**:
- 函数引用脱离对象后独立调用，this 退回默认绑定
- 三种典型场景：赋值给变量、作为回调传递、间接调用
- 本质：调用时点号前面没有对象
- 修复方式：bind、箭头函数包裹、class 箭头属性

**加分细节**:
- React class 组件中事件处理器的 this 丢失是经典应用场景
- addEventListener 传入方法引用也会丢失

### 4. call/apply/bind 的区别与手写

**面试问法**: call、apply、bind 三者有什么区别？请手写实现一个。

**核心回答**:
- call：逐个传参，立即执行
- apply：数组传参，立即执行
- bind：逐个传参（支持偏函数），返回新函数，不立即执行
- 手写 call 核心思路：将函数临时挂到 context 上调用（利用隐式绑定），用 Symbol 避免属性冲突

**加分细节**:
- bind 支持偏函数：`add.bind(null, 5)` 预设第一个参数
- context 为 null/undefined 时兜底到 globalThis

### 5. new 操作符的过程

**面试问法**: new 操作符具体做了什么？

**核心回答**:
- 四步：创建空对象 → 原型指向 Fn.prototype → 绑定 this 执行构造函数 → 判断返回值
- return 对象/函数 → 替代 new 创建的对象
- return 原始值/无 return → 返回 new 创建的对象

**加分细节**:
- 能手写 myNew 实现
- new.target 可检测是否通过 new 调用
- 这也解释了 new 优先级高于 bind

## 常见追问及回答

| 追问 | 回答要点 |
|------|---------|
| new 能覆盖 bind 吗？ | 能，new 内部重新绑定 this 到新对象 |
| 箭头函数能 new 吗？ | 不能，报 TypeError |
| bind 返回的函数有 prototype 吗？ | 没有（ES6 规范） |
| 如何判断函数是否通过 new 调用？ | 使用 new.target，有值说明是 new 调用 |
| 隐式丢失怎么修？ | bind / 箭头函数包裹 / class 箭头属性 |

## 易错点

- 误区"bind 优先级最高"：new 可以覆盖 bind
- 误区"箭头函数 this 指向定义时所在的对象"：指向外层第一个普通函数的 this，对象字面量不算
- 误区"方法定义在对象上就不会丢 this"：调用方式才决定，赋值/传参都会丢
- 误区"return 函数会被 new 忽略"：函数也是对象，会替代 new 创建的对象
