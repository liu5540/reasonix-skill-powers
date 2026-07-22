## 子代理调度需要多代理支持

添加到你的 Codex 配置（`~/.codex/config.toml`）：

```toml
[features]
multi_agent = true
```

这为 `dispatching-parallel-agents` 和 `subagent-driven-development` 等技能启用了 `spawn_agent`、`wait_agent` 和 `close_agent`。在使用 `subagent-driven-development` 时，当实现者和审查者子代理完成所有工作后，你应始终关闭它们。

## 环境检测

创建 worktree 或完成分支的技能应在继续之前使用只读 git 命令检测其环境：

```bash
GIT_DIR=$(cd "$(git rev-parse --git-dir)" 2>/dev/null && pwd -P)
GIT_COMMON=$(cd "$(git rev-parse --git-common-dir)" 2>/dev/null && pwd -P)
BRANCH=$(git branch --show-current)
```

- `GIT_DIR != GIT_COMMON` → 已经处于 linked worktree 中（跳过创建）
- `BRANCH` 为空 → 分离 HEAD 状态（无法从沙箱中创建分支/推送/发起 PR）

请参阅 `using-git-worktrees` 的步骤 0 和 `finishing-a-development-branch` 的步骤 1，了解每个技能如何使用这些信号。

## Codex App 收尾

当沙箱阻止分支/推送操作（外部管理的 worktree 中处于分离 HEAD 状态）时，代理会提交所有工作，并告知用户使用 App 的原生控件：

- **“Create branch”** — 命名分支，然后通过 App UI 提交/推送/发起 PR
- **“Hand off to local”** — 将工作转移到用户的本地检出目录

代理仍然可以运行测试、暂存文件，并输出建议的分支名称、提交消息和 PR 描述供用户复制。
