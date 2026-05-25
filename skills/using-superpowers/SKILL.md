---
name: using-superpowers
description: Use at the start of any conversation — establishes how to find and use skills. If you think there's even a 1% chance a skill applies, you must invoke it. Subagents dispatched for specific tasks should skip this skill.
---

# 使用 Superpowers

## 指令优先级

1. **用户明确指令**（AGENTS.md、直接请求）— 最高优先级
2. **技能** — 覆盖默认系统行为
3. **默认系统提示** — 最低优先级

用户指令说"不用 TDD"，技能说"始终用 TDD"→ 遵循用户指令。

## 如何访问技能

技能位于 `<project>/.reasonix/skills/<name>/SKILL.md` 或 `~/.reasonix/skills/<name>/SKILL.md`。

## 使用规则

在任何响应或操作之前调用相关或被请求的技能。哪怕只有 1% 可能性某个技能适用，都必须调用它来检查。调用后发现不适用可以不使用。

## 工作流程

收到用户消息 → 已经头脑风暴过？否 → 调用 brainstorming。是 → 可能有技能适用？否 → 响应。是（哪怕 1%）→ 调用技能 → 宣布"使用 [技能] 来 [目的]" → 严格遵循技能。

## 退出条件硬化

brainstorming 技能的设计批准 ≠ 可以开始实现。设计批准后仍有强制步骤：编写设计文档 → 规格自检 → 用户审查书面规格 → 调用 writing-plans。所有步骤完成前不能进入实现阶段。

## 红线

以下想法意味着在合理化——立刻停止：

"这只是一个简单的问题"、"我需要先了解更多上下文"、"让我先探索一下代码库"、"这不需要正式的技能"、"我记得这个技能"、"技能太小题大做了"、"让我先做这一件事"、"设计已经批准了直接实现吧"、"改动很小事后补文档就行"

## 技能优先级

流程技能优先（brainstorming、systematic-debugging）→ 实现技能其次。"构建 X" → 先 brainstorming 再实现。"修复 bug" → 先 debugging 再领域技能。

## 中文场景技能路由

检测到以下场景时必须优先调用对应技能：

- 代码审查且中文沟通 → chinese-code-review
- Gitee/Coding/极狐 GitLab → chinese-git-workflow
- 编写中文技术文档/README → chinese-documentation
- 写 git commit（中文项目）→ chinese-commit-conventions

判断依据：项目有中文注释/README/`.gitee` 目录、commit 历史有中文、用户用中文交流。

## 技能类型

刚性（TDD、调试）：严格遵循。柔性（模式）：根据上下文调整。技能本身会告诉你属于哪种。