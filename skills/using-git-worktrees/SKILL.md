---
name: using-git-worktrees
description: Use when starting feature development that needs isolation from the current workspace, or before executing an implementation plan — creates isolated git worktrees with smart directory selection and safety verification.
---

# 使用 Git 工作树

## 概述

Git 工作树创建共享同一仓库的隔离工作区，允许同时在多个分支上工作无需切换。

**核心原则：** 系统化目录选择 + 安全验证 = 可靠隔离。

## 目录选择（优先级从高到低）

1. **检查现有目录** — `ls -d .worktrees 2>/dev/null`（首选隐藏目录），其次 `ls -d worktrees 2>/dev/null`。两者都存在用 `.worktrees/`。
2. **检查 AGENTS.md** — `grep -i "worktree.*director" AGENTS.md`，有偏好直接使用。
3. **询问用户** — 都不存在则问：`.worktrees/`（项目本地）还是 `~/.config/reasonix/worktrees/<project>/`（全局）。

## 安全验证

项目本地目录必须验证已忽略：`git check-ignore -q .worktrees`。未忽略则在 `.gitignore` 添加条目并提交。全局目录无需验证。

## 创建步骤

1. **检测项目名** — `git rev-parse --show-toplevel` 提取项目名
2. **创建工作树** — `git worktree add <path> -b <branch-name>`
3. **运行项目设置** — 自动检测项目类型（`ls package.json Cargo.toml requirements.txt ...`），用 `run_background` 运行对应安装命令
4. **验证基线** — `run_background(command="npm test")`，确保初始状态干净
5. **报告就绪** — 告知工作树路径和测试结果

测试失败则报告失败情况，询问是否继续或排查。测试通过则报告就绪。

## 快速参考

| 情况 | 操作 |
|------|------|
| `.worktrees/` 存在 | 使用它（验证已忽略） |
| 都不存在 | 检查 AGENTS.md → 询问用户 |
| 目录未被忽略 | 添加到 .gitignore + 提交 |
| 基线测试失败 | 报告失败 + 询问 |
| 无 package.json/Cargo.toml | 跳过依赖安装 |

## 常见错误

跳过忽略验证 → 工作树内容被跟踪，污染 git status。假设目录位置 → 造成不一致。带着失败测试继续 → 无法区分新 bug 和已有问题。硬编码设置命令 → 不同工具链项目出错。

## 红线

- 创建项目本地工作树前必须验证已忽略
- 不跳过基线测试验证
- 不询问不带着失败测试继续
- 遵循目录优先级：现有目录 > AGENTS.md > 询问

## 上下游

**谁调用你：** brainstorming（设计批准后）、subagent-driven-development（非简单任务时）、executing-plans（开始执行前）。

**你完成后：** 下游技能（subagent-driven-development 或 executing-plans）继续执行任务。

**最终清理：** finishing-a-development-branch 的步骤 5 会调用 `git worktree remove` 清理本技能创建的工作树。
