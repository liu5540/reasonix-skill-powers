---
name: requesting-code-review
description: 在完成任务、实现主要功能或在合并前验证工作是否符合要求时使用
---

# 请求代码审查

派遣一名代码审查子代理，在问题蔓延之前将其捕获。审查者会获得精确构建的评估上下文——永远不会是你的会话历史。这让审查者专注于工作成果，而非你的思考过程，同时为你保留上下文以便继续工作。

**核心原则：** 尽早审查，频繁审查。

## 何时请求审查

**强制要求：**
- 在子代理驱动开发中的每个任务之后
- 在完成主要功能之后
- 在合并到 main 分支之前

**可选但很有价值：**
- 卡住时（获得新视角）
- 重构前（基线检查）
- 修复复杂 bug 后

## 如何请求

**1. 获取 git SHA：**
```bash
BASE_SHA=$(git rev-parse HEAD~1)  # or origin/main
HEAD_SHA=$(git rev-parse HEAD)
```

**2. 派遣代码审查子代理：**

派遣一个 `general-purpose` 子代理，填写 [code-reviewer.md](code-reviewer.md) 中的模板

**占位符：**
- `{DESCRIPTION}` - 你所构建内容的简要摘要
- `{PLAN_OR_REQUIREMENTS}` - 它应该做什么
- `{BASE_SHA}` - 起始提交
- `{HEAD_SHA}` - 结束提交

**3. 根据反馈采取行动：**
- 立即修复 Critical 问题
- 在继续前修复 Important 问题
- 将 Minor 问题记录到后续处理
- 如果审查者错了，提出异议（附上理由）

## 示例

```
[Just completed Task 2: Add verification function]

You: Let me request code review before proceeding.

BASE_SHA=$(git log --oneline | grep "Task 1" | head -1 | awk '{print $1}')
HEAD_SHA=$(git rev-parse HEAD)

[Dispatch code reviewer subagent]
  DESCRIPTION: Added verifyIndex() and repairIndex() with 4 issue types
  PLAN_OR_REQUIREMENTS: Task 2 from docs/superpowers/plans/deployment-plan.md
  BASE_SHA: a7981ec
  HEAD_SHA: 3df7661

[Subagent returns]:
  Strengths: Clean architecture, real tests
  Issues:
    Important: Missing progress indicators
    Minor: Magic number (100) for reporting interval
  Assessment: Ready to proceed

You: [Fix progress indicators]
[Continue to Task 3]
```

## 与工作流集成

**子代理驱动开发：**
- 每个任务后都进行审查
- 在问题累积前捕获它们
- 在进入下一个任务前修复

**执行计划：**
- 每个任务后或在自然检查点进行审查
- 获取反馈，应用，继续

**临时开发：**
- 合并前审查
- 卡住时审查

## 危险信号

**绝不要：**
- 因为“很简单”而跳过审查
- 忽略 Critical 问题
- 带着未修复的 Important 问题继续
- 对有效的技术反馈进行争辩

**如果审查者错了：**
- 用技术理由提出异议
- 展示证明其有效的代码/测试
- 请求澄清

查看模板：[code-reviewer.md](code-reviewer.md)
