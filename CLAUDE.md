# Mock-Interview — AI 模拟面试训练系统

## 项目说明

本仓库是一个 AI 模拟面试训练系统，用于帮助用户通过 Claude Code 的 skill 机制进行模拟面试练习和知识点深度学习。包含面试模式（AI 面试官选题提问）和学习模式（从面试八股文角度深入学习指定主题）。

## 仓库结构

```
Mock-Interview/
├── CLAUDE.md                  # 本文件
├── .claude/skills/            # Claude Code Skills
│   ├── interview/             # /interview — 启动模拟面试
│   ├── study/                 # /study — 面试八股文深度学习
│   ├── comment/               # /comment — 面试中标记知识盲区
│   ├── review/                # /review — 面试复盘分析
│   └── progress/              # /progress — 进度统计
├── questions/                 # 题库（按岗位/方向组织）
│   └── frontend/              # 前端方向题库
├── records/                   # 面试记录 & 学习记录
│   ├── markdown/              # 面试记录 Markdown 格式
│   ├── json/                  # 面试记录 JSON 格式
│   ├── comments/              # 面试中标记的知识盲区记录
│   └── study/                 # 学习记录
│       ├── markdown/          # 学习笔记（可直接复习）
│       └── json/              # 结构化数据（供其他 skill 消费）
└── templates/                 # 记录模板
```

## 面试记录命名规范

文件名格式：`YYYY-MM-DD-{position}.md` / `YYYY-MM-DD-{position}.json`

示例：
- `2026-04-15-frontend.md`
- `2026-04-15-frontend.json`

同一天有多次面试时追加序号：`2026-04-15-frontend-2.md`

## 评分标准

每道题按 A/B/C/D 四档评分：

| 等级 | 含义 | 标准 |
|------|------|------|
| A | 优秀 | 回答准确、完整，能深入阐述原理，有自己的思考和实际经验 |
| B | 良好 | 回答基本正确，覆盖主要知识点，但深度或广度有所欠缺 |
| C | 一般 | 回答部分正确，存在明显知识盲区或理解偏差 |
| D | 不足 | 回答错误、不完整，或无法回答 |

综合评分规则：
- A 占比 >= 70%：综合 A
- A+B 占比 >= 70%：综合 B
- D 占比 >= 50%：综合 D
- 其余情况：综合 C

## 题库方向（前端）

| 文件 | 方向 | 说明 |
|------|------|------|
| `js-basics.md` | JavaScript 基础 | 变量、作用域、闭包、原型链、异步等 |
| `css-layout.md` | CSS 布局 | 盒模型、Flex、Grid、BFC、响应式等 |
| `network.md` | 计算机网络 | HTTP/HTTPS、TCP、DNS、CDN、缓存等 |
| `browser.md` | 浏览器渲染机制 | 渲染流程、重排重绘、事件循环、V8 等 |
| `react.md` | React | Hooks、虚拟 DOM、Fiber、状态管理等 |
| `vue.md` | Vue | 响应式原理、虚拟 DOM、生命周期、Pinia 等 |
| `scenario.md` | 场景题/手写题 | 手写 Promise、防抖节流、深拷贝等 |
| `performance.md` | 性能优化 | 加载优化、渲染优化、打包优化等 |

## 学习记录命名规范

文件名格式：`YYYY-MM-DD-{topic-slug}.md` / `YYYY-MM-DD-{topic-slug}.json`

topic-slug 规则：主题名转为小写英文短横线格式。示例：
- "Event Loop" → `2026-05-19-event-loop.md`
- "闭包" → `2026-05-19-closure.md`
- "虚拟DOM" → `2026-05-19-virtual-dom.md`

同一天同主题多次学习时追加序号：`2026-05-19-event-loop-2.md`

## 学习记录数据消费

学习记录的 JSON 数据会被其他 skill 消费：

| 消费者 | 用途 |
|--------|------|
| `/interview` | 从已学习的知识点中选题验证掌握情况 |
| `/review` | 对比面试表现 vs 学习记录，定位"学了但没掌握"的知识点 |
| `/progress` | 展示知识覆盖率和学习进度 |
