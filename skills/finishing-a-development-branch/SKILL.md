---
name: finishing-a-development-branch
description: 当实现完成、所有测试通过、需要决定如何集成工作时使用——呈现合并/PR/保留/丢弃结构化选项
---

# 完成开发分支

## 概述

通过呈现清晰选项并处理所选工作流来引导开发工作的完成。

**核心原则：** 验证测试 → 检测环境 → 呈现选项 → 执行选择 → 清理。

**开始时宣布：** "正在使用完成开发分支技能来完成这项工作。"

## 过程

### 第1步：验证测试

**在呈现选项前，验证测试通过：**

```bash
go test ./...
```

**如果测试失败：**
```
测试失败（<N> 个失败）。必须修复后才能完成：

[展示失败]

在测试通过前不能继续合并/PR。
```

停止。在测试通过前不进行第2步。

**如果测试通过：** 继续第2步。

### 第2步：检测环境

```bash
GIT_DIR=$(cd "$(git rev-parse --git-dir)" 2>/dev/null && pwd -P)
GIT_COMMON=$(cd "$(git rev-parse --git-common-dir)" 2>/dev/null && pwd -P)
```

| 状态 | 菜单 | 清理 |
|------|------|------|
| `GIT_DIR == GIT_COMMON`（普通仓库） | 标准4选项 | 无 |
| `GIT_DIR != GIT_COMMON`，命名分支 | 标准4选项 | 按来源清理 |
| `GIT_DIR != GIT_COMMON`，分离 HEAD | 精简3选项（无合并） | 不清理 |

### 第3步：确定基础分支

```bash
git merge-base HEAD main 2>/dev/null || git merge-base HEAD master 2>/dev/null
```

### 第4步：呈现选项

**普通仓库和命名分支工作树 — 呈现恰好这4个选项：**

```
实现完成。你想做什么？

1. 合并回 <基础分支> 本地
2. 推送并创建 Pull Request
3. 保持分支现状（我稍后处理）
4. 丢弃这项工作

哪个选项？
```

**分离 HEAD — 呈现恰好这3个选项：**

```
实现完成。你处于分离 HEAD（外部管理工作空间）。

1. 作为新分支推送并创建 Pull Request
2. 保持现状（我稍后处理）
3. 丢弃这项工作

哪个选项？
```

### 第5步：执行选择

#### 选项1：本地合并

```bash
MAIN_ROOT=$(git -C "$(git rev-parse --git-common-dir)/.." rev-parse --show-toplevel)
cd "$MAIN_ROOT"
git checkout <基础分支>
git pull
git merge <功能分支>
# 验证合并结果上的测试
go test ./...
# 合并成功后才移除工作树和删除分支
```

#### 选项2：推送并创建 PR

```bash
git push -u origin <功能分支>
gh pr create --title "<标题>" --body "..."
```

**不要清理工作树** — 用户需要它来迭代 PR 反馈。

#### 选项3：保持现状

报告："保持分支 <名称>。工作树保留。"

**不要清理工作树。**

#### 选项4：丢弃

**先确认：**
```
这将永久删除：
- 分支 <名称>
- 所有提交：<提交列表>

输入 'discard' 确认。
```

等待精确确认。

### 第6步：清理工作空间

**仅对选项1和4执行。** 选项2和3始终保留工作树。

如果 `GIT_DIR == GIT_COMMON`：普通仓库，无事清理。

如果工作树路径在 `.worktrees/` 或 `worktrees/` 下：
```bash
MAIN_ROOT=$(git -C "$(git rev-parse --git-common-dir)/.." rev-parse --show-toplevel)
cd "$MAIN_ROOT"
git worktree remove "$WORKTREE_PATH"
git worktree prune
```

否则：宿主环境拥有此工作空间，不要移除。

## 红线

**绝不：**
- 带着失败测试继续
- 未验证合并结果测试就合并
- 未经确认删除工作
- 未经明确请求强制推送
- 在选项2和3清理工作树
- 从工作树内部运行 `git worktree remove`
- 清理不是你创建的工作树
