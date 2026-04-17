---
name: comment
description: 面试中记录知识盲区。在回答面试官问题时，面试者可以用 /comment 记录自己发现的知识盲区，这些记录不会影响面试流程，会在复盘阶段统一解决。用法：/comment 我不太确定 Promise.allSettled 和 Promise.all 的区别
argument-hint: "[你发现的知识盲区描述]"
disable-model-invocation: true
allowed-tools: Write Read Glob Bash(date *)
---

# 记录知识盲区

面试者在回答问题的过程中发现了自己的知识盲区，需要记录下来。

## 操作步骤

1. 使用 `date +%Y-%m-%d` 获取今天的日期
2. 确定 comments 文件路径：`records/comments/YYYY-MM-DD-comments.md`
3. 检查该文件是否已存在：
   - **已存在**：读取文件内容，在末尾追加新条目
   - **不存在**：创建新文件，写入标题和第一条记录
4. 将面试者的 comment 追加到文件中，格式如下：

```markdown
- [ ] $ARGUMENTS
```

5. 追加完成后，**只用一句话简短确认**，如"已记录。"，然后**立即停止输出**。不要做任何解释、分析或展开讨论。这条 comment 不属于面试对话的一部分，不要干扰面试节奏。

## comments 文件完整格式

```markdown
# 面试知识盲区记录 — YYYY-MM-DD

- [ ] 第一条记录内容
- [ ] 第二条记录内容
- [ ] ...
```

## 重要规则

- **极简输出**：只回复"已记录。"三个字，不要有任何额外内容
- **不要解答**：不要对 comment 中的问题做任何解释或回答
- **不要影响面试**：这个 skill 的唯一作用就是把文字写入文件
