---
name: requesting-code-review
description: Use after completing a task, implementing a major feature, or before merging — to verify work against requirements by dispatching a reviewer subagent.
---

# 请求代码审查

派遣 flow-review 子代理在问题扩散前发现它们。审查者获得精心组织的评估上下文，不是你的会话历史。

**核心原则：** 早审查，勤审查。

## 何时请求

**必须审查：** 子代理驱动开发中每个任务完成后、完成重要功能后、合并到 main 之前。

**可选：** 卡住时（换视角）、重构前（建立基线）、修复复杂 bug 后。

## 如何请求

1. 获取 git SHA：
```bash
BASE_SHA=$(git rev-parse HEAD~1)    # 或 origin/main
HEAD_SHA=$(git rev-parse HEAD)
```

2. 派遣审查子代理：
```
run_skill(
  name="flow-review",
  arguments="审查当前分支变更。实现内容：{WHAT_WAS_IMPLEMENTED}。参考计划：{PLAN_OR_REQUIREMENTS}。范围：{BASE_SHA}..{HEAD_SHA}。"
)
```

3. 处理反馈：关键问题立即修复、重要问题继续前修复、建议问题稍后处理、审查者有误用技术理由反驳。

## 示例

```
[刚完成任务 2：添加验证功能]

run_command(command="git rev-parse HEAD~3")    # BASE_SHA
run_command(command="git rev-parse HEAD")      # HEAD_SHA

run_skill(
  name="flow-review",
  arguments="审查从 a7981ec 到 3df7661 的变更。实现了 verifyIndex() 和 repairIndex()，支持 4 种问题类型。参考计划：deployment-plan.md 任务 2。"
)

[子代理返回]:
  优点：架构清晰，测试真实
  问题：Important：缺少进度指示器 / Minor：魔法数字 100
  评估：可以继续

[修复进度指示器，继续任务 3]
```

## 与工作流集成

**子代理驱动开发：** 每个任务完成后审查，修复后再进下一任务。

**执行计划：** 每批（3 个任务）后审查，获取反馈、修复、继续。

**临时开发：** 合并前审查、卡住时审查。

## 红线

- 不因"很简单"跳过审查
- 不忽略关键问题
- 不带着未修复的重要问题继续
- 不对合理反馈争辩——审查者有误则用技术理由反驳
