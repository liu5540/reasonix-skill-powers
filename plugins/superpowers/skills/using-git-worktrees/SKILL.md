---
name: using-git-worktrees
description: 在需要与当前工作区隔离以开始功能开发，或执行实现方案之前使用——优先通过平台原生工具确保隔离工作区存在，必要时回退到手动 git worktree
---

# 使用 Git Worktrees

## 概述

确保工作在隔离的工作区中进行。优先使用你平台的原生 worktree 工具。仅当没有原生工具可用时，才回退到手动 git worktree。

**核心原则：** 首先检测现有隔离状态。然后使用原生工具。最后再回退到 git。不要与执行环境对抗。

**开始时声明：** "我正在使用 `using-git-worktrees` skill 设置一个隔离工作区。"

## 第 0 步：检测现有隔离

**在创建任何内容之前，检查你是否已经处于隔离工作区中。**

```bash
GIT_DIR=$(cd "$(git rev-parse --git-dir)" 2>/dev/null && pwd -P)
GIT_COMMON=$(cd "$(git rev-parse --git-common-dir)" 2>/dev/null && pwd -P)
BRANCH=$(git branch --show-current)
```

**子模块注意：** 在 git 子模块中，`GIT_DIR != GIT_COMMON` 同样成立。在得出“已在 worktree 中”的结论前，先确认你不是在子模块中：

```bash
# 如果返回路径，说明你在子模块中，而不是 worktree —— 按普通仓库处理
git rev-parse --show-superproject-working-tree 2>/dev/null
```

**如果 `GIT_DIR != GIT_COMMON`（且不是子模块）：** 你已经在链接的 worktree 中。跳到第 2 步（项目设置）。不要再创建另一个 worktree。

按分支状态报告：
- 在分支上："已在隔离工作区 `<path>`，分支 `<name>`。"
- 分离 HEAD："已在隔离工作区 `<path>`（分离 HEAD，由外部管理）。完成时需要创建分支。"

**如果 `GIT_DIR == GIT_COMMON`（或在子模块中）：** 你在普通仓库检出中。

用户是否已在你的指令中表明了对 worktree 的偏好？如果没有，在创建 worktree 前请求同意：

> "你希望我来设置一个隔离的 worktree 吗？这样可以保护你当前的分支不被改动。"

尊重任何已声明的偏好，无需再次询问。如果用户不同意，就在原地工作并跳到第 2 步。

## 第 1 步：创建隔离工作区

**你有两种机制。按以下顺序尝试。**

### 1a. 原生 Worktree 工具（首选）

用户已同意创建隔离工作区（第 0 步）。你是否已有创建工作区的方式？它可能是名为 `EnterWorktree`、`WorktreeCreate` 的工具，`/worktree` 命令，或 `--worktree` 标志。如果有，请使用它并跳到第 2 步。

原生工具会自动处理目录放置、分支创建和清理。当你拥有原生工具时仍然使用 `git worktree add`，会产生你的执行环境无法看到或管理的“幽灵状态”。

仅当没有可用的原生 worktree 工具时，才进入第 1b 步。

### 1b. Git Worktree 回退

**仅在第 1a 步不适用时使用**——即你没有可用的原生 worktree 工具。使用 git 手动创建 worktree。

#### 目录选择

遵循以下优先级顺序。用户显式指定的目录优先于文件系统当前状态。

1. **检查你的指令中是否声明了 worktree 目录偏好。** 如果用户已经指定，直接使用，无需询问。

2. **检查是否存在项目本地的 worktree 目录：**
   ```bash
   ls -d .worktrees 2>/dev/null     # 首选（隐藏）
   ls -d worktrees 2>/dev/null      # 替代
   ```
   如果找到，使用它。如果两者都存在，`.worktrees` 优先。

3. **如果没有其他可用指引**，默认使用项目根目录下的 `.worktrees/`。

#### 安全检查（仅项目本地目录）

**必须**在创建 worktree 前确认目录已被忽略：

