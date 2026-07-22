---
name: finishing-a-development-branch
description: 在实现完成、所有测试通过，且需要决定如何集成工作时使用——通过提供结构化的合并、PR 或清理选项来指导开发工作的收尾。
---

# 完成开发分支

## 概述

通过提供清晰的选项并执行所选的工作流，指导开发工作的收尾。

**核心原则：** 验证测试 → 检测环境 → 呈现选项 → 执行选择 → 清理。

**开始时声明：** “我正在使用 finishing-a-development-branch skill 来完成这项工作。”

## 流程

### 第 1 步：验证测试

**在呈现选项之前，验证测试是否通过：**

```bash
# 运行项目的测试套件
npm test / cargo test / pytest / go test ./...
```

**如果测试失败：**
```
测试失败（<N> 处失败）。必须先修复才能完成：

[显示失败信息]

在测试通过之前，无法继续合并/提交 PR。
```

停止。不要进入第 2 步。

**如果测试通过：** 继续第 2 步。

### 第 2 步：检测环境

**在呈现选项之前，确定工作区状态：**

```bash
GIT_DIR=$(cd "$(git rev-parse --git-dir)" 2>/dev/null && pwd -P)
GIT_COMMON=$(cd "$(git rev-parse --git-common-dir)" 2>/dev/null && pwd -P)
```

这将决定显示哪个菜单以及清理方式：

| 状态 | 菜单 | 清理 |
|------|------|---------|
| `GIT_DIR == GIT_COMMON`（普通仓库） | 标准 4 个选项 | 无需清理 worktree |
| `GIT_DIR != GIT_COMMON`，命名分支 | 标准 4 个选项 | 基于来源（见第 6 步） |
| `GIT_DIR != GIT_COMMON`，分离 HEAD | 精简 3 个选项（无合并） | 无需清理（外部管理） |

### 第 3 步：确定基础分支

```bash
# 尝试常见的基础分支
git merge-base HEAD main 2>/dev/null || git merge-base HEAD master 2>/dev/null
```

或询问：“该分支是从 main 分出的——是否正确？”

### 第 4 步：呈现选项

**普通仓库和命名分支 worktree —— 仅显示以下 4 个选项：**

```
实现已完成。你想做什么？

1. 本地合并回 <base-branch>
2. 推送并创建 Pull Request
3. 保持原样（我稍后自己处理）
4. 丢弃此工作

请选择哪个选项？
```

**分离 HEAD —— 仅显示以下 3 个选项：**

```
实现已完成。你当前处于分离 HEAD 状态（外部管理的工作空间）。

1. 推送为新分支并创建 Pull Request
2. 保持原样（我稍后自己处理）
3. 丢弃此工作

请选择哪个选项？
```

**不要添加解释** —— 保持选项简洁。

### 第 5 步：执行选择

#### 选项 1：本地合并

```bash
# 获取主仓库根目录，确保当前工作目录安全
MAIN_ROOT=$(git -C "$(git rev-parse --git-common-dir)/.." rev-parse --show-toplevel)
cd "$MAIN_ROOT"

# 先合并——在删除任何内容之前确认成功
git checkout <base-branch>
git pull
git merge <feature-branch>

# 验证合并后的测试结果
<test command>

# 只有在合并成功后：清理 worktree（第 6 步），然后删除分支
```

然后：清理 worktree（第 6 步），再删除分支：

```bash
git branch -d <feature-branch>
```

#### 选项 2：推送并创建 PR

```bash
# 推送分支
git push -u origin <feature-branch>
```

**不要清理 worktree** —— 用户需要它来处理 PR 反馈并进行迭代。

#### 选项 3：保持原样

报告：“保持分支 <name>。worktree 保留在 <path>。”

**不要清理 worktree。**

#### 选项 4：丢弃

**先确认：**
```
这将永久删除：
- 分支 <name>
- 所有提交：<commit-list>
- 位于 <path> 的 worktree

输入 'discard' 以确认。
```

