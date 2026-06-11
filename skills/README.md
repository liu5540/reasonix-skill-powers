# 技能索引

> 基于 [Superpowers v5.1.0](https://github.com/obra/superpowers/tree/v5.1.0) 翻译适配，面向 Reasonix 1.3.0 编码助手。

## 快速开始

在 AGENTS.md 中已声明行为准则。每个技能通过 `/技能名 参数（可选）` 调用。技能会在触发条件匹配时自动建议。

**核心规则：** 如果某个技能有≥1%的可能性适用，先调用它再行动。

---

## 技能全景

### 引导技能

| 技能名 | 类型 | 何时触发 |
|--------|------|----------|
| `using-superpowers` | inline | 任何对话开始——建立技能查找和使用规则 |

### 核心开发流程（推荐按此顺序使用）

| 技能名 | 中文名 | 类型 | 何时触发 |
|--------|--------|------|----------|
| `brainstorming` | 头脑风暴 | subagent | 任何创造性工作前——构建功能、添加组件、修改行为。澄清需求→产出设计文档 |
| `writing-plans` | 编写计划 | inline | 有规格/设计后——拆解为小口任务(2-5分钟/步)，精确文件路径+完整代码 |
| `subagent-driven-development` | 子代理驱动开发 | subagent | 有实现计划且任务独立——每任务分派全新子代理，两阶段审查(规格+质量) |
| `executing-plans` | 内联执行计划 | inline | 有实现计划但在当前会话执行——批量执行带检查点 |
| `test-driven-development` | 测试驱动开发 | inline | 任何功能实现/Bug修复前——强制红-绿-重构循环 |
| `requesting-code-review` | 请求代码审查 | inline | 完成任务/功能后、合并前——分派审查子代理 |
| `receiving-code-review` | 接收代码审查 | inline | 收到审查反馈时——技术验证、禁止表演性同意 |
| `finishing-a-development-branch` | 完成开发分支 | inline | 实现完成、测试通过后——合并/PR/保留/丢弃选项 |

### 调试与质量

| 技能名 | 中文名 | 类型 | 何时触发 |
|--------|--------|------|----------|
| `systematic-debugging` | 系统性调试 | inline | 遇到Bug/测试失败/意外行为——四阶段根因分析，禁止治标 |
| `verification-before-completion` | 完成前验证 | inline | 声称完成/修复前——强制运行验证命令，证据先于断言 |

### 工程基础设施

| 技能名 | 中文名 | 类型 | 何时触发 |
|--------|--------|------|----------|
| `using-git-worktrees` | Git工作树 | subagent | 需要隔离工作空间时——检测已有隔离+git worktree回退 |
| `dispatching-parallel-agents` | 并行分派 | subagent | 2+个独立任务可并行时——每问题域一个子代理 |

### 元技能

| 技能名 | 中文名 | 类型 | 何时触发 |
|--------|--------|------|----------|
| `writing-skills` | 编写技能 | inline | 创建/编辑/验证技能——TDD红绿重构应用于过程文档 |

---

## 工作流示例

### 从零构建新功能

```
用户: "添加用户排行榜功能"
→ using-superpowers: 检查技能适用性
→ brainstorming: 澄清需求、生成设计文档 docs/superpowers/specs/<当前日期>-排行榜-design.md
→ writing-plans: 拆解为 bite-sized 任务 docs/superpowers/plans/<当前日期>-排行榜-plan.md
→ subagent-driven-development: 逐任务分派子代理（每任务含 TDD → 审查循环）
→ finishing-a-development-branch: 合并/PR 选项
```

### 修复 Bug

```
用户: "玩家积分有时不保存"
→ systematic-debugging: 四阶段根因分析
→ test-driven-development: 先写失败回归测试
→ verification-before-completion: 验证修复
```

### 批量测试失败

```
→ systematic-debugging: 分析是否为独立失败
→ dispatching-parallel-agents: 如果独立则并行分派修复
```

---

## 文件结构

```
.reasonix/skills/
├── using-superpowers/SKILL.md     # 技能系统引导
├── brainstorming/SKILL.md         # 头脑风暴
├── writing-plans/SKILL.md         # 编写计划
├── subagent-driven-development/
│   ├── SKILL.md                   # 子代理驱动开发
│   ├── implementer-prompt.md      # 实现者提示模板
│   ├── spec-reviewer-prompt.md    # 规格审查者提示模板
│   └── code-quality-reviewer-prompt.md # 代码质量审查者提示模板
├── test-driven-development/SKILL.md
├── requesting-code-review/SKILL.md
├── receiving-code-review/SKILL.md
├── finishing-a-development-branch/SKILL.md
├── systematic-debugging/SKILL.md
├── verification-before-completion/SKILL.md
├── using-git-worktrees/SKILL.md
├── executing-plans/SKILL.md
├── dispatching-parallel-agents/SKILL.md
├── writing-skills/SKILL.md
└── README.md                      # 本文件
```
