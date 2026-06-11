---
name: using-git-worktrees
description: 在开始需要与当前工作空间隔离的功能工作时使用——通过检测已有隔离+git worktree回退确保隔离工作空间
runAs: subagent
---

# Git 工作树隔离

## 概述

确保工作在隔离的工作空间中进行。

**核心原则：** 先检测已有隔离。再检查是否有原生工作树工具可用。没有则用 git worktree 回退方案。

**开始时宣布：** "正在使用 Git 工作树技能设置隔离工作空间。"

## 第0步：检测已有隔离

**创建任何东西前，先检查是否已在隔离工作空间中。**

```bash
GIT_DIR=$(cd "$(git rev-parse --git-dir)" 2>/dev/null && pwd -P)
GIT_COMMON=$(cd "$(git rev-parse --git-common-dir)" 2>/dev/null && pwd -P)
BRANCH=$(git branch --show-current)
```

**子模块守卫：** `GIT_DIR != GIT_COMMON` 在 git 子模块中也成立。用以下命令排除子模块：
```bash
git rev-parse --show-superproject-working-tree 2>/dev/null
```

**如果 `GIT_DIR != GIT_COMMON`（且不是子模块）：** 已在链接的工作树中。跳到第3步（项目设置）。

**如果 `GIT_DIR == GIT_COMMON`（或在子模块中）：** 在普通仓库检出中。询问用户是否创建隔离工作树。

## 第1步：创建隔离工作空间

### 1a. 原生工作树工具（优先）

Reasonix 可能提供原生工作树工具。查找 `EnterWorktree` 类似的工具。如果有，使用它。原生工具处理目录放置、分支创建和自动清理。

### 1b. Git Worktree 回退

仅在无原生工具时使用。手动创建 worktree：

#### 目录选择

优先级顺序：
1. 检查指令文件中声明的工作树目录偏好
2. 检查已有项目本地目录：`.worktrees/`（优先）或 `worktrees/`
3. 默认 `.worktrees/`

#### 安全验证（仅项目本地目录）

```bash
git check-ignore -q .worktrees 2>/dev/null || git check-ignore -q worktrees 2>/dev/null
```

如果未被忽略：添加到 `.gitignore`，提交变更，再继续。

#### 创建工作树

```bash
project=$(basename "$(git rev-parse --show-toplevel)")
git worktree add ".worktrees/$BRANCH_NAME" -b "$BRANCH_NAME"
cd ".worktrees/$BRANCH_NAME"
```

## 第3步：项目设置

```bash
# Go 项目
if [ -f go.mod ]; then go mod download; fi
```

## 第4步：验证干净基线

```bash
go test ./...
```

**如果测试失败：** 报告失败，询问是否继续或调查。

**如果测试通过：** 报告就绪。

```
工作树就绪：<完整路径>
测试通过（<N> 个测试，0 失败）
准备实现 <功能名>
```

## 红线

**绝不：**
- 在第0步检测到已有隔离时创建新工作树
- 有原生工具时使用 `git worktree add`
- 跳过第0步检测
- 未验证目录被忽略就创建
- 带着失败测试继续

**始终：**
- 先运行第0步检测
- 优先原生工具而非 git 回退
- 自动检测并运行项目设置
- 验证干净测试基线
