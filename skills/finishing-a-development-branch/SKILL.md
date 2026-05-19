---
name: finishing-a-development-branch
description: 当实现完成、所有测试通过、需要决定如何集成工作时使用——通过提供合并、PR 或清理等结构化选项来引导开发工作的收尾
---

# 完成开发分支

## 概述

通过提供清晰的选项并执行所选工作流来引导开发工作的收尾。

**核心原则：** 验证测试 → 展示选项 → 执行选择 → 清理。

**开始时宣布：** "我正在使用 finishing-a-development-branch 技能来完成这项工作。"

## 流程

开始工作时复制此清单，逐项标记进度：

```
完成进度：
- [ ] 步骤 1：验证测试通过
- [ ] 步骤 2：确定基础分支
- [ ] 步骤 3：展示选项并等待用户选择
- [ ] 步骤 4：执行选择（合并/PR/保留/丢弃）
- [ ] 步骤 5：清理工作树
```

**每完成一步，将 `[ ]` 改为 `[x]`。**

### 步骤 1：验证测试

**在展示选项之前，验证测试通过：**

> 测试文件, 使用后台运行 `run_background`

```bash
run_background(command="npm test")  # 或 cargo test / pytest / go test ./...
```

**如果测试失败：** 显示失败信息，停止。不要继续到步骤 2。

**如果测试通过：** 将清单改为 `[x] 步骤 1`，继续步骤 2。

### 步骤 2：确定基础分支

```bash
run_command(command="git merge-base HEAD main 2>/dev/null || git merge-base HEAD master 2>/dev/null")
```

或者询问："这个分支是从 main 分出来的——对吗？"

完成后将清单改为 `[x] 步骤 2`。

### 步骤 3：展示选项

展示以下 4 个选项，不要添加解释：

```
实现已完成。你想怎么做？

1. 在本地合并回 <base-branch>
2. 推送并创建 Pull Request
3. 保持分支现状（我稍后处理）
4. 丢弃这项工作

选哪个？
```

用户选择后，将清单改为 `[x] 步骤 3`，进入步骤 4。

### 步骤 4：执行选择

#### 选项 1：本地合并

```bash
run_command(command="git checkout <base-branch>")
run_command(command="git pull")
run_command(command="git merge <feature-branch>")
run_background(command="npm test")  # 或 cargo test / pytest / go test ./...
run_command(command="git branch -d <feature-branch>")  # 测试通过后
```

全部完成后将子清单全部改为 `[x]`，主清单 `[x] 步骤 4`，进入步骤 5。

#### 选项 2：推送并创建 PR

```bash
run_command(command="git push -u origin <feature-branch>")
run_command(command="gh pr create --title '<title>' --body '## 摘要\n<2-3 条变更要点>\n\n## 测试计划\n- [ ] <验证步骤>'")
```

完成后将子清单全部改为 `[x]`，主清单 `[x] 步骤 4`，进入步骤 5。

#### 选项 3：保持现状

主清单 `[x] 步骤 4`。**不要清理工作树，跳过步骤 5。**

报告："保留分支 <name>。工作树保留在 <path>。"

#### 选项 4：丢弃

**先确认：**
```
这将永久删除：
- 分支 <name>
- 所有提交：<commit-list>
- 工作树 <path>

输入 'discard' 确认。
```

等待精确的确认。确认后：
```bash
run_command(command="git checkout <base-branch>")
run_command(command="git branch -D <feature-branch>")
```

将子清单改为 `[x]`，主清单 `[x] 步骤 4`，然后清理工作树（步骤 5）。

### 步骤 5：清理工作树

**对于选项 1、2、4：**

```bash
run_command(command="git branch --show-current")   # 获取当前分支名
run_command(command="git worktree list")             # 列出工作树，从中找到对应路径
run_command(command="git worktree remove <worktree-path>")  # 如果在工作树中
```

**对于选项 3：** 保留工作树（已跳过）。

步骤 5 完成后，将主清单全部标记为 `[x]`。

## 快速参考

| 选项 | 合并 | 推送 | 保留工作树 | 清理分支 |
|------|------|------|-----------|---------|
| 1. 本地合并 | ✓ | - | - | ✓ |
| 2. 创建 PR | - | ✓ | ✓ | - |
| 3. 保持现状 | - | - | ✓ | - |
| 4. 丢弃 | - | - | - | ✓（强制） |

## 常见错误

| 错误 | 问题 | 修复 |
|------|------|------|
| 跳过测试验证 | 合并损坏的代码、创建失败的 PR | 在提供选项前始终验证测试 |
| 开放式问题 | "接下来该做什么？" → 含糊不清 | 准确展示 4 个结构化选项 |
| 自动清理工作树 | 在可能还需要工作树时就删除了 | 只在选项 1 和 4 时清理 |
| 丢弃时不确认 | 意外删除工作成果 | 要求输入 "discard" 确认 |

## 红线

**绝不：**
- 在测试失败时继续
- 合并前不验证测试结果
- 不确认就删除工作成果
- 未经明确请求就强制推送

**始终：**
- 在提供选项前验证测试
- 准确展示 4 个选项
- 选项 4 要求输入确认
- 只在选项 1 和 4 时清理工作树

## 集成

**被以下技能调用：**
- **subagent-driven-development**（所有任务完成后）
- **executing-plans**（所有批次完成后）

**配合使用：**
- **using-git-worktrees** - 清理由该技能创建的工作树
