# 技能编写最佳实践

> 学习如何编写高效的 Skill，让 Agent 能够成功发现并有效使用。

优秀的 Skill 简洁、结构清晰，并经过真实使用场景的测试。本指南提供实用的编写决策建议，帮助你写出 Agent 能够发现并有效使用的 Skill。

关于 Skill 工作原理的概念背景，请参见 [Skills overview](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview)。

## 核心原则

### 简洁是关键

[上下文窗口](https://platform.claude.com/docs/en/build-with-claude/context-windows) 是一种公共资源。你的 Skill 会与 Agent 所需了解的其他所有内容共享上下文窗口，包括：

* 系统提示词
* 对话历史
* 其他 Skill 的元数据
* 用户的实际请求

Skill 中的每个 token 并非都会产生即时成本。启动时，只会预加载所有 Skill 的元数据（name 和 description）。Agent 仅在 Skill 变得相关时才会阅读 SKILL.md，并且只在需要时阅读其他文件。然而，SKILL.md 中的内容一旦加载，每个 token 都会与对话历史和其他上下文竞争，因此保持简洁仍然很重要。

**默认假设**：Agent 已经非常聪明

只添加 Agent 尚不了解的上下文。对每条信息提出质疑：

* “Agent 真的需要这条解释吗？”
* “我能假设 Agent 已经知道这一点吗？”
* “这段文字是否对得起它消耗的 token？”

**好示例：简洁**（约 50 个 token）：

````markdown  theme={null}
## 提取 PDF 文本

使用 pdfplumber 提取文本：

```python
import pdfplumber

with pdfplumber.open("file.pdf") as pdf:
    text = pdf.pages[0].extract_text()
```
`````

**坏示例：太冗长**（约 150 个 token）：

```markdown  theme={null}
## Extract PDF text

PDF (Portable Document Format) files are a common file format that contains
text, images, and other content. To extract text from a PDF, you'll need to
use a library. There are many libraries available for PDF processing, but we
recommend pdfplumber because it's easy to use and handles most cases well.
First, you'll need to install it using pip. Then you can use the code below...
```

简洁版本假设 Agent 知道 PDF 是什么以及如何使用库。

### 设置适当的自由度

根据任务的脆弱性和可变性，匹配合适的具体程度。

**高自由度**（基于文本的指令）：

使用时机：

* 多种方法都有效
* 决策取决于上下文
* 启发式方法指导方案

示例：

```markdown  theme={null}
## Code review process

1. Analyze the code structure and organization
2. Check for potential bugs or edge cases
3. Suggest improvements for readability and maintainability
4. Verify adherence to project conventions
```

**中等自由度**（带参数的伪代码或脚本）：

使用时机：

* 存在首选模式
* 允许一定变化
* 配置会影响行为

示例：

````markdown  theme={null}
## Generate report

Use this template and customize as needed:

```python
def generate_report(data, format="markdown", include_charts=True):
    # Process data
    # Generate output in specified format
    # Optionally include visualizations
```
`````

**低自由度**（特定脚本，很少或没有参数）：

使用时机：

* 操作脆弱且容易出错
* 一致性至关重要
* 必须遵循特定顺序

示例：

````markdown  theme={null}
## Database migration

Run exactly this script:

```bash
python scripts/migrate.py --verify --backup
```

Do not modify the command or add additional flags.
`````

**类比**：把 Agent 想象成在路径上探索的机器人：

* **两侧都是悬崖的窄桥**：只有一条安全的前进道路。提供具体的护栏和精确指令（低自由度）。示例：必须按精确顺序运行的数据库迁移。
* **没有障碍物的开阔田野**：很多路径都能成功。给出大致方向，相信 Agent 能找到最佳路线（高自由度）。示例：代码审查，其中上下文决定最佳方法。

### 在所有计划使用的模型上测试

Skill 作为模型的补充，其有效性取决于底层模型。在所有计划使用的模型上测试你的 Skill。

**按模型区分的测试考虑**：

* **Claude Haiku**（快速、经济）：Skill 是否提供了足够指导？
* **Claude Sonnet**（平衡）：Skill 是否清晰高效？
* **Claude Opus**（强推理）：Skill 是否避免过度解释？

对 Opus 完美的内容，对 Haiku 可能需要更多细节。如果你计划跨多个模型使用 Skill，目标应设定为对所有模型都效果良好的指令。

## Skill 结构

<Note>
  **YAML Frontmatter**: SKILL.md 的 frontmatter 需要两个字段：

  * `name` - Skill 的可读名称（最多 64 个字符）
  * `description` - 一行描述，说明 Skill 做什么以及何时使用（最多 1024 个字符）

  完整的 Skill 结构细节请参见 [Skills overview](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview#skill-structure)。
</Note>

### 命名约定

使用一致的命名模式，让 Skill 更容易引用和讨论。我们建议对 Skill 名称使用 **动名词形式**（动词 + -ing），因为这能清楚描述 Skill 提供的活动或能力。

**好的命名示例（动名词形式）**：

* "Processing PDFs"
* "Analyzing spreadsheets"
* "Managing databases"
* "Testing code"
* "Writing documentation"

**可接受的替代方案**：

* 名词短语："PDF Processing", "Spreadsheet Analysis"
* 动作导向："Process PDFs", "Analyze Spreadsheets"

**避免**：

* 模糊名称："Helper", "Utils", "Tools"
* 过于通用："Documents", "Data", "Files"
* 技能集合中不一致的模式

一致的命名让以下事情更容易：

* 在文档和对话中引用 Skill
* 一眼看出 Skill 的作用
* 整理和搜索多个 Skill
* 维护一个专业、一致的技能库

### 编写有效的 description

`description` 字段让 Skill 可被 discovery，应同时包含 Skill 做什么以及何时使用。

<Warning>
  **始终使用第三人称**。description 会被注入系统提示词，不一致的人称视角可能导致发现 problem。

  * **好：** "Processes Excel files and generates reports"
  * **避免：** "I can help you process Excel files"
  * **避免：** "You can use this to process Excel files"
</Warning>

**具体并包含关键词**。同时包含 Skill 做什么以及何时使用的具体触发器/上下文。

每个 Skill 只有一个 description 字段。description 对技能选择至关重要：Agent 用它从可能 100 多个可用 Skill 中选择合适的 Skill。你的 description 必须提供足够细节，让 Agent 知道何时选择这个 Skill，而 SKILL.md 的其余部分提供实现细节。

有效示例：

**PDF Processing skill:**

```yaml  theme={null}
description: Extract text and tables from PDF files, fill forms, merge documents. Use when working with PDF files or when the user mentions PDFs, forms, or document extraction.
```

**Excel Analysis skill:**

```yaml  theme={null}
description: Analyze Excel spreadsheets, create pivot tables, generate charts. Use when analyzing Excel files, spreadsheets, tabular data, or .xlsx files.
```

**Git Commit Helper skill:**

```yaml  theme={null}
description: Generate descriptive commit messages by analyzing git diffs. Use when the user asks for help writing commit messages or reviewing staged changes.
```

避免如下模糊的 description：

```yaml  theme={null}
description: Helps with documents
```

```yaml  theme={null}
description: Processes data
```

```yaml  theme={null}
description: Does stuff with files
```

### 渐进式披露模式

SKILL.md 作为 overview，根据需要指向详细材料，就像入职指南中的目录。关于渐进式披露如何运作的解释，请参见 overview 中的 [How Skills work](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview#how-skills-work)。

**实用指导：**

* 为获得最佳性能，SKILL.md 正文控制在 500 行以内
* 接近此限制时，将内容拆分到单独文件中
* 使用以下模式有效组织指令、代码和资源

#### 视觉概览：从简单到复杂

一个基本 Skill 从仅包含 SKILL.md 文件开始，其中有元数据和指令：

<img src="https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-simple-file.png?fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=87782ff239b297d9a9e8e1b72ed72db9" alt="Simple SKILL.md file showing YAML frontmatter and markdown body" data-og-width="2048" width="2048" data-og-height="1153" height="1153" data-path="images/agent-skills-simple-file.png" data-optimize="true" data-opv="3" srcset="https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-simple-file.png?w=280&fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=c61cc33b6f5855809907f7fda94cd80e 280w, https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-simple-file.png?w=560&fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=90d2c0c1c76b36e8d485f49e0810dbfd 560w, https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-simple-file.png?w=840&fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=ad17d231ac7b0bea7e5b4d58fb4aeabb 840w, https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-simple-file.png?w=1100&fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=f5d0a7a3c668435bb0aee9a3a8f8c329 1100w, https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-simple-file.png?w=1650&fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=0e927c1af9de5799cfe557d12249f6e6 1650w, https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-simple-file.png?w=2500&fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=46bbb1a51dd4c8202a470ac8c80a893d 2500w" />

随着 Skill 增长，你可以捆绑仅在需要时加载的额外内容：

<img src="https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-bundling-content.png?fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=a5e0aa41e3d53985a7e3e43668a33ea3" alt="Bundling additional reference files like reference.md and forms.md." data-og-width="2048" width="2048" data-og-height="1327" height="1327" data-path="images/agent-skills-bundling-content.png" data-optimize="true" data-opv="3" srcset="https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-bundling-content.png?w=280&fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=f8a0e73783e99b4a643d79eac86b70a2 280w, https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-bundling-content.png?w=560&fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=dc510a2a9d3f14359416b706f067904a 560w, https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-bundling-content.png?w=840&fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=82cd6286c966303f7dd914c28170e385 840w, https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-bundling-content.png?w=1100&fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=56f3be36c77e4fe4b523df209a6824c6 1100w, https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-bundling-content.png?w=1650&fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=d22b5161b2075656417d56f41a74f3dd 1650w, https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-bundling-content.png?w=2500&fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=3dd4bdd6850ffcc96c6c45fcb0acd6eb 2500w" />

完整的 Skill 目录结构可能如下：

```
pdf/
├── SKILL.md              # 需要时加载的主指令
├── FORMS.md              # 需要时加载的表单填写指南
├── reference.md          # 需要时加载的 API 参考
├── examples.md           # 需要时加载的使用示例
└── scripts/
    ├── analyze_form.py   # 执行的实用脚本（不加载）
    ├── fill_form.py      # 表单填写脚本
    └── validate.py       # 验证脚本
```

#### 模式 1：带参考的高级指南

````markdown  theme={null}
---
name: PDF Processing
description: Extracts text and tables from PDF files, fills forms, and merges documents. Use when working with PDF files or when the user mentions PDFs, forms, or document extraction.
---

# PDF Processing

## Quick start

Extract text with pdfplumber:
```python
import pdfplumber
with pdfplumber.open("file.pdf") as pdf:
    text = pdf.pages[0].extract_text()
```

## Advanced features

**Form filling**: See [FORMS.md](FORMS.md) for complete guide
**API reference**: See [REFERENCE.md](REFERENCE.md) for all methods
**Examples**: See [EXAMPLES.md](EXAMPLES.md) for common patterns
`````

Agent 仅在需要时加载 FORMS.md、REFERENCE.md 或 EXAMPLES.md。

#### 模式 2：按领域组织

对于包含多个领域的 Skill，按领域组织内容以避免加载不相关上下文。当用户询问销售指标时，Agent 只需要阅读销售相关 schema，而不是财务或营销数据。这样 token 使用更低，上下文更聚焦。

```
bigquery-skill/
├── SKILL.md（概览与导航）
└── reference/
    ├── finance.md（收入、计费指标）
    ├── sales.md（商机、 pipeline）
    ├── product.md（API 用量、功能）
    └── marketing.md（活动、归因）
```

````markdown SKILL.md theme={null}
# BigQuery Data Analysis

## Available datasets

**Finance**: Revenue, ARR, billing → See [reference/finance.md](reference/finance.md)
**Sales**: Opportunities, pipeline, accounts → See [reference/sales.md](reference/sales.md)
**Product**: API usage, features, adoption → See [reference/product.md](reference/product.md)
**Marketing**: Campaigns, attribution, email → See [reference/marketing.md](reference/marketing.md)

## Quick search

使用 grep 查找特定指标：

```bash
grep -i "revenue" reference/finance.md
grep -i "pipeline" reference/sales.md
grep -i "api usage" reference/product.md
```
`````

#### 模式 3：条件细节

展示基础内容，链接到高级内容：

```markdown  theme={null}
# DOCX Processing

## Creating documents

Use docx-js for new documents. See [DOCX-JS.md](DOCX-JS.md).

## Editing documents

For simple edits, modify the XML directly.

**For tracked changes**: See [REDLINING.md](REDLINING.md)
**For OOXML details**: See [OOXML.md](OOXML.md)
```

Agent 仅在用户需要这些功能时才阅读 REDLINING.md 或 OOXML.md。

### 避免过深嵌套引用

Agent 可能会部分阅读从其他引用文件引用的文件。遇到嵌套引用时，Agent 可能使用 `head -100` 等命令预览内容，而非阅读完整文件，导致信息不完整。

**让引用从 SKILL.md 出发只深入一层**。所有参考文件都应直接从 SKILL.md 链接，确保 Agent 在需要时阅读完整文件。

**坏示例：太深**：

```markdown  theme={null}
# SKILL.md
See [advanced.md](advanced.md)...

# advanced.md
See [details.md](details.md)...

# details.md
Here's the actual information...
```

**好示例：只深一层**：

```markdown  theme={null}
# SKILL.md

**Basic usage**: [instructions in SKILL.md]
**Advanced features**: See [advanced.md](advanced.md)
**API reference**: See [reference.md](reference.md)
**Examples**: See [examples.md](examples.md)
```

### 为较长的参考文件添加目录

对于超过 100 行的参考文件，在顶部包含目录。这确保 Agent 即使在使用部分阅读预览时也能看到完整信息范围。

**示例**：

```markdown  theme={null}
# API Reference

## Contents
- Authentication and setup
- Core methods (create, read, update, delete)
- Advanced features (batch operations, webhooks)
- Error handling patterns
- Code examples

## Authentication and setup
...

## Core methods
...
```

Agent 随后可以阅读完整文件或跳转到特定 section。

关于这种基于文件系统的架构如何实现渐进式披露的更多细节，请参见下方 Advanced 部分的 [Runtime environment](#runtime-environment)。

## 工作流与反馈循环

### 对复杂任务使用工作流

将复杂操作拆分为清晰的顺序步骤。对于特别复杂的工作流，提供 Agent 可以复制到响应中并逐步勾选的清单。

**示例 1：研究综合工作流**（适用于无代码 Skill）：

````markdown  theme={null}
## Research synthesis workflow

复制此清单并跟踪进度：

```
Research Progress:
- [ ] Step 1: Read all source documents
- [ ] Step 2: Identify key themes
- [ ] Step 3: Cross-reference claims
- [ ] Step 4: Create structured summary
- [ ] Step 5: Verify citations
```

**Step 1: Read all source documents**

Review each document in the `sources/` directory. Note the main arguments and supporting evidence.

**Step 2: Identify key themes**

Look for patterns across sources. What themes appear repeatedly? Where do sources agree or disagree?

**Step 3: Cross-reference claims**

For each major claim, verify it appears in the source material. Note which source supports each point.

**Step 4: Create structured summary**

Organize findings by theme. Include:
- Main claim
- Supporting evidence from sources
- Conflicting viewpoints (if any)

**Step 5: Verify citations**

Check that every claim references the correct source document. If citations are incomplete, return to Step 3.
`````

此示例展示了工作流如何应用于不需要代码的分析任务。清单模式适用于任何复杂的多步骤流程。

**示例 2：PDF 表单填写工作流**（适用于有代码 Skill）：

````markdown  theme={null}
## PDF form filling workflow

复制此清单并在完成时勾选：

```
Task Progress:
- [ ] Step 1: Analyze the form (run analyze_form.py)
- [ ] Step 2: Create field mapping (edit fields.json)
- [ ] Step 3: Validate mapping (run validate_fields.py)
- [ ] Step 4: Fill the form (run fill_form.py)
- [ ] Step 5: Verify output (run verify_output.py)
```

**Step 1: Analyze the form**

运行：`python scripts/analyze_form.py input.pdf`

This extracts form fields and their locations, saving to `fields.json`.

**Step 2: Create field mapping**

编辑 `fields.json`，为每个字段添加值。

**Step 3: Validate mapping**

运行：`python scripts/validate_fields.py fields.json`

在继续前修复所有验证错误。

**Step 4: Fill the form**

运行：`python scripts/fill_form.py input.pdf fields.json output.pdf`

**Step 5: Verify output**

运行：`python scripts/verify_output.py output.pdf`

如果验证失败，返回 Step 2。
`````

清晰的步骤防止 Agent 跳过关键验证。清单帮助你与 Agent 跟踪多步骤工作流进度。

### 实现反馈循环

**常见模式**：运行验证器 → 修复错误 → 重复

此模式显著提升输出质量。

**示例 1：风格指南合规**（适用于无代码 Skill）：

```markdown  theme={null}
## Content review process

1. Draft your content following the guidelines in STYLE_GUIDE.md
2. Review against the checklist:
   - Check terminology consistency
   - Verify examples follow the standard format
   - Confirm all required sections are present
3. If issues found:
   - Note each issue with specific section reference
   - Revise the content
   - Review the checklist again
4. Only proceed when all requirements are met
5. Finalize and save the document
```

这展示了使用参考文档而非脚本实现验证循环的模式。“验证器”是 STYLE\_GUIDE.md，Agent 通过阅读并比较来执行检查。

**示例 2：文档编辑流程**（适用于有代码 Skill）：

```markdown  theme={null}
## Document editing process

1. Make your edits to `word/document.xml`
2. **Validate immediately**: `python ooxml/scripts/validate.py unpacked_dir/`
3. If validation fails:
   - Review the error message carefully
   - Fix the issues in the XML
   - Run validation again
4. **Only proceed when validation passes**
5. Rebuild: `python ooxml/scripts/pack.py unpacked_dir/ output.docx`
6. Test the output document
```

验证循环尽早捕获错误。

## 内容指南

### 避免时间敏感信息

不要包含会过时的信息：

**坏示例：时间敏感**（终将出错）：

```markdown  theme={null}
If you're doing this before August 2025, use the old API.
After August 2025, use the new API.
```

**好示例**（使用“旧模式”部分）：

```markdown  theme={null}
## Current method

Use the v2 API endpoint: `api.example.com/v2/messages`

## Old patterns

<details>
<summary>Legacy v1 API (deprecated 2025-08)</summary>

The v1 API used: `api.example.com/v1/messages`

This endpoint is no longer supported.
</details>
```

Old patterns 部分提供历史上下文，而不让主要内容显得杂乱。

### 使用一致的术语

选择一个术语并在整个 Skill 中一致使用：

**好 - 一致**：

* 始终使用 “API endpoint”
* 始终使用 “field”
* 始终使用 “extract”

**坏 - 不一致**：

* 混用 “API endpoint”、“URL”、“API route”、“path”
* 混用 “field”、“box”、“element”、“control”
* 混用 “extract”、“pull”、“get”、“retrieve”

一致性帮助 Agent 理解和遵循指令。

## 常见模式

### 模板模式

为输出格式提供模板。根据需求匹配严格程度。

**严格要求的输出**（如 API 响应或数据格式）：

````markdown  theme={null}
## Report structure

ALWAYS use this exact template structure:

```markdown
# [Analysis Title]

## Executive summary
[One-paragraph overview of key findings]

## Key findings
- Finding 1 with supporting data
- Finding 2 with supporting data
- Finding 3 with supporting data

## Recommendations
1. Specific actionable recommendation
2. Specific actionable recommendation
```
`````

**灵活指导**（当适应性有用时）：

````markdown  theme={null}
## Report structure

Here is a sensible default format, but use your best judgment based on the analysis:

```markdown
# [Analysis Title]

## Executive summary
[Overview]

## Key findings
[Adapt sections based on what you discover]

## Recommendations
[Tailor to the specific context]
```

根据具体分析类型按需调整 section。
`````

### 示例模式

对于输出质量依赖示例的 Skill，像常规提示一样提供输入/输出对：

````markdown  theme={null}
## Commit message format

按照以下示例生成提交信息：

**示例 1：**
Input: Added user authentication with JWT tokens
Output:
```
feat(auth): implement JWT-based authentication

Add login endpoint and token validation middleware
```

**示例 2：**
Input: Fixed bug where dates displayed incorrectly in reports
Output:
```
fix(reports): correct date formatting in timezone conversion

Use UTC timestamps consistently across report generation
```

**示例 3：**
Input: Updated dependencies and refactored error handling
Output:
```
chore: update dependencies and refactor error handling

- Upgrade lodash to 4.17.21
- Standardize error response format across endpoints
```

遵循此风格：type(scope): brief description，然后是详细说明。
`````

示例比单独描述更清楚地向 Agent 传达期望风格和详细程度。

### 条件工作流模式

引导 Agent 通过决策点：

```markdown  theme={null}
## Document modification workflow

1. Determine the modification type:

   **Creating new content?** → Follow "Creation workflow" below
   **Editing existing content?** → Follow "Editing workflow" below

2. Creation workflow:
   - Use docx-js library
   - Build document from scratch
   - Export to .docx format

3. Editing workflow:
   - Unpack existing document
   - Modify XML directly
   - Validate after each change
   - Repack when complete
```

<Tip>
  如果工作流变得庞大或复杂，步骤众多，考虑将它们推入单独的文件，并告诉 Agent 根据当前任务阅读相应文件。
</Tip>

## 评估与迭代

### 先构建评估

**在编写大量文档之前先创建评估。** 这确保你的 Skill 解决真实问题，而不是记录想象的问题。

**评估驱动开发：**

1. **识别缺口**：在没有 Skill 的情况下让 Agent 运行代表性任务。记录具体失败或缺失的上下文
2. **创建评估**：构建三个测试这些缺口的场景
3. **建立基线**：衡量 Agent 在没有 Skill 时的表现
4. **编写最小指令**：创建刚好足以弥补缺口并通过评估的内容
5. **迭代**：执行评估，与基线比较，并 refine

此方法确保你在解决实际问题，而非预测可能永远不会出现的需求。

**评估结构**：

```json  theme={null}
{
  "skills": ["pdf-processing"],
  "query": "Extract all text from this PDF file and save it to output.txt",
  "files": ["test-files/document.pdf"],
  "expected_behavior": [
    "Successfully reads the PDF file using an appropriate PDF processing library or command-line tool",
    "Extracts text content from all pages in the document without missing any pages",
    "Saves the extracted text to a file named output.txt in a clear, readable format"
  ]
}
```

<Note>
  此示例展示了一个带简单测试评分标准的数据驱动评估。我们目前没有提供内置方式来运行这些评估。用户可以创建自己的评估系统。评估是衡量 Skill 效果的真实来源。
</Note>

### 与 Agent 一起迭代开发 Skill

最高效的 Skill 开发过程是让 Agent 本身参与。与一个实例（“Agent A”）合作创建 Skill，供其他实例（“Agent B”）使用。Agent A 帮助你设计和 refine 指令，Agent B 在真实任务中测试它们。这之所以有效，是因为底层模型既理解如何编写高效的 Agent 指令，也理解 Agent 需要什么信息。

**创建新 Skill：**

1. **在没有 Skill 的情况下完成任务**：使用正常提示与 Agent A 一起解决问题。在过程中，你会自然地提供上下文、解释偏好、分享流程知识。注意你反复提供的信息。

2. **识别可复用模式**：完成任务后，找出你提供的哪些上下文对未来类似任务有用。

   **示例**：如果你做了一个 BigQuery 分析，你可能提供了表名、字段定义、过滤规则（如“始终排除测试账号”）和常用查询模式。

3. **让 Agent A 创建 Skill**：“创建一个 Skill，捕捉我们刚刚使用的 BigQuery 分析模式。包括表 schema、命名约定以及关于过滤测试账号的规则。”

   <Tip>
     现代 Agent 原生理解 Skill 格式和结构。你不需要特殊的 system prompt 或“编写技能”的 skill 来帮忙创建 Skill。直接让 Agent 创建 Skill，它就会生成结构正确的 SKILL.md 内容，包括合适的 frontmatter 和正文。
   </Tip>

4. **检查简洁性**：确认 Agent A 没有添加不必要的解释。询问：“删掉关于胜率含义的解释——Agent 已经知道那个了。”

5. **改进信息架构**：让 Agent A 更有效地组织内容。例如：“把表 schema 放到单独的参考文件中。我们以后可能会添加更多表。”

6. **在类似任务上测试**：让 Agent B（一个加载了 Skill 的新实例）在相关用例上使用该 Skill。观察 Agent B 是否能找到正确信息、正确应用规则、成功完成任务。

7. **基于观察迭代**：如果 Agent B 遇到困难或遗漏了什么，带着具体信息回到 Agent A：“当 Agent 使用这个 Skill 时，它忘了按 Q4 日期过滤。我们应该添加一个关于日期过滤模式的 section 吗？”

**迭代改进现有 Skill：**

改进 Skill 时，同样的层级模式继续适用。你在以下之间交替：

* **与 Agent A 合作**（帮助 refine Skill 的专家）
* **与 Agent B 测试**（使用 Skill 完成真实工作的 Agent）
* **观察 Agent B 的行为** 并将洞察带回 Agent A

1. **在真实工作流中使用 Skill**：给 Agent B（加载了 Skill）真实任务，而非测试场景

2. **观察 Agent B 的行为**：注意它在哪些地方遇到困难、成功或做出意外选择

   **示例观察**：“当我让 Agent B 生成区域销售报告时，它写了查询但忘了过滤测试账号，即使 Skill 提到了这条规则。”

3. **回到 Agent A 改进**：分享当前 SKILL.md 并描述你观察到的现象。询问：“我注意到当我要求区域报告时，Agent B 忘了过滤测试账号。Skill 提到了过滤，但也许不够突出？”

4. **审查 Agent A 的建议**：Agent A 可能会建议重新组织以让规则更突出、使用更强的措辞如 “MUST filter” 而非 “always filter”，或重构工作流部分。

5. **应用并测试更改**：用 Agent A 的改进更新 Skill，然后在类似请求上再次用 Agent B 测试

6. **基于使用重复**：随着遇到新场景，继续这个观察-refine-测试循环。每次迭代都基于真实 Agent 行为而非假设来改进 Skill。

**收集团队反馈：**

1. 与队友分享 Skill 并观察他们的使用
2. 询问：Skill 是否在预期时激活？指令是否清晰？缺少什么？
3. 纳入反馈以弥补你自己使用模式中的盲点

**为什么此方法有效**：Agent A 理解 Agent 的需求，你提供领域专业知识，Agent B 通过真实使用揭示缺口，迭代优化基于观察到的行为而非假设来改进 Skill。

### 观察 Agent 如何浏览 Skill

迭代 Skill 时，注意 Agent 在实践中实际如何使用它们。留意：

* **意外的浏览路径**：Agent 是否以你未预料的顺序阅读文件？这可能表明你的结构没有你想象的那么直观
* **遗漏的关联**：Agent 是否未能遵循指向重要文件的引用？你的链接可能需要更明确或更突出
* **过度依赖某些 section**：如果 Agent 反复阅读同一文件，考虑该内容是否应放到主 SKILL.md 中
* **被忽略的内容**：如果 Agent 从不访问某个捆绑文件，它可能不必要，或在主指令中信号不足

基于这些观察而非假设进行迭代。Skill 元数据中的 `name` 和 `description` 尤其关键。Agent 用它们决定是否对当前任务触发 Skill。确保它们清楚描述 Skill 做什么以及何时使用。

## 应避免的反模式

### 避免 Windows 风格路径

即使在 Windows 上，文件路径也要始终使用正斜杠：

* ✓ **好**：`scripts/helper.py`、`reference/guide.md`
* ✗ **避免**：`scripts\helper.py`、`reference\guide.md`

Unix 风格路径在所有平台都有效，而 Windows 风格路径在 Unix 系统上会导致错误。

### 避免提供过多选项

除非必要，否则不要展示多种方法：

````markdown  theme={null}
**坏示例：选择太多**（令人困惑）：
"You can use pypdf, or pdfplumber, or PyMuPDF, or pdf2image, or..."

**好示例：提供默认值**（带逃生口）：
"Use pdfplumber for text extraction:
```python
import pdfplumber
```

For scanned PDFs requiring OCR, use pdf2image with pytesseract instead."
`````

## 高级：包含可执行代码的 Skill

以下部分专注于包含可执行脚本的 Skill。如果你的 Skill 只使用 markdown 指令，请跳到 [高效 Skill 检查清单](#checklist-for-effective-skills)。

### 解决，不要推诿

为 Skill 编写脚本时，处理错误条件，而不是推给 Agent。

**好示例：显式处理错误**：

```python  theme={null}
def process_file(path):
    """Process a file, creating it if it doesn't exist."""
    try:
        with open(path) as f:
            return f.read()
    except FileNotFoundError:
        # 文件不存在时创建默认内容，而不是失败
        print(f"File {path} not found, creating default")
        with open(path, 'w') as f:
            f.write('')
        return ''
    except PermissionError:
        # 提供替代方案，而不是失败
        print(f"Cannot access {path}, using default")
        return ''
```

**坏示例：推给 Agent**：

```python  theme={null}
def process_file(path):
    # 直接失败，让 Agent 自己解决
    return open(path).read()
```

配置参数也应有正当理由和文档说明，避免“巫毒常量”（Ousterhout 定律）。如果你都不知道正确值，Agent 怎么确定？

**好示例：自解释**：

```python  theme={null}
# HTTP 请求通常在 30 秒内完成
# 更长的超时是为了应对慢连接
REQUEST_TIMEOUT = 30

# 3 次重试在可靠性与速度之间取得平衡
# 大多数间歇性失败会在第二次重试时解决
MAX_RETRIES = 3
```

**坏示例：魔术数字**：

```python  theme={null}
TIMEOUT = 47  # 为什么是 47？
RETRIES = 5   # 为什么是 5？
```

### 提供实用脚本

即使 Agent 能自己写脚本，预制脚本也有优势：

**实用脚本的好处**：

* 比生成的代码更可靠
* 节省 token（无需在上下文中包含代码）
* 节省时间（无需代码生成）
* 确保各次使用的一致性

<img src="https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-executable-scripts.png?fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=4bbc45f2c2e0bee9f2f0d5da669bad00" alt="Bundling executable scripts alongside instruction files" data-og-width="2048" width="2048" data-og-height="1154" height="1154" data-path="images/agent-skills-executable-scripts.png" data-optimize="true" data-opv="3" srcset="https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-executable-scripts.png?w=280&fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=9a04e6535a8467bfeea492e517de389f 280w, https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-executable-scripts.png?w=560&fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=e49333ad90141af17c0d7651cca7216b 560w, https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-executable-scripts.png?w=840&fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=954265a5df52223d6572b6214168c428 840w, https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-executable-scripts.png?w=1100&fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=2ff7a2d8f2a83ee8af132b29f10150fd 1100w, https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-executable-scripts.png?w=1650&fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=48ab96245e04077f4d15e9170e081cfb 1650w, https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-executable-scripts.png?w=2500&fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=0301a6c8b3ee879497cc5b5483177c90 2500w" />

上图展示了可执行脚本如何与指令文件协同工作。指令文件（forms.md）引用脚本，Agent 可以在不将脚本内容加载到上下文的情况下执行它。

**重要区别**：在指令中明确 Agent 应该：

* **执行脚本**（最常见）："Run `analyze_form.py` to extract fields"
* **作为参考阅读**（复杂逻辑）："See `analyze_form.py` for the field extraction algorithm"

对于大多数实用脚本，执行更受青睐，因为它更可靠高效。关于脚本执行工作原理的更多细节，请参见下方的 [Runtime environment](#runtime-environment) 部分。

**示例**：

````markdown  theme={null}
## Utility scripts

**analyze_form.py**: Extract all form fields from PDF

```bash
python scripts/analyze_form.py input.pdf > fields.json
```

Output format:
```json
{
  "field_name": {"type": "text", "x": 100, "y": 200},
  "signature": {"type": "sig", "x": 150, "y": 500}
}
```

**validate_boxes.py**: Check for overlapping bounding boxes

```bash
python scripts/validate_boxes.py fields.json
# Returns: "OK" or lists conflicts
```

**fill_form.py**: Apply field values to PDF

```bash
python scripts/fill_form.py input.pdf fields.json output.pdf
```
`````

### 使用视觉分析

当输入可以渲染为图像时，让 Agent 分析它们：

````markdown  theme={null}
## Form layout analysis

1. Convert PDF to images:
   ```bash
   python scripts/pdf_to_images.py form.pdf
   ```

2. Analyze each page image to identify form fields
3. The agent can see field locations and types visually
`````

<Note>
  此示例中，你需要编写 `pdf_to_images.py` 脚本。
</Note>

Agent 的视觉能力有助于理解布局和结构。

### 创建可验证的中间输出

当 Agent 执行复杂、开放式的任务时，它们可能出错。“计划-验证-执行”模式通过让 Agent 首先以结构化格式创建计划，然后在执行前用脚本验证该计划，从而尽早捕获错误。

**示例**：想象你要求 Agent 根据电子表格更新 PDF 中的 50 个表单字段。没有验证的话，它可能引用不存在的字段、创建冲突值、遗漏必填字段或错误应用更新。

**解决方案**：使用上方展示的工作流模式（PDF 表单填写），但添加一个中间 `changes.json` 文件，在应用更改前进行验证。工作流变为：analyze → **create plan file** → **validate plan** → execute → verify。

**为什么这个模式有效：**

* **尽早捕获错误**：验证在应用更改前发现问题
* **机器可验证**：脚本提供客观的验证
* **可逆的计划**：Agent 可以在不碰原始文件的情况下迭代计划
* **清晰的调试**：错误消息指向具体问题

**何时使用**：批量操作、破坏性更改、复杂验证规则、高风险操作。

**实现提示**：让验证脚本输出详细的错误消息，例如 “Field 'signature\_date' not found. Available fields: customer\_name, order\_total, signature\_date\_signed”，以帮助 Agent 修复问题。

### 包依赖

Skill 在代码执行环境中运行，受平台特定限制：

* **claude.ai**：可以从 npm 和 PyPI 安装包，并从 GitHub 仓库拉取
* **Anthropic API**：没有网络访问，也没有运行时包安装

在 SKILL.md 中列出所需包，并在 [code execution tool documentation](https://platform.claude.com/docs/en/agents-and-tools/tool-use/code-execution-tool) 中验证它们是否可用。

### 运行时环境

Skill 在具有文件系统访问、bash 命令和代码执行能力的代码执行环境中运行。关于此架构的概念解释，请参见 overview 中的 [The Skills architecture](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview#the-skills-architecture)。

**这对你的编写有何影响：**

**Agent 如何访问 Skill：**

1. **元数据预加载**：启动时，所有 Skill 的 YAML frontmatter 中的 name 和 description 都会加载到系统提示词中
2. **文件按需读取**：Agent 使用文件读取工具在需要时从文件系统访问 SKILL.md 和其他文件
3. **脚本高效执行**：实用脚本可以通过 bash 执行，而无需将其完整内容加载到上下文中。只有脚本输出消耗 token
4. **大文件没有上下文惩罚**：参考文件、数据或文档在实际读取前不消耗上下文 token

* **文件路径很重要**：Agent 像浏览文件系统一样浏览你的 skill 目录。使用正斜杠（`reference/guide.md`），而非反斜杠
* **文件名应具有描述性**：使用能表明内容的名称：`form_validation_rules.md`，而不是 `doc2.md`
* **为发现而组织**：按领域或功能组织目录
  * 好：`reference/finance.md`、`reference/sales.md`
  * 坏：`docs/file1.md`、`docs/file2.md`
* **捆绑综合资源**：包含完整的 API 文档、大量示例、大型数据集；在访问前没有上下文惩罚
* **对确定性操作优先使用脚本**：编写 `validate_form.py`，而不是让 Agent 生成验证代码
* **明确执行意图**：
  * “Run `analyze_form.py` to extract fields”（执行）
  * “See `analyze_form.py` for the extraction algorithm”（作为参考阅读）
* **测试文件访问模式**：通过真实请求验证 Agent 能否浏览你的目录结构

**示例：**

```
bigquery-skill/
├── SKILL.md（概览，指向参考文件）
└── reference/
    ├── finance.md（收入指标）
    ├── sales.md（pipeline 数据）
    └── product.md（使用分析）
```

当用户询问收入时，Agent 读取 SKILL.md，看到指向 `reference/finance.md` 的引用，然后调用 bash 只读取该文件。sales.md 和 product.md 文件保留在文件系统中，在需要前消耗零上下文 token。这种基于文件系统的模型实现了渐进式披露。Agent 可以导航并选择性地加载每个任务所需的确切内容。

关于技术架构的完整细节，请参见 Skills overview 中的 [How Skills work](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview#how-skills-work)。

### MCP 工具引用

如果你的 Skill 使用 MCP（Model Context Protocol）工具，始终使用完全限定工具名，以避免 “tool not found” 错误。

**格式**：`ServerName:tool_name`

**示例**：

```markdown  theme={null}
Use the BigQuery:bigquery_schema tool to retrieve table schemas.
Use the GitHub:create_issue tool to create issues.
```

其中：

* `BigQuery` 和 `GitHub` 是 MCP 服务器名称
* `bigquery_schema` 和 `create_issue` 是这些服务器中的工具名称

没有服务器前缀，Agent 可能无法定位工具，尤其是在有多个 MCP 服务器可用时。

### 不要假设工具已安装

不要假设包可用：

````markdown  theme={null}
**坏示例：假设已安装**：
"Use the pdf library to process the file."

**好示例：明确依赖**：
"Install required package: `pip install pypdf`

Then use it:
```python
from pypdf import PdfReader
reader = PdfReader("file.pdf")
```"
`````

## 技术说明

### YAML frontmatter 要求

SKILL.md frontmatter 需要 `name`（最多 64 个字符）和 `description`（最多 1024 个字符）字段。完整结构细节请参见 [Skills overview](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview#skill-structure)。

### Token 预算

为获得最佳性能，SKILL.md 正文控制在 500 行以内。如果内容超过此限制，使用前面描述的渐进式披露模式将其拆分到单独文件中。架构细节请参见 [Skills overview](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview#how-skills-work)。

## 高效 Skill 检查清单

在分享 Skill 前，验证：

### 核心质量

* [ ] Description 具体且包含关键词
* [ ] Description 同时包含 Skill 做什么以及何时使用
* [ ] SKILL.md 正文在 500 行以内
* [ ] 额外细节放在单独文件中（如需要）
* [ ] 没有时间敏感信息（或在“旧模式”部分）
* [ ] 全篇术语一致
* [ ] 示例具体，不抽象
* [ ] 文件引用只深一层
* [ ] 适当使用渐进式披露
* [ ] 工作流步骤清晰

### 代码与脚本

* [ ] 脚本解决问题，而不是推给 Agent
* [ ] 错误处理显式且有用
* [ ] 没有“巫毒常量”（所有值都有正当理由）
* [ ] 所需包已在指令中列出并验证可用
* [ ] 脚本文档清晰
* [ ] 没有 Windows 风格路径（全部正斜杠）
* [ ] 关键操作包含验证/确认步骤
* [ ] 质量关键任务包含反馈循环

### 测试

* [ ] 至少创建了三个评估
* [ ] 在 Haiku、Sonnet 和 Opus 上测试过
* [ ] 在真实使用场景中测试过
* [ ] 纳入了团队反馈（如适用）

## 下一步

<CardGroup cols={2}>
  <Card title="Get started with Agent Skills" icon="rocket" href="https://platform.claude.com/docs/en/agents-and-tools/agent-skills/quickstart">
    创建你的第一个 Skill
  </Card>

  <Card title="Use Skills in Claude Code" icon="terminal" href="https://code.claude.com/docs/en/skills">
    在 Claude Code 中创建和管理 Skill
  </Card>

  <Card title="Use Skills with the API" icon="code" href="https://platform.claude.com/docs/en/build-with-claude/skills-guide">
    以编程方式上传和使用 Skill
  </Card>
</CardGroup>
