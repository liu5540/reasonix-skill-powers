---
name: subagent-driven-development
description: Use when executing an implementation plan with independent tasks in the current session — dispatches a fresh subagent per task with two-phase review (spec compliance then code quality) after each.
---

# 子代理驱动开发

## 概述

每个任务一个全新子代理 + 两阶段审查（先规格后质量）= 高质量快速迭代。

**核心原则：** 子代理有隔离上下文，不继承会话历史。你精确构造它们所需的一切。

## 何时使用

**使用条件：** 有实现计划、任务基本独立、留在当前会话。

**不满足时：** 无计划 → 先 brainstorming 或手动执行；任务紧密耦合 → 手动执行；并行会话 → 用 executing-plans。

## 前置检查

1. 确保已有实现计划 — 无计划时先调用 `run_skill(name="writing-plans")`
2. 评估任务复杂度：简单修复（1-2 个文件、少量代码、无架构变更）→ 跳过。其他情况 → 先调用 `run_skill(name="using-git-worktrees")` 建立隔离工作区

## 流程

1. 用 `read_file` 读取计划，提取所有任务完整文本，创建待办清单
2. 对每个任务循环：
   - 分派实现：`run_skill(name="implement", arguments="完整任务文本 + 上下文")`
   - 实现完成 → 规格审查：`run_skill(name="flow-review", arguments="审查实现是否匹配原始规格...")`
   - 规格不通过 → 实现者修复后重新审查
   - 规格通过 → 代码质量审查：`run_skill(name="flow-review", arguments="审查代码质量：命名、结构、重复、边界、测试")`
   - 质量不通过 → 实现者修复后重新审查
   - 通过 → 标记任务完成，进入下一任务
3. 全部完成后分派最终审查
4. 用 finishing-a-development-branch 收尾

## 处理实现者状态

| 状态 | 处理 |
|------|------|
| DONE | 进入规格合规审查 |
| DONE_WITH_CONCERNS | 阅读疑虑，涉及正确性/范围的在审查前解决 |
| NEEDS_CONTEXT | 提供缺失上下文并重新分派 |
| BLOCKED | 提供更多上下文 / 用更强模型 / 拆分任务 / 上报 |

绝不忽略上报或不做更改让同一模型重试。

## 模型选择

机械性实现（1-2 文件，清晰规格）→ 便宜快速模型。集成和判断（多文件协调）→ 标准模型。架构、设计和审查 → 最强模型。

## 红线

- 不经同意不在 main/master 实现
- 不跳过审查（规格或质量）
- 不带未修复问题继续
- 不并行分派多个实现子代理（会冲突）
- 不让子代理自己读计划文件（主代理读取后提供完整文本）
- 规格合规审查通过前不开始代码质量审查
- 任一审查有未解决问题时不进入下一任务
- 子代理提问 → 清晰完整回答，不催促
- 审查者发现问题 → 实现者修复 → 审查者再次审查 → 重复直到通过
- 子代理失败 → 分派修复子代理，不手动修复（上下文污染）


