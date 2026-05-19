---
name: dispatching-parallel-agents
description: 当存在 2 个以上互不关联的问题（测试失败、bug、独立任务），需要并行分派子代理并发排查或执行时
---

# 并行分派子代理

## 概述

将多个独立任务并发委派给隔离的子代理。通过一次并行工具批次同时发起多个 `spawn_subagent` 或 `run_skill` 调用，让它们在后台并发工作，只返回最终答案。

**核心原则：** 一个独立问题域 = 一个子代理任务；多个调用放在同一 tool batch 中并行发出。

## 何时使用

**适用：**
- 3 个以上测试文件因不同根因失败
- 多个子系统独立出现故障
- 需要同时调查代码库不同区域
- 任务之间无共享文件/资源竞争

**不适用：**
- 失败有关联（修复一个可能修复其他）
- 需要完整系统状态才能决策
- 子代理会编辑同一文件
- 任务有明确顺序依赖

## 对接 Reasonix 工具

Reasonix 的 `spawn_subagent` 和 `run_skill` 标记为 `parallelSafe: true`，可在**同一 tool batch 中同时发出多个调用**，它们会并发执行。

### 方式一：直接调用 `spawn_subagent`

```
run_skill(name="implement", arguments="修复 src/agents/agent-tool-abort.test.ts 中 3 个失败测试...")
run_skill(name="implement", arguments="修复 src/agents/batch-completion.test.ts 中 2 个失败测试...")
run_skill(name="implement", arguments="修复 src/agents/tool-approval.test.ts 中 1 个失败测试...")
```

参数说明：
- `task`（**必需**）：子代理的具体任务。子代理**没有任何你的会话上下文**，task 就是它的全部输入
- `type`（可选）：内置人设。`explore`= 只读探索，`verify`= 验证检查
- `system`（可选）：自定义系统提示
- `model`（可选）：`deepseek-v4-flash`（默认）或 `deepseek-v4-pro`

### 方式二：调用 `run_skill` 使用已有 skill

```
run_skill(name="explore", arguments="找出 agent-tool-abort.test.ts 失败根因")
run_skill(name="research", arguments="调查事件结构是否与上次重构有关")
```

**注意：** `run_skill` 调用的 skill 若标记 `runAs: subagent`，会自动 spawn 子代理；若 `runAs: inline`，只是把 body 返回给你。

**预算提示：** 每 session 超过 4 次 spawn 会触发强制确认，建议单次不超过 5 个并行子代理。

## 操作流程

### 步骤 1：识别独立问题域

按故障/任务分组，确保各组之间：
- 不编辑同一文件
- 无顺序依赖
- 根因独立

### 步骤 2：构造自包含的任务描述

每个子代理没有任何你的上下文，task 必须包含：
- 明确的范围（一个文件/子系统）
- 清晰的目标（让哪些测试通过 / 回答什么问题）
- 必要的背景（失败信息、测试名称、相关路径）
- 约束条件（不要修改其他代码）
- 输出格式要求

**提示词模板：**

```markdown
修复 {文件路径} 中 {N} 个失败测试：

{列出失败测试名称和错误信息}

这些是 {问题类型} 问题。你的任务：
1. 阅读测试文件，理解验证内容
2. 找到根因
3. 修复（{具体方向 1}、{具体方向 2}）

约束：不要修改其他代码；不要只增加超时。
返回：根因分析 + 修改内容总结。
```

### 步骤 3：并行分派

在一次模型回合中**同时发出**所有子代理调用，不要等一个完成再发下一个：

```
▸ 并行分派 3 个子代理...
  子代理 A → agent-tool-abort 测试
  子代理 B → batch-completion 测试
  子代理 C → tool-approval 竞态条件
```

### 步骤 4：收集结果

每个子代理返回 JSON：

```json
{
  "success": true,
  "output": "子代理的最终总结...",
  "turns": 3,
  "tool_iters": 5,
  "elapsed_ms": 4200,
  "cost_usd": 0.003
}
```

若 `success: false`，查看 `error` 了解原因（被中止、超时、或任务无法完成）。

### 步骤 5：审查与集成

- 阅读每个 `output`，理解发现和修改建议
- 检查冲突：多个子代理是否建议编辑同一段代码？
- 运行完整验证（测试套件 / 构建）
- 抽查关键修改，子代理可能犯系统性错误

## 常见错误

| 错误 | 正确 |
|------|------|
| `run_skill(name="implement", arguments="修复所有测试")` — 范围太宽 | `run_skill(name="implement", arguments="修复 agent-tool-abort.test.ts 中 3 个失败测试...")` — 具体聚焦 |
| `run_skill(name="implement", arguments="修复竞态条件")` — 无上下文 | 提供具体路径和错误信息 |
| 顺序分派：等 A 完成再发 B | 同批次并行发出 A + B + C |
| 两个子代理编辑同一文件 | 提前确认各子代理目标文件互不重叠 |
| 对关联性失败并行分派 | 先排查是否指向同一根因，是则用一个子代理 |

## 验证清单

- [ ] 各子代理的 `task` 是否自包含？（不依赖你的会话上下文）
- [ ] 任务范围是否互不重叠？（无共享文件编辑）
- [ ] 是否在同一次 tool batch 中发出？（而非顺序等待）
- [ ] 返回结果是否都 `success: true`？
- [ ] 修改是否冲突？
- [ ] 是否运行了完整验证？
