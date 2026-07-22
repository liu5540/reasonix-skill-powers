# Antigravity CLI（`agy`）工具映射

技能以动作形式表达（“调度子代理”、“创建待办事项”、“读取文件”）。在 Antigravity CLI（`agy`）上，这些动作对应到以下工具。

| 技能请求的动作 | Antigravity CLI 等效工具 |
|----------------------|----------------------|
| 调度子代理（`Subagent (general-purpose):` 模板） | `invoke_subagent` 配合内置 `TypeName` — `self` 用于全能力工作，`research` 用于只读工作（参见 [Subagent support](#subagent-support)） |
| 任务跟踪（“create a todo”、“mark complete”） | **task artifact** — 使用 `write_to_file` 并设置 `IsArtifact: true` 和 `ArtifactType: "task"`（参见 [Task tracking](#task-tracking)）。**不要**使用 `manage_task`，它管理的是后台进程。 |

## 任务跟踪

Antigravity 没有 todo 工具（`manage_task` 管理后台进程——`list`/`kill`/`status`/`send_input`——它*不是*检查清单）。当技能要求创建待办列表或跟踪任务时，维护一个 **task artifact**：一个使用 `write_to_file`（`IsArtifact: true`、`ArtifactMetadata.ArtifactType: "task"`）保存的 Markdown 检查清单，并通过 `replace_file_content` / `multi_replace_file_content` 进行编辑。

在任何多步骤任务开始时，创建 task artifact 列出计划中的每一步。完成每一步后，编辑该 artifact 标记为已完成（`- [x]`）。如果计划发生变化，更新检查清单。保持其最新状态——它是你剩余工作的真实来源；一旦对话变长，在每一步开始前重新阅读它。
