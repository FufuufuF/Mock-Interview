# 原型与继承 — 面试要点笔记

> 学习日期：2026-05-21 | 分类：js-basics

## 一句话总结

原型链通过 __proto__ 逐层向上查找属性（终点 null），继承的最优解是寄生组合式（call 继承实例属性 + Object.create 连接原型），ES6 class 是其语法糖。

## 知识要点

### 1. prototype 与 __proto__ 的关系

**面试问法**: 请解释 prototype、__proto__ 和 constructor 三者的关系。

**核心回答**:
- 函数有 prototype 属性（对象没有），用于给实例提供原型
- 实例有 __proto__ 属性，指向构造函数的 prototype
- prototype 对象上有 constructor 属性，指回构造函数本身
- 三角关系：`f.__proto__ === Foo.prototype`，`Foo.prototype.constructor === Foo`

**加分细节**:
- __proto__ 不是标准属性，是浏览器实现的访问器，规范中叫 [[Prototype]]
- 推荐用 Object.getPrototypeOf() 读取原型

### 2. 原型链查找机制

**面试问法**: 原型链是怎么工作的？查找的终点是什么？

**核心回答**:
- 属性查找沿 __proto__ 链逐层向上：实例 → 构造函数.prototype → Object.prototype → null
- 找到就返回，找到 null 还没有就返回 undefined
- 和作用域链类似：作用域链找变量，原型链找属性

**加分细节**:
- Object.prototype 上有 toString/valueOf/hasOwnProperty 等"继承方法"
- hasOwnProperty 可区分自身属性和原型属性
- 各内置类型会覆写 toString（Array 返回逗号分隔字符串，Number 返回数字字符串）

### 3. 继承方式演进

**面试问法**: JavaScript 有哪些实现继承的方式？请手写寄生组合式继承。

**核心回答**:

继承需要做两件事：继承实例属性 + 继承原型方法。

1. **原型链继承**：`Child.prototype = new Parent()`
   - 问题：引用类型属性被所有子实例共享污染
2. **借用构造函数**：`Parent.call(this)`
   - 问题：拿不到父类原型上的方法
3. **组合继承**：call + new Parent()
   - 问题：Parent 被调用两次，原型上有多余属性
4. **寄生组合式继承**（最优解）：call + Object.create(Parent.prototype)
   - Parent 只调用一次，原型链干净

```js
function Child(name, age) {
  Parent.call(this, name);
  this.age = age;
}
Child.prototype = Object.create(Parent.prototype);
Child.prototype.constructor = Child;
```

**加分细节**:
- 演进逻辑：共享问题 → call 解决 → 丢了原型 → 结合 → 多调一次 → Object.create 替代 new
- Object.create(Parent.prototype) vs new Parent()：前者只建立链接不执行构造函数

### 4. ES6 class 的本质

**面试问法**: ES6 class 本质上是什么？和构造函数有什么关系？

**核心回答**:
- class 是语法糖，底层就是寄生组合式继承
- extends 对应 Object.create 连接原型，super() 对应 Parent.call(this)
- typeof class === 'function'

与构造函数的差异：
- 不能不用 new 直接调用
- 原型方法不可枚举
- 自动严格模式
- 不提升（类似 let 的 TDZ）
- 子类 constructor 必须先调 super() 才能用 this

**加分细节**:
- super 有两个身份：作为函数调用父构造函数，作为对象访问父原型方法
- 子类不创建 this，this 由父类构造函数创建后传下来
- 静态方法也会被继承：Child.__proto__ === Parent

### 5. Object.create 与原型操作

**面试问法**: Object.create(null) 和 {} 有什么区别？

**核心回答**:
- `{}` 的 __proto__ 指向 Object.prototype，继承了 toString 等方法
- `Object.create(null)` 创建无原型的纯净对象，没有任何继承方法
- 用途：作为纯字典使用，避免原型属性干扰（如 hasOwnProperty 被覆盖）

手写 Object.create：
```js
function myCreate(proto) {
  function F() {}
  F.prototype = proto;
  return new F();
}
```

**加分细节**:
- Vue2 源码大量使用 Object.create(null) 创建存储对象
- Object.create 第二个参数可传属性描述符

## 常见追问及回答

| 追问 | 回答要点 |
|------|---------|
| instanceof 的原理？ | 沿着实例的 __proto__ 链查找是否有某个构造函数的 prototype |
| 如何实现多继承？ | Mixin 模式：Object.assign(Child.prototype, mixin1, mixin2) |
| Object.getPrototypeOf 和 __proto__ 的区别？ | 前者是标准 API，后者是非标准访问器，推荐用前者 |
| 为什么寄生组合式最优？ | Parent 只调一次 + 原型无多余属性 + 实例属性独立 |
| class 中为什么必须先 super()？ | 子类不创建 this，this 由父类创建后返回 |

## 易错点

- 误区"每个对象都有 prototype"：只有函数有 prototype，普通对象只有 __proto__
- 误区"组合继承没问题"：Parent 被调用两次是面试考点
- 误区"class 是全新机制"：底层还是 prototype + __proto__，机制没变
- 误区"Object.create(null) 和 {} 差不多"：前者连 toString 都没有，是纯字典
- 误区"原型链终点是 Object.prototype"：终点是 null（Object.prototype.__proto__ === null）
