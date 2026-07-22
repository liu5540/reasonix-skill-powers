# Skill 设计中的说服原则

## 概述

大语言模型（LLM）与人类一样会受到说服原则的影响。理解这种心理学有助于你设计更有效的 Skill——不是为了操控，而是为了确保关键实践即使在压力下也能被遵循。

**研究基础：** Meincke 等人（2025）以 N=28,000 次 AI 对话测试了 7 种说服原则。说服技巧使遵从率提升了一倍以上（33% → 72%，p < .001）。

## 七项原则

### 1. 权威（Authority）
**是什么：** 对专业知识、资历或官方来源的遵从。

**在 Skill 中如何起作用：**
- 命令式语言："YOU MUST"、"Never"、"Always"
- 不容协商的框架："No exceptions"
- 消除决策疲劳和自我合理化

**何时使用：**
- 纪律约束型 Skill（TDD、验证要求）
- 安全关键实践
- 已确立的最佳实践

**示例：**
```markdown
✅ Write code before test? Delete it. Start over. No exceptions.
❌ Consider writing tests first when feasible.
```

### 2. 承诺（Commitment）
**是什么：** 与先前的行为、言论或公开声明保持一致。

**在 Skill 中如何起作用：**
- 要求声明："Announce skill usage"
- 强制明确选择："Choose A, B, or C"
- 使用追踪：待办清单用于检查项

**何时使用：**
- 确保 Skill 真正被遵循
- 多步骤流程
- 问责机制

**示例：**
```markdown
✅ When you find a skill, you MUST announce: "I'm using [Skill Name]"
❌ Consider letting your partner know which skill you're using.
```

### 3. 稀缺（Scarcity）
**是什么：** 来自时间限制或有限可用性的紧迫感。

**在 Skill 中如何起作用：**
- 有时间限制的要求："Before proceeding"
- 顺序依赖："Immediately after X"
- 防止拖延

**何时使用：**
- 即时验证要求
- 时间敏感的工作流
- 防止"我稍后再做"

**示例：**
```markdown
✅ After completing a task, IMMEDIATELY request code review before proceeding.
❌ You can review code when convenient.
```

### 4. 社会认同（Social Proof）
**是什么：** 遵从他人做法或被认为是常态的行为。

**在 Skill 中如何起作用：**
- 通用模式："Every time"、"Always"
- 失败模式："X without Y = failure"
- 建立规范

**何时使用：**
- 记录通用实践
- 警示常见失败
- 强化标准

**示例：**
```markdown
✅ Checklists without todo tracking = steps get skipped. Every time.
❌ Some people find a todo list helpful for checklists.
```

### 5. 归属感（Unity）
**是什么：** 共享身份、"我们感"、群体归属。

**在 Skill 中如何起作用：**
- 协作式语言："our codebase"、"we're colleagues"
- 共同目标："we both want quality"

**何时使用：**
- 协作工作流
- 建立团队文化
- 非层级化实践

**示例：**
```markdown
✅ We're colleagues working together. I need your honest technical judgment.
❌ You should probably tell me if I'm wrong.
```

### 6. 互惠（Reciprocity）
**是什么：** 对获得的好处产生回报的义务。

**如何起作用：**
- 谨慎使用——可能让人感觉被操控
- 在 Skill 中很少需要

**何时避免：**
- 几乎总是如此（其他原则更有效）

### 7. 好感（Liking）
**是什么：** 更愿意与我们喜欢的人合作。

**如何起作用：**
- **不要用于获取遵从**
- 与诚实的反馈文化冲突
- 会产生谄媚行为

**何时避免：**
- 在纪律执行中始终避免

## 按 Skill 类型组合原则

| Skill 类型 | 使用 | 避免 |
|------------|-----|-------|
| 纪律约束型 | Authority + Commitment + Social Proof | Liking, Reciprocity |
| 指导/技巧型 | Moderate Authority + Unity | Heavy authority |
| 协作型 | Unity + Commitment | Authority, Liking |
| 参考型 | 仅清晰表达 | 所有说服技巧 |

## 为什么有效：心理学解释

**明确界限规则减少自我合理化：**
- "YOU MUST" 消除决策疲劳
- 绝对性语言消除"这是否是例外？"的问题
- 明确的反合理化表述堵住具体漏洞

**执行意图创造自动化行为：**
- 清晰触发条件 + 必要行动 = 自动执行
- "When X, do Y" 比 "generally do Y" 更有效
- 降低遵从的认知负担

**LLM 是类人的（parahuman）：**
- 训练数据包含人类文本中的这些模式
- 权威语言在训练数据中先于遵从行为出现
- 承诺序列（声明 → 行动）经常被建模
- 社会认同模式（每个人都做 X）建立规范

## 伦理使用

**合法用途：**
- 确保关键实践被遵循
- 创建有效的文档
- 防止可预见的失败

**不合法用途：**
- 为个人利益操控他人
- 制造虚假紧迫感
- 基于内疚的遵从

**检验标准：** 如果用户完全理解这种技巧，它是否仍能服务于用户的真正利益？

## 研究引用

**Cialdini, R. B. (2021).** *Influence: The Psychology of Persuasion (New and Expanded).* Harper Business.
- 说服的七项原则
- 影响力研究的实证基础

**Meincke, L., Shapiro, D., Duckworth, A. L., Mollick, E., Mollick, L., & Cialdini, R. (2025).** Call Me A Jerk: Persuading AI to Comply with Objectionable Requests. University of Pennsylvania.
- 以 N=28,000 次 LLM 对话测试了 7 项原则
- 使用说服技巧后遵从率从 33% 提升至 72%
- Authority、commitment、scarcity 最有效
- 验证了 LLM 行为的类人模型

## 快速参考

设计 Skill 时，问自己：

1. **它是什么类型？**（纪律约束型 vs. 指导型 vs. 参考型）
2. **我想改变什么行为？**
3. **哪些原则适用？**（纪律约束型通常是 authority + commitment）
4. **是否组合了太多原则？**（不要全部七种都用）
5. **这是否符合伦理？**（是否服务于用户的真正利益？）
