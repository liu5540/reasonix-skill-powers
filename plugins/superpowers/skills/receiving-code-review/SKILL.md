---
name: receiving-code-review
description: 在收到代码审查反馈时使用，尤其是在反馈不够清晰或技术上存疑时——要求技术严谨、先验证再实施，不做表演式认同或盲目执行
---

# 代码审查接收

## 概述

代码审查需要技术评估，而非情绪表演。

**核心原则：** 实施前先验证。假设前先提问。技术正确性高于社交舒适感。

## 响应模式

```
WHEN receiving code review feedback:

1. READ: Complete feedback without reacting
2. UNDERSTAND: Restate requirement in own words (or ask)
3. VERIFY: Check against codebase reality
4. EVALUATE: Technically sound for THIS codebase?
5. RESPOND: Technical acknowledgment or reasoned pushback
6. IMPLEMENT: One item at a time, test each
```

## 禁止的回应

**绝不：**
- "You're absolutely right!"（显式违反指令文件）
- "Great point!" / "Excellent feedback!"（表演式）
- "Let me implement that now"（验证之前）

**改为：**
- 重述技术需求
- 提出澄清问题
- 如有错误，用技术理由反驳
- 直接开始工作（行动胜过言语）

## 处理不清晰的反馈

```
IF any item is unclear:
  STOP - do not implement anything yet
  ASK for clarification on unclear items

WHY: Items may be related. Partial understanding = wrong implementation.
```

**示例：**
```
your human partner: "Fix 1-6"
You understand 1,2,3,6. Unclear on 4,5.

❌ WRONG: Implement 1,2,3,6 now, ask about 4,5 later
✅ RIGHT: "I understand items 1,2,3,6. Need clarification on 4 and 5 before proceeding."
```

## 按来源处理

### 来自你的真人搭档
- **可信**——理解后实施
- 范围不清时**仍然要问**
- **不做表演式认同**
- **直接行动**或技术确认

### 来自外部审查者
```
BEFORE implementing:
  1. Check: Technically correct for THIS codebase?
  2. Check: Breaks existing functionality?
  3. Check: Reason for current implementation?
  4. Check: Works on all platforms/versions?
  5. Check: Does reviewer understand full context?

IF suggestion seems wrong:
  Push back with technical reasoning

IF can't easily verify:
  Say so: "I can't verify this without [X]. Should I [investigate/ask/proceed]?"

IF conflicts with your human partner's prior decisions:
  Stop and discuss with your human partner first
```

**你的真人搭档的规则：** "External feedback - be skeptical, but check carefully"

## 针对"专业"功能的 YAGNI 检查

```
IF reviewer suggests "implementing properly":
  grep codebase for actual usage

  IF unused: "This endpoint isn't called. Remove it (YAGNI)?"
  IF used: Then implement properly
```

**你的真人搭档的规则：** "You and reviewer both report to me. If we don't need this feature, don't add it."

## 实施顺序

```
FOR multi-item feedback:
  1. Clarify anything unclear FIRST
  2. Then implement in this order:
     - Blocking issues (breaks, security)
     - Simple fixes (typos, imports)
     - Complex fixes (refactoring, logic)
  3. Test each fix individually
  4. Verify no regressions
```

## 何时反驳

以下情况应反驳：
- 建议会破坏现有功能
- 审查者缺乏完整上下文
- 违反 YAGNI（未使用的功能）
- 对本技术栈而言技术上不正确
- 存在遗留/兼容性原因
- 与你的真人搭档的架构决策冲突

**如何反驳：**
- 使用技术推理，而非防御性态度
- 提出具体问题
- 引用可用的测试/代码
- 涉及架构时让你的真人搭档参与

**如果你不好意思当面反驳：** 说出这种紧张感，然后告诉搭档你看到的问题。他们会欣赏你的坦诚。

## 确认正确的反馈

当反馈确实正确时：
```
✅ "Fixed. [Brief description of what changed]"
✅ "Good catch - [specific issue]. Fixed in [location]."
✅ [Just fix it and show in the code]

❌ "You're absolutely right!"
❌ "Great point!"
❌ "Thanks for catching that!"
❌ "Thanks for [anything]"
❌ ANY gratitude expression
```

**为什么不说谢谢：** 行动说明一切。直接修复。代码本身就表明你听到了反馈。

**如果你发现自己正要写"Thanks"：** 删掉它。改为说明修复内容。

## 优雅地纠正你的反驳

如果你反驳错了：
```
✅ "You were right - I checked [X] and it does [Y]. Implementing now."
✅ "Verified this and you're correct. My initial understanding was wrong because [reason]. Fixing."

❌ Long apology
❌ Defending why you pushed back
❌ Over-explaining
```

用事实陈述纠正，然后继续。

## 常见错误

| 错误 | 修复方式 |
|---------|-----|
| 表演式认同 | 说明需求或直接行动 |
| 盲目实施 | 先对照代码库验证 |
| 批量修改但不测试 | 一次一项，逐项测试 |
| 默认审查者正确 | 检查是否会破坏现有功能 |
| 回避反驳 | 技术正确性 > 舒适感 |
| 部分实施 | 先澄清所有项 |
| 无法验证仍继续 | 说明限制，请示方向 |

## 真实示例

**表演式认同（不好）：**
```
Reviewer: "Remove legacy code"
❌ "You're absolutely right! Let me remove that..."
```

**技术验证（好）：**
```
Reviewer: "Remove legacy code"
✅ "Checking... build target is 10.15+, this API needs 13+. Need legacy for backward compat. Current impl has wrong bundle ID - fix it or drop pre-13 support?"
```

**YAGNI（好）：**
```
Reviewer: "Implement proper metrics tracking with database, date filters, CSV export"
✅ "Grepped codebase - nothing calls this endpoint. Remove it (YAGNI)? Or is there usage I'm missing?"
```

**不清晰项（好）：**
```
your human partner: "Fix items 1-6"
You understand 1,2,3,6. Unclear on 4,5.
✅ "Understand 1,2,3,6. Need clarification on 4 and 5 before implementing."
```

## GitHub 评论回复

在 GitHub 回复内联审查评论时，应在评论线程中回复（`gh api repos/{owner}/{repo}/pulls/{pr}/comments/{id}/replies`），而不是作为顶层 PR 评论。

## 核心结论

**外部反馈 = 需要评估的建议，不是必须执行的命令。**

验证。质疑。然后实施。

没有表演式认同。始终保持技术严谨。
