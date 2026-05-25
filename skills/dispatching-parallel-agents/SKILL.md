---
name: dispatching-parallel-agents
description: Use when there are 2+ independent problems (test failures, bugs, separate tasks) that can be investigated or executed concurrently by subagents with no shared files.
---

# 并行分派子代理

## 概述

将多个独立任务并发委派给隔离的子代理。在同一 tool batch 中同时发起多个 `run_skill` 调用，子代理并发工作，只返回最终答案。

**核心原则：** 一个独立问题域 = 一个子代理任务。多个调用放在同一 batch 中并行发出。

## 何时使用

**适用：** 3 个以上测试文件因不同根因失败、多个子系统独立故障、需同时调查代码库不同区域、任务间无共享文件。

**不适用：** 失败有关联（修复一个可能修复其他）、子代理会编辑同一文件、任务有明确顺序依赖。

## 操作流程

### 步骤 1：识别独立问题域

确保各组之间不编辑同一文件、无顺序依赖、根因独立。

### 步骤 2：构造自包含任务描述

子代理没有任何会话上下文，task 必须包含：明确范围、清晰目标、失败信息/测试名称/路径、约束条件、输出格式。

### 步骤 3：并行分派

在同一次 tool batch 中同时发出所有调用：

```
run_skill(name="implement", arguments="修复 agent-tool-abort.test.ts 中 3 个失败测试...")
run_skill(name="implement", arguments="修复 batch-completion.test.ts 中 2 个失败测试...")
run_skill(name="implement", arguments="修复 tool-approval.test.ts 中 1 个失败测试...")
```

选择 `run_skill(name="skill-name", arguments="...")` 调用已有 skill；标记 `runAs: subagent` 的 skill 会自动 spawn 子代理。

**预算提示：** 建议单次不超过 5 个并行子代理。

### 步骤 4：收集结果

每个子代理返回 `{ success, output, turns, tool_iters, elapsed_ms, cost_usd }`。若 `success: false`，查看 `error` 了解原因。

### 步骤 5：审查与集成

阅读每个 output → 检查冲突（多个子代理编辑同一段代码？）→ 运行完整验证 → 抽查关键修改。

## 常见错误

| 错误 | 正确 |
|------|------|
| 范围太宽："修复所有测试" | 具体聚焦："修复 agent-tool-abort.test.ts 中 3 个测试" |
| 无上下文："修复竞态条件" | 提供具体路径和错误信息 |
| 顺序分派：等 A 完成再发 B | 同批次并行发出 A + B + C |
| 两个子代理编辑同一文件 | 提前确认目标文件互不重叠 |
| 关联性失败并行分派 | 先排查是否同一根因，是则用一个子代理 |

## 验证清单

- [ ] 各子代理 task 自包含（不依赖会话上下文）
- [ ] 任务范围互不重叠（无共享文件编辑）
- [ ] 在同一次 tool batch 中发出（非顺序等待）
- [ ] 返回结果都 success: true
- [ ] 修改无冲突
- [ ] 运行了完整验证