等待精确确认。

如果确认：
```bash
MAIN_ROOT=$(git -C "$(git rev-parse --git-common-dir)/.." rev-parse --show-toplevel)
cd "$MAIN_ROOT"
```

然后：清理 worktree（第 6 步），再强制删除分支：
```bash
git branch -D <feature-branch>
```

### 第 6 步：清理工作空间

**仅对选项 1 和 4 运行。** 选项 2 和 3 始终保留 worktree。

```bash
GIT_DIR=$(cd "$(git rev-parse --git-dir)" 2>/dev/null && pwd -P)
GIT_COMMON=$(cd "$(git rev-parse --git-common-dir)" 2>/dev/null && pwd -P)
WORKTREE_PATH=$(git rev-parse --show-toplevel)
```

**如果 `GIT_DIR == GIT_COMMON`：** 普通仓库，无需清理 worktree。完成。

**如果 worktree 路径位于 `.worktrees/` 或 `worktrees/` 下：** Superpowers 创建了该 worktree —— 我们负责清理。

```bash
MAIN_ROOT=$(git -C "$(git rev-parse --git-common-dir)/.." rev-parse --show-toplevel)
cd "$MAIN_ROOT"
git worktree remove "$WORKTREE_PATH"
git worktree prune  # 自我修复：清理所有过期的注册信息
```

**否则：** 主环境（harness）拥有该工作空间。不要删除它。如果你的平台提供了 workspace-exit 工具，请使用它。否则，保留该工作空间。

## 速查表

| 选项 | 合并 | 推送 | 保留 Worktree | 清理分支 |
|--------|-------|------|---------------|----------------|
| 1. 本地合并 | 是 | - | - | 是 |
| 2. 创建 PR | - | 是 | 是 | - |
| 3. 保持原样 | - | - | 是 | - |
| 4. 丢弃 | - | - | - | 是（强制） |

## 常见错误

**跳过测试验证**
- **问题：** 合并没有通过测试的代码，创建失败的 PR
- **修复：** 在提供选项之前始终验证测试

**开放式问题**
- **问题：** “我接下来该做什么？”含义模糊
- **修复：** 仅提供 4 个结构化选项（分离 HEAD 时为 3 个）

**为选项 2 清理 worktree**
- **问题：** 删除了用户处理 PR 反馈所需的 worktree
- **修复：** 仅对选项 1 和 4 进行清理

**在移除 worktree 之前删除分支**
- **问题：** `git branch -d` 失败，因为 worktree 仍引用该分支
- **修复：** 先合并，再移除 worktree，然后删除分支

**在 worktree 内部运行 git worktree remove**
- **问题：** 当当前工作目录位于待移除的 worktree 中时，命令静默失败
- **修复：** 在执行 `git worktree remove` 前始终 `cd` 到主仓库根目录

**清理 harness 拥有的 worktree**
- **问题：** 删除 harness 创建的 worktree 会导致幽灵状态
- **修复：** 仅清理位于 `.worktrees/` 或 `worktrees/` 下的 worktree

**丢弃前未确认**
- **问题：** 误删工作成果
- **修复：** 要求输入 "discard" 确认

## 红线

**永远不要：**
- 在测试失败时继续
- 未在合并结果上验证测试就合并
- 未确认就删除工作成果
- 未明确要求就强制推送
- 在确认合并成功前移除 worktree
- 清理非你创建的 worktree（来源检查）
- 在 worktree 内部运行 `git worktree remove`

**始终要：**
- 在提供选项前验证测试
- 在展示菜单前检测环境
- 仅提供 4 个选项（分离 HEAD 时为 3 个）
- 为选项 4 获取输入确认
- 仅对选项 1 和 4 清理 worktree
- 在移除 worktree 前 `cd` 到主仓库根目录
- 在移除后运行 `git worktree prune`
