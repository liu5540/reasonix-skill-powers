---
name: executing-plans
description: 当有一份书面实现计划需要在当前会话中逐步执行，并设有审查检查点时
---

# 执行计划

加载计划，批判性审查，使用工具逐步执行，验证每个步骤，完成后报告。

**开始时宣布：** "我正在使用 executing-plans 技能来实现此计划。"

**注意：** 如果任务相互独立且子代理可用，优先使用 subagent-driven-development。

## 前置条件

- [ ] 计划文件路径明确
- [ ] 不在 main/master 分支（未经同意不得修改）
- [ ] 已确认使用 using-git-worktrees 建立隔离工作区

## 流程

### 步骤 1：加载并审查

```
read_file(path="计划文件路径")
```

如有多个计划文件，一并读取。

**审查重点：**
- 步骤依赖顺序是否正确？
- 验证条件是否具体可执行？（"运行 go test ./... 通过"✓，"确认正常"✗）
- 是否有隐含环境假设？
- 涉及文件是否已存在？（用 `search_content` / `list_directory` 快速确认）

有疑虑时向用户提出，不要猜测。

### 步骤 2：建立追踪并执行

提取任务列表，在对话中维护待办：

```
▸ 任务 1/5：xxx
- [ ] 任务 2：xxx
...
```

**每个任务的循环：**

1. **标记进行中** — 声明当前任务
2. **读取相关代码** — `read_file`、`search_content` 理解现状
3. **修改实现** — `edit_file`（SEARCH/REPLACE）
4. **运行验证** — `run_command`（测试、lint、构建）
5. **提交** — `run_command(command="git commit -m '...'")`
6. **标记完成** — 更新待办，进入下一任务

**工具使用原则：**

| 场景 | 工具 |
|------|------|
| 读取文件 | `read_file` |
| 搜索代码 | `search_content`（内容）、`search_files`（文件名） |
| 修改代码 | `edit_file`（SEARCH/REPLACE） |
| 运行测试/构建/git | `run_command` |
| 启动服务器/watch | `run_background` |

**`run_command` vs `run_background`：** 需要等待结果做决策的用 `run_command`（测试、lint、git）；启动持续进程用 `run_background`（服务器、watch）。

**SEARCH/REPLACE 规范：**
- SEARCH 块精确匹配原文（含缩进）
- 每次一个独立变更
- 同一文件多处修改分多次调用
- 修改后读取确认

### 步骤 3：审查检查点

每完成 3 个任务暂停回顾：

```
run_command(command="git diff --stat HEAD~3")   # 查看变更范围
run_command(command="go test ./...")            # 完整回归测试
```

发现前期问题先修复，再继续。

### 步骤 4：完成收尾

全部完成后：
1. 运行最终验证
2. 生成执行报告（见模板）
3. 调用 finishing-a-development-branch 收尾

## 完成报告模板

```markdown
## 执行报告

**计划：** docs/plans/xxx.md
**分支：** feature/xxx
**任务：** N/N 已完成

### 完成的任务
1. ✅ xxx
2. ✅ xxx
...

### 验证结果
- 测试：X/X 通过
- lint：0 警告

### 偏离计划
- 任务 X：xxx（经用户同意）

### 下一步
按 finishing-a-development-branch 处理合并
```

## 异常处理

| 异常 | 处理 |
|------|------|
| 测试失败 | 读错误→定位→修复→重跑；同一失败 2 次以上停止求助 |
| 依赖缺失 | 停止，向用户报告 |
| 指令不清 | 列出理解+困惑，等用户澄清 |
| 计划缺陷 | 停止，建议修正方案 |

## 红线

- 未经同意不在 main/master 分支实现
- 不跳过验证
- 不猜测意图—— clarifying questions 是 feature
- 每个任务单独提交
- 验证失败不进入下一任务

## 集成

**必需技能：**
- using-git-worktrees — 建立隔离工作区
- writing-plans — 创建计划
- finishing-a-development-branch — 收尾
