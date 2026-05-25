---
name: finishing-a-development-branch
description: Use when implementation is complete, all tests pass, and you need to decide how to integrate the work — merge, PR, keep, or discard.
---

# 完成开发分支

## 概述

验证测试 → 展示选项 → 执行选择 → 清理。

**核心原则：** 在提供选项前必须验证测试通过。不确认不删除。

## 流程

### 步骤 1：验证测试通过

在展示选项之前，运行测试验证：

```bash
run_command(command="npm test")  # 或 cargo test / pytest / go test ./...
```

测试失败则停止，不要继续。测试通过则进入步骤 2。

### 步骤 2：确定基础分支

```bash
run_command(command="git merge-base HEAD main 2>/dev/null || git merge-base HEAD master 2>/dev/null")
```

或直接询问用户确认。

### 步骤 3：展示 4 个选项

```
实现已完成。你想怎么做？

1. 在本地合并回 <base-branch>
2. 推送并创建 Pull Request
3. 保持分支现状（我稍后处理）
4. 丢弃这项工作
```

### 步骤 4：执行选择

**选项 1 — 本地合并：**
```bash
git checkout <base-branch> && git pull
git merge <feature-branch>
npm test           # 验证
git branch -d <feature-branch>
```

**选项 2 — 推送并创建 PR：**
```bash
git push -u origin <feature-branch>
gh pr create --title '<title>' --body '<summary>'
```

**选项 3 — 保持现状：** 不清理工作树。报告"保留分支 <name>，工作树保留在 <path>"。

**选项 4 — 丢弃：** 先要求输入 'discard' 确认，确认后：
```bash
git checkout <base-branch>
git branch -D <feature-branch>
```

### 步骤 5：清理工作树

选项 1、2、4 时清理工作树：
```bash
git worktree list          # 找到工作树路径
git worktree remove <path> # 如果在工作树中
```

选项 3 保留工作树，跳过此步。

## 快速参考

| 选项 | 合并 | 推送 | 保留工作树 | 清理分支 |
|------|------|------|-----------|---------|
| 1. 本地合并 | ✓ | - | - | ✓ |
| 2. 创建 PR | - | ✓ | ✓ | - |
| 3. 保持现状 | - | - | ✓ | - |
| 4. 丢弃 | - | - | - | ✓（强制） |

## 红线

- 测试失败时不继续
- 合并前不验证测试
- 不确认不删除工作成果
- 选项 4 必须输入 'discard' 确认
- 只在选项 1 和 4 时清理工作树

## 前置与后续步骤

**上游调用者：** subagent-driven-development 和 executing-plans 在所有任务完成后应调用本技能收尾。

**清理工作树：** 如果使用了 using-git-worktrees 创建隔离工作区，步骤 5 中执行 `git worktree remove <path>` 清理。
