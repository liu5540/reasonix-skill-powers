---
name: writing-plans
description: 在你拿到一份规格说明或多步骤任务的需求、但尚未动手编码时使用
---

# 编写计划

## 概述

撰写详尽的实现计划，并假设工程师对我们的代码库一无所知、品味也堪忧。记录下他们需要知道的一切：每个任务要改动哪些文件、代码怎么写、如何测试、需要查阅哪些文档。把整个计划拆成一口一个的小任务。遵循 DRY、YAGNI、TDD，频繁提交。

假设他们是熟练的开发者，但几乎不了解我们的工具集或问题域。同时假设他们对良好的测试设计也不太熟悉。

**开场声明：** "I'm using the writing-plans skill to create the implementation plan."

**上下文：** 如果你在隔离的 worktree 中工作，它应当在执行时通过 `superpowers:using-git-worktrees` 技能创建。

**保存位置：** `docs/superpowers/plans/YYYY-MM-DD-<feature-name>.md`(`YYYY-MM-DD`是当前日期)
- （用户对计划位置的偏好优先于此默认值）

## 范围检查

如果规格说明涉及多个相互独立的子系统，它应当在头脑风暴阶段就被拆分为子项目规格。如果没有，建议将其拆分为多个计划——每个子系统一个。每个计划都应独立产出可运行、可测试的软件。

## 文件结构

在定义任务之前，先梳理会创建或修改哪些文件，以及每个文件的职责。分解决策在这里确定下来。

- 设计边界清晰、接口明确的单元。每个文件应当只有一个清晰的职责。
- 你能同时记在脑子里的代码，思考起来最清晰；文件聚焦时，你的修改也更可靠。优先选择小而聚焦的文件，而不是职责过多的大文件。
- 一起变化的文件应当放在一起。按职责拆分，而不是按技术层次拆分。
- 在已有代码库中，遵循既定模式。如果代码库本身使用大文件，不要单方面重构——但如果你要修改的文件已经臃肿，在计划中提出拆分是合理的。

这一结构会指导任务分解。每个任务都应产生自成一体、独立可理解的变更。

## 任务粒度

任务是能够拥有自己完整测试周期的最小单元，也值得一次新的审阅把关。划分任务边界时：把搭建、配置、脚手架和文档步骤折叠到真正需要它们的交付任务中；只有在审阅者可能拒绝某个任务而批准相邻任务时，才进行拆分。每个任务都以一个可独立测试的交付物结束。

## 小步快跑的任务粒度

**每一步都是一个动作（2-5 分钟）：**
- "编写失败的测试" - 步骤
- "运行它，确保失败" - 步骤
- "编写最小实现让测试通过" - 步骤
- "运行测试，确保通过" - 步骤
- "提交" - 步骤

## 计划文档头部

**每个计划都必须以以下头部开头：**

```markdown
# [Feature Name] Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** [One sentence describing what this builds]

**Architecture:** [2-3 sentences about approach]

**Tech Stack:** [Key technologies/libraries]

## Global Constraints

[The spec's project-wide requirements — version floors, dependency limits,
naming and copy rules, platform requirements — one line each, with exact
values copied verbatim from the spec. Every task's requirements implicitly
include this section.]

---
```

## 任务结构

````markdown
### Task N: [Component Name]

**Files:**
- Create: `exact/path/to/file.py`
- Modify: `exact/path/to/existing.py:123-145`
- Test: `tests/exact/path/to/test.py`

**Interfaces:**
- Consumes: [what this task uses from earlier tasks — exact signatures]
- Produces: [what later tasks rely on — exact function names, parameter
  and return types. A task's implementer sees only their own task; this
  block is how they learn the names and types neighboring tasks use.]

- [ ] **Step 1: Write the failing test**

```python
def test_specific_behavior():
    result = function(input)
    assert result == expected
```

- [ ] **Step 2: Run test to verify it fails**

Run: `pytest tests/path/test.py::test_name -v`
Expected: FAIL with "function not defined"

- [ ] **Step 3: Write minimal implementation**

```python
def function(input):
    return expected
```

- [ ] **Step 4: Run test to verify it passes**

Run: `pytest tests/path/test.py::test_name -v`
Expected: PASS

- [ ] **Step 5: Commit**

```bash
git add tests/path/test.py src/path/file.py
git commit -m "feat: add specific feature"
```
````

## 禁止占位符

每一步都必须包含工程师实际需要的具体内容。以下都是**计划缺陷**——绝对不要写：
- "TBD"、"TODO"、"implement later"、"fill in details"
- "Add appropriate error handling" / "add validation" / "handle edge cases"
- "Write tests for the above"（没有给出实际测试代码）
- "Similar to Task N"（重复代码——工程师可能会跳着读任务）
- 只描述要做什么、却不展示怎么做的步骤（涉及代码的步骤必须包含代码块）
- 引用任何任务中未定义的类型、函数或方法

## 注意事项
- 始终给出精确的文件路径
- 每一步都包含完整代码——如果某一步修改了代码，就要展示代码
- 给出精确命令及其预期输出
- DRY、YAGNI、TDD、频繁提交

## 自我审查

写完完整计划后，用全新的眼光审视规格说明，并对照检查计划。这是你自己运行的清单——不是派发子代理去做的。

**1. 规格覆盖：** 浏览规格说明的每个章节/需求。你能指出实现它的任务吗？列出遗漏项。

**2. 占位符扫描：** 在计划中搜索危险信号——上面“禁止占位符”章节中的任何模式。修复它们。

**3. 类型一致性：** 你在后续任务中使用的类型、方法签名和属性名是否与前面任务中定义的一致？例如 Task 3 里叫 `clearLayers()`，而 Task 7 里却叫 `clearFullLayers()`，这就是 bug。

如果发现问题，直接_inline_修复。不需要重新审阅——修复后继续。如果发现某个规格需求没有对应任务，就补上该任务。

## 执行交接

保存计划后，提供执行选择：

**"Plan complete and saved to `docs/superpowers/plans/<filename>.md`. Two execution options:**

**1. Subagent-Driven (recommended)** - I dispatch a fresh subagent per task, review between tasks, fast iteration

**2. Inline Execution** - Execute tasks in this session using executing-plans, batch execution with checkpoints

**Which approach?"**

**如果选择 Subagent-Driven：**
- **必需子技能：** 使用 `superpowers:subagent-driven-development`
- 每个任务派一个新的子代理 + 两阶段审阅

**如果选择 Inline Execution：**
- **必需子技能：** 使用 `superpowers:executing-plans`
- 批量执行并设置检查点供审阅
