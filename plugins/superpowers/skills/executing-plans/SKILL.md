---
name: executing-plans
description: 当你有一份书面实施计划需要在独立会话中逐步执行并设置检查点时使用
---

# 执行计划

## 概述

加载计划，批判性审阅，执行所有任务，完成后报告。

**开始时声明：** "I'm using the executing-plans skill to implement this plan."

**注意：** 告诉你的合作伙伴，Superpowers 在有子代理支持的环境中效果更佳。如果在支持子代理的平台上运行（Claude Code、Codex CLI、Codex App 和 Copilot CLI 都符合条件；请参阅 `../using-superpowers/references/` 中的各平台工具参考），工作质量会显著提高。如果子代理可用，请使用 superpowers:subagent-driven-development 替代本 skill。

## 流程

### 步骤 1：加载并审阅计划
1. 读取计划文件
2. 批判性审阅 — 找出计划中的任何问题或疑虑
3. 如有疑虑：开始前先向合作伙伴提出
4. 如无疑虑：为计划项创建待办事项并继续

### 步骤 2：执行任务

对于每项任务：
1. 标记为进行中
2. 严格遵循每一步（计划包含小而明确的步骤）
3. 按要求运行验证
4. 标记为已完成

### 步骤 3：完成开发

所有任务完成并验证后：
- 声明："I'm using the finishing-a-development-branch skill to complete this work."
- **必需子技能：** 使用 superpowers:finishing-a-development-branch
- 按照该 skill 验证测试、呈现选项并执行选择

## 何时停止并求助

**出现以下情况时立即停止执行：**
- 遇到阻塞项（缺少依赖、测试失败、指令不清）
- 计划存在严重漏洞导致无法开始
- 不理解某项指令
- 验证反复失败

**应请求澄清，而不是猜测。**

## 何时返回 earlier 步骤

**出现以下情况时返回审阅（步骤 1）：**
- 合作伙伴根据你的反馈更新了计划
- 基本方法需要重新思考

**不要强行绕过阻塞项** —— 停下来提问。

## 记住
- 首先批判性审阅计划
- 严格遵循计划步骤
- 不要跳过验证
- 当计划要求时引用 skill
- 遇到阻塞时停止，不要猜测
- 未经用户明确同意，切勿在 main/master 分支上开始实施

## 集成

**必需的工作流 skill：**
- **superpowers:using-git-worktrees** - 确保隔离的工作空间（创建或验证现有空间）
- **superpowers:writing-plans** - 创建本 skill 执行的计划
- **superpowers:finishing-a-development-branch** - 完成所有任务后的开发收尾