```bash
git check-ignore -q .worktrees 2>/dev/null || git check-ignore -q worktrees 2>/dev/null
```

**如果未被忽略：** 将其加入 `.gitignore`，提交变更，然后再继续。

**为何关键：** 防止误将 worktree 内容提交到仓库。

#### 创建 Worktree

```bash
# 根据选定的位置确定路径
path="$LOCATION/$BRANCH_NAME"

git worktree add "$path" -b "$BRANCH_NAME"
cd "$path"
```

**沙箱回退：** 如果 `git worktree add` 因权限错误失败（沙箱拒绝），请告诉用户沙箱阻止了 worktree 创建，你将改在当前目录工作。然后在原地运行设置和基线测试。

## 第 2 步：项目设置

自动检测并运行相应的设置：

```bash
# Node.js
if [ -f package.json ]; then npm install; fi

# Rust
if [ -f Cargo.toml ]; then cargo build; fi

# Python
if [ -f requirements.txt ]; then pip install -r requirements.txt; fi
if [ -f pyproject.toml ]; then poetry install; fi

# Go
if [ -f go.mod ]; then go mod download; fi
```

## 第 3 步：验证干净的基线

运行测试，确保工作区初始状态干净：

```bash
# 使用适合项目的命令
npm test / cargo test / pytest / go test ./...
```

**如果测试失败：** 报告失败，并询问是继续还是调查。

**如果测试通过：** 报告准备就绪。

### 报告

```
Worktree ready at <full-path>
Tests passing (<N> tests, 0 failures)
Ready to implement <feature-name>
```

## 快速参考

| 场景 | 操作 |
|-----------|--------|
| 已在链接 worktree 中 | 跳过创建（第 0 步） |
| 在子模块中 | 按普通仓库处理（第 0 步检查） |
| 有原生 worktree 工具可用 | 使用它（第 1a 步） |
| 无原生工具 | Git worktree 回退（第 1b 步） |
| `.worktrees/` 存在 | 使用它（确认已忽略） |
| `worktrees/` 存在 | 使用它（确认已忽略） |
| 两者都存在 | 使用 `.worktrees/` |
| 都不存在 | 先检查指令文件，再默认 `.worktrees/` |
| 目录未被忽略 | 加入 `.gitignore` 并提交 |
| 创建时权限错误 | 沙箱回退，原地工作 |
| 基线测试失败 | 报告失败并询问 |
| 没有 package.json/Cargo.toml 等 | 跳过依赖安装 |

## 常见错误

### 与执行环境对抗

- **问题：** 当平台已经提供隔离时仍然使用 `git worktree add`
- **修复：** 第 0 步检测现有隔离；第 1a 步优先使用原生工具。

### 跳过检测

- **问题：** 在已有 worktree 中嵌套创建 worktree
- **修复：** 创建任何内容前始终运行第 0 步。

### 跳过忽略验证

- **问题：** worktree 内容被跟踪，污染 git 状态
- **修复：** 创建项目本地 worktree 前始终使用 `git check-ignore`。

### 假设目录位置

- **问题：** 造成不一致，违反项目约定
- **修复：** 遵循优先级：显式指令 > 现有项目本地目录 > 默认值

### 在测试失败时继续

- **问题：** 无法区分新 bug 与已有问题
- **修复：** 报告失败，获得明确许可后再继续

## 红线

**禁止：**
- 第 0 步检测到已有隔离时仍创建 worktree
- 当你拥有原生 worktree 工具（例如 `EnterWorktree`）时仍使用 `git worktree add`。这是头号错误——如果有，就使用它。
- 跳过第 1a 步，直接使用第 1b 步的 git 命令
- 未经确认已忽略就创建 worktree（项目本地）
- 跳过基线测试验证
- 未询问就在测试失败时继续

**必须：**
- 首先运行第 0 步检测
- 优先使用原生工具而非 git 回退
- 遵循目录优先级：显式指令 > 现有项目本地目录 > 默认值
- 确认项目本地目录已被忽略
- 自动检测并运行项目设置
- 验证干净的测试基线
