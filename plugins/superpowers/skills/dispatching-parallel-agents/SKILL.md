---
name: dispatching-parallel-agents
description: 当面临两个及以上相互独立、无需共享状态或顺序依赖的任务时使用
---

# 派发并行代理

## 概述

你将任务委派给具有隔离上下文的专业代理。通过精确设计它们的指令和上下文，你确保它们保持专注并完成任务。它们永远不应该继承你会话的上下文或历史记录——你只构造它们真正需要的内容。这也为你保留了协调工作所需的上下文。

当你遇到多个互不相关的失败（不同的测试文件、不同的子系统、不同的 bug）时，按顺序调查会浪费时间。每次调查都是独立的，可以并行进行。

**核心原则：** 每个独立的问题域派发一个代理。让它们并发工作。

## 何时使用

```dot
digraph when_to_use {
    "Multiple failures?" [shape=diamond];
    "Are they independent?" [shape=diamond];
    "Single agent investigates all" [shape=box];
    "One agent per problem domain" [shape=box];
    "Can they work in parallel?" [shape=diamond];
    "Sequential agents" [shape=box];
    "Parallel dispatch" [shape=box];

    "Multiple failures?" -> "Are they independent?" [label="yes"];
    "Are they independent?" -> "Single agent investigates all" [label="no - related"];
    "Are they independent?" -> "Can they work in parallel?" [label="yes"];
    "Can they work in parallel?" -> "Parallel dispatch" [label="yes"];
    "Can they work in parallel?" -> "Sequential agents" [label="no - shared state"];
}
```

**适用场景：**
- 3 个以上测试文件因不同根因失败
- 多个子系统独立损坏
- 每个问题都可以在不依赖其他问题上下文的情况下理解
- 调查之间没有共享状态

**不适用场景：**
- 失败彼此相关（修复一个可能会修复其他）
- 需要理解完整系统状态
- 代理之间会相互干扰

## 模式

### 1. 识别独立域

按损坏的内容对失败进行分组：
- 文件 A 测试：工具审批流程
- 文件 B 测试：批处理完成行为
- 文件 C 测试：中止功能

每个域都是独立的——修复工具审批不会影响中止测试。

### 2. 创建聚焦的代理任务

每个代理获得：
- **明确的范围：** 一个测试文件或子系统
- **清晰的目标：** 让这些测试通过
- **约束条件：** 不要修改其他代码
- **预期输出：** 你发现问题及修复内容的摘要

### 3. 并行派发

在同一次响应中发出所有三个子代理派发——它们会并行运行：

```text
Subagent (general-purpose): "Fix agent-tool-abort.test.ts failures"
Subagent (general-purpose): "Fix batch-completion-behavior.test.ts failures"
Subagent (general-purpose): "Fix tool-approval-race-conditions.test.ts failures"
# All three run concurrently.
```

一次响应中的多个派发调用 = 并行执行。每个响应只发一个 = 顺序执行。

### 4. 审查与整合

当代理返回后：
- 阅读每个摘要
- 验证修复不会冲突
- 运行完整测试套件
- 整合所有变更

## 代理提示结构

良好的代理提示具备以下特点：
1. **聚焦** - 一个清晰的问题域
2. **自包含** - 理解问题所需的所有上下文
3. **输出明确** - 代理应该返回什么？

```markdown
Fix the 3 failing tests in src/agents/agent-tool-abort.test.ts:

1. "should abort tool with partial output capture" - expects 'interrupted at' in message
2. "should handle mixed completed and aborted tools" - fast tool aborted instead of completed
3. "should properly track pendingToolCount" - expects 3 results but gets 0

These are timing/race condition issues. Your task:

1. Read the test file and understand what each test verifies
2. Identify root cause - timing issues or actual bugs?
3. Fix by:
   - Replacing arbitrary timeouts with event-based waiting
   - Fixing bugs in abort implementation if found
   - Adjusting test expectations if testing changed behavior

Do NOT just increase timeouts - find the real issue.

Return: Summary of what you found and what you fixed.
```

## 常见错误

**❌ 范围太广：** "Fix all the tests" - 代理会迷失方向
**✅ 具体明确：** "Fix agent-tool-abort.test.ts" - 聚焦的范围

**❌ 没有上下文：** "Fix the race condition" - 代理不知道在哪里
**✅ 提供上下文：** 粘贴错误消息和测试名称

**❌ 没有约束：** 代理可能会重构所有内容
**✅ 设定约束：** "Do NOT change production code" 或 "Fix tests only"

**❌ 输出模糊：** "Fix it" - 你不知道改了什么
**✅ 输出明确：** "Return summary of root cause and changes"

## 何时不使用

**相关失败：** 修复一个可能会修复其他——先一起调查
**需要完整上下文：** 理解需要看到整个系统
**探索性调试：** 你还不知道哪里坏了
**共享状态：** 代理会相互干扰（编辑相同文件、使用相同资源）

## 会话中的真实示例

**场景：** 重大重构后 3 个文件共 6 个测试失败

**失败：**
- agent-tool-abort.test.ts: 3 个失败（时序问题）
- batch-completion-behavior.test.ts: 2 个失败（工具未执行）
- tool-approval-race-conditions.test.ts: 1 个失败（执行次数 = 0）

**决策：** 独立域——中止逻辑、批处理完成和竞态条件彼此独立

**派发：**
```
Agent 1 → Fix agent-tool-abort.test.ts
Agent 2 → Fix batch-completion-behavior.test.ts
Agent 3 → Fix tool-approval-race-conditions.test.ts
```

**结果：**
- Agent 1: 将超时替换为基于事件的等待
- Agent 2: 修复了事件结构 bug（threadId 位置错误）
- Agent 3: 添加了等待异步工具执行完成的逻辑

**整合：** 所有修复相互独立，无冲突，完整套件通过

**节省时间：** 3 个问题并行解决，而非顺序解决

## 主要优势

1. **并行化** - 多个调查同时进行
2. **聚焦** - 每个代理范围狭窄，需要跟踪的上下文更少
3. **独立性** - 代理之间互不干扰
4. **速度** - 3 个问题在 1 个时间内解决

## 验证

代理返回后：
1. **审查每个摘要** - 了解变更内容
2. **检查冲突** - 代理是否编辑了相同代码？
3. **运行完整套件** - 验证所有修复协同工作
4. **抽查** - 代理可能会出现系统性错误

## 实际影响

来自调试会话（2025-10-03）：
- 3 个文件共 6 个失败
- 3 个代理并行派发
- 所有调查并发完成
- 所有修复成功整合
- 代理变更之间零冲突
