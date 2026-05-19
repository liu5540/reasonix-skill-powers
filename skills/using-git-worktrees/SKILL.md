---
name: using-git-worktrees
description: 当需要开始与当前工作区隔离的功能开发或执行实现计划之前使用——创建具有智能目录选择和安全验证的隔离 git 工作树
---

# 使用 Git 工作树

## 概述

Git 工作树创建共享同一仓库的隔离工作区，允许同时在多个分支上工作而无需切换。

**核心原则：** 系统化的目录选择 + 安全验证 = 可靠的隔离。

**开始时宣布：** "我正在使用 using-git-worktrees 技能来建立一个隔离的工作区。"

## 目录选择流程

按以下优先顺序执行：

### 1. 检查现有目录

```bash
run_command(command="ls -d .worktrees 2>/dev/null")     # 首选（隐藏目录）
run_command(command="ls -d worktrees 2>/dev/null")      # 备选
```

**如果找到：** 使用该目录。如果两者都存在，`.worktrees` 优先。

### 2. 检查 AGENTS.md

```bash
run_command(command="grep -i \"worktree.*director\" AGENTS.md 2>/dev/null")
```

**如果指定了偏好：** 直接使用，无需询问。

### 3. 询问用户

如果无现有目录且 AGENTS.md 中无偏好：

```
未找到工作树目录。我应该在哪里创建工作树？
1. .worktrees/（项目本地，隐藏目录）
2. ~/.config/reasonix/worktrees/<project-name>/（全局位置）
你倾向哪个？
```

## 安全验证

### 项目本地目录

创建工作树前必须验证目录已被忽略：

```bash
run_command(command="git check-ignore -q .worktrees 2>/dev/null || git check-ignore -q worktrees 2>/dev/null")
```

**如果未被忽略：** 根据"立即修复坏掉的东西"原则，在 `.gitignore` 中添加相应条目并提交，然后继续。

### 全局目录

无需 `.gitignore` 验证——完全在项目之外。

## 创建步骤

### 1. 检测项目名称

```
run_command(command="git rev-parse --show-toplevel")   # 获取仓库根目录，提取项目名
```

### 2. 创建工作树

```
# 根据目录类型构造路径，然后创建工作树
run_command(command="git worktree add <path> -b <branch-name>")
# 注意：cd 在 run_command 中不持久，后续命令使用完整路径或 cwd 参数
```

### 3. 运行项目设置

自动检测并运行相应的设置命令（耗时较长，使用后台运行）：

```
# 检测项目类型并后台运行对应设置命令
run_command(command="ls package.json Cargo.toml requirements.txt pyproject.toml go.mod 2>/dev/null")
# 根据存在的文件选择对应的设置命令：
# run_background(command="npm install")          # Node.js
# run_background(command="cargo build")          # Rust
# run_background(command="pip install -r requirements.txt")  # Python
# run_background(command="poetry install")       # Python (poetry)
# run_background(command="go mod download")      # Go
```

### 4. 验证基线正常

运行测试确保工作树初始状态干净（耗时较长，使用后台运行）：

```bash
run_background(command="npm test")  # 或 cargo test / pytest / go test ./...
```

**如果测试失败：** 报告失败情况，询问是否继续或排查。
**如果测试通过：** 报告就绪。

### 5. 报告位置

```
工作树已就绪：<full-path>
测试通过（<N> 个测试，0 个失败）
准备实现 <feature-name>
```

## 快速参考

| 情况 | 操作 |
|------|------|
| `.worktrees/` 存在 | 使用它（验证已忽略） |
| `worktrees/` 存在 | 使用它（验证已忽略） |
| 两者都存在 | 使用 `.worktrees/` |
| 都不存在 | 检查 AGENTS.md → 询问用户 |
| 目录未被忽略 | 添加到 .gitignore + 提交 |
| 基线测试失败 | 报告失败 + 询问 |
| 无 package.json/Cargo.toml | 跳过依赖安装 |

## 常见错误

- **跳过忽略验证** → 工作树内容被跟踪，污染 git status。修复：创建项目本地工作树前始终使用 `run_command(command="git check-ignore -q <dir>")`。
- **假设目录位置** → 造成不一致。修复：遵循优先级：现有目录 > AGENTS.md > 询问。
- **带着失败的测试继续** → 无法区分新 bug 和已有问题。修复：报告失败，获得明确许可后再继续。
- **硬编码设置命令** → 在使用不同工具的项目上会出错。修复：从项目文件自动检测。

## 示例工作流

```
你：我正在使用 using-git-worktrees 技能来建立一个隔离的工作区。

[检查 .worktrees/ - run_command(command="ls -d .worktrees 2>/dev/null") 存在]
[验证已忽略 - run_command(command="git check-ignore -q .worktrees 2>/dev/null") 确认 .worktrees/ 已被忽略]
[创建工作树：run_command(command="git worktree add .worktrees/auth -b feature/auth")]
[运行项目设置：run_background(command="npm install")]
[运行测试：run_background(command="npm test") - 47 个通过]

工作树已就绪：/Users/jesse/myproject/.worktrees/auth
测试通过（47 个测试，0 个失败）
准备实现 auth 功能
```

## 红线

**绝不：**
- 创建项目本地工作树时不验证是否已忽略
- 跳过基线测试验证
- 不询问就带着失败的测试继续
- 在有歧义时假设目录位置

**始终：**
- 遵循目录优先级：现有目录 > AGENTS.md > 询问
- 对项目本地目录验证是否已忽略
- 自动检测并运行项目设置
- 验证测试基线干净

## 集成

**被以下技能调用：** brainstorming（阶段 4）、subagent-driven-development、executing-plans，以及任何需要隔离工作区的技能。

**配合使用：** finishing-a-development-branch - 工作完成后清理时必需。
