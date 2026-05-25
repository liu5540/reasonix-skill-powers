---
name: executing-plans
description: Use when you have a written implementation plan to execute step-by-step in the current session, with review checkpoints. If tasks are independent and subagents are available, prefer subagent-driven-development.
---

# 执行计划

加载计划，批判性审查，逐步执行，每步验证，完成后报告。

**核心原则：** 不在 main/master 分支实现。不跳过验证。不猜测意图。

## 前置条件

- 计划文件路径明确 — 无计划时先调用 `run_skill(name="writing-plans")` 创建
- 不在 main/master 分支
- 已调用 `run_skill(name="using-git-worktrees")` 建立隔离工作区

## 流程

### 步骤 1：加载并审查计划

用 `read_file` 读取计划文件。审查：步骤依赖顺序是否正确？验证条件是否具体可执行？是否有隐含环境假设？涉及文件是否存在？有疑虑向用户提出，不猜测。

### 步骤 2：逐任务执行

每个任务的循环：

1. 标记进行中
2. 用 `read_file` / `search_content` 读取相关代码
3. 用 `edit_file`（SEARCH/REPLACE）修改实现，每次一个独立变更
4. 用 `run_command` 运行验证（测试/lint/构建）
5. 用 `run_command(command="git commit -m '...'")` 提交
6. 标记完成，进入下一任务

`run_command` 用于需等待结果做决策的场景（测试、lint、git）；`run_background` 用于持续进程（服务器、watch）。

`edit_file` 规范：SEARCH 块精确匹配原文（含缩进）、每次一个独立变更、修改后读取确认。

### 步骤 3：审查检查点

每完成 3 个任务暂停回顾：`git diff --stat HEAD~3` + 完整回归测试。发现前期问题先修复再继续。

### 步骤 4：完成收尾

运行最终验证 → 生成执行报告 → 调用 finishing-a-development-branch 收尾。

## 执行报告模板

```markdown
## 执行报告

**计划：** docs/plans/xxx.md
**分支：** feature/xxx
**任务：** N/N 已完成

### 完成的任务
1. ✅ xxx
...

### 验证结果
- 测试：X/X 通过
- lint：0 警告

### 偏离计划
- 任务 X：xxx（经用户同意）

### 下一步
按 finishing-a-development-branch 处理合并
```

## 异常处理

| 异常 | 处理 |
|------|------|
| 测试失败 | 读错误→定位→修复→重跑；同一失败 2 次以上停止求助 |
| 依赖缺失 | 停止，向用户报告 |
| 指令不清 | 列出理解+困惑，等用户澄清 |
| 计划缺陷 | 停止，建议修正方案 |

## 红线

- 未经同意不在 main/master 分支实现
- 不跳过验证
- 不猜测意图
- 每个任务单独提交
- 验证失败不进入下一任务


