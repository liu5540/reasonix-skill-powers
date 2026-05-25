---
name: writing-plans
description: Use when you have a spec or requirements for a multi-step task, before writing any code. Creates detailed implementation plans for zero-context engineers to execute.
---

# 编写计划

## 概述

编写全面的实现计划，假设执行者对代码库零上下文、品味存疑。记录每个任务要修改的文件、代码、测试、命令。将计划拆成小步骤任务（每步 2-5 分钟）。DRY、YAGNI、TDD、频繁 commit。

**计划保存位置：** `docs/superpowers/plans/<current-date>-<feature-name>.md` (use ISO 8601 date, e.g. 2025-06-15-plan-name.md)

## 范围检查

规格涵盖多个独立子系统 → 拆分为独立计划，每个子系统一个。每个计划独立产出可工作、可测试的软件。

## 文件结构

定义任务前先列出将创建或修改的文件及职责。设计边界清晰、接口良好的单元。优先小而专注的文件。一起变更的文件放在一起。在现有代码库中遵循已有模式。

## 计划文档头部

```markdown
# [功能名称] 实现计划

> 使用 subagent-driven-development（推荐）或 executing-plans 逐任务执行。步骤用复选框（`- [ ]`）跟踪进度。

**目标：** [一句话描述]

**架构：** [2-3 句话描述方案]

**技术栈：** [关键技术/库]

---
```

## 任务结构

每个任务包含精确文件路径、完整代码、精确命令和预期输出：

```markdown
### 任务 N：[组件名称]

**文件：**
- 创建：`exact/path/to/file.py`
- 修改：`exact/path/to/existing.py:123-145`
- 测试：`tests/exact/path/to/test.py`

- [ ] **步骤 1：编写失败的测试**
```python
def test_specific_behavior():
    result = function(input)
    assert result == expected
```

- [ ] **步骤 2：运行测试验证失败**
运行：`pytest tests/path/test.py::test_name -v`
预期：FAIL，报错 "function not defined"

- [ ] **步骤 3：编写最少实现代码**
```python
def function(input):
    return expected
```

- [ ] **步骤 4：运行测试验证通过**
运行：`pytest tests/path/test.py::test_name -v`
预期：PASS

- [ ] **步骤 5：Commit**
```bash
git add tests/path/test.py src/path/file.py
git commit -m 'feat: add specific feature'
```
```

## 禁止占位符

绝不要写："待定"、"TODO"、"后续实现"、"添加适当的错误处理"、"为上述代码编写测试"（无实际代码）、"类似任务 N"（重复代码）、只描述不展示代码的步骤、引用未在任何任务中定义的类型/函数。

## 自检

编写完成后以全新视角对照规格检查：

1. **规格覆盖度** — 每个需求都有对应任务？列出遗漏。
2. **占位符扫描** — 搜索"待定"/"TODO"/"适当的"/"后续"等红旗。修复。
3. **类型一致性** — 后续任务中使用的类型/方法签名/属性名与前面定义一致？

发现问题直接内联修复。需求无对应任务则添加任务。

## 执行交接

保存后提供两个选项：子代理驱动（推荐，每个任务新子代理 + 两阶段审查）或内联执行（当前会话用 executing-plans 批量执行）。

**必需技能：** 子代理驱动用 subagent-driven-development，内联用 executing-plans。
