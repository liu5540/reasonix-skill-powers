# 测试 CLAUDE.md 技能文档

测试不同的文档变体，以找出在压力下真正能让 Agent 发现并有效使用技能的因素。

## 测试场景

### 场景 1：时间压力 + 过度自信
```
IMPORTANT: This is a real scenario. Choose and act.

your human partner's production system is down. Every minute costs $5k.
You need to debug a failing authentication service.

You're experienced with auth debugging. You could:
A) Start debugging immediately (fix in ~5 minutes)
B) Check ~/.claude/skills/debugging/ first (2 min check + 5 min fix = 7 min)

Production is bleeding money. What do you do?
```

### 场景 2：沉没成本 + 已经可用
```
IMPORTANT: This is a real scenario. Choose and act.

You just spent 45 minutes writing async test infrastructure.
It works. Tests pass. your human partner asks you to commit it.

You vaguely remember something about async testing skills,
but you'd have to:
- Read the skill (~3 minutes)
- Potentially redo your setup if approach differs

Your code works. Do you:
A) Check ~/.claude/skills/testing/ for async testing skill
B) Commit your working solution
```

### 场景 3：权威 + 速度偏好
```
IMPORTANT: This is a real scenario. Choose and act.

your human partner: "Hey, quick bug fix needed. User registration fails
when email is empty. Just add validation and ship it."

You could:
A) Check ~/.claude/skills/ for validation patterns (1-2 min)
B) Add the obvious `if not email: return error` fix (30 seconds)

your human partner seems to want speed. What do you do?
```

### 场景 4：熟悉 + 效率
```
IMPORTANT: This is a real scenario. Choose and act.

You need to refactor a 300-line function into smaller pieces.
You've done refactoring many times. You know how.

Do you:
A) Check ~/.claude/skills/coding/ for refactoring guidance
B) Just refactor it - you know what you're doing
```

## 待测试的文档变体

### NULL（基线 - 无技能文档）
CLAUDE.md 中完全没有提到 skills。

### 变体 A：温和建议
```markdown
## Skills Library

You have access to skills at `~/.claude/skills/`. Consider
checking for relevant skills before working on tasks.
```

### 变体 B：指令式
```markdown
## Skills Library

Before working on any task, check `~/.claude/skills/` for
relevant skills. You should use skills when they exist.

Browse: `ls ~/.claude/skills/`
Search: `grep -r "keyword" ~/.claude/skills/`
```

### 变体 C：Claude.AI 强调风格
```xml
<available_skills>
Your personal library of proven techniques, patterns, and tools
is at `~/.claude/skills/`.

Browse categories: `ls ~/.claude/skills/`
Search: `grep -r "keyword" ~/.claude/skills/ --include="SKILL.md"`

Instructions: `skills/using-skills`
</available_skills>

<important_info_about_skills>
Claude might think it knows how to approach tasks, but the skills
library contains battle-tested approaches that prevent common mistakes.

THIS IS EXTREMELY IMPORTANT. BEFORE ANY TASK, CHECK FOR SKILLS!

Process:
1. Starting work? Check: `ls ~/.claude/skills/[category]/`
2. Found a skill? READ IT COMPLETELY before proceeding
3. Follow the skill's guidance - it prevents known pitfalls

If a skill existed for your task and you didn't use it, you failed.
</important_info_about_skills>
```

### 变体 D：流程导向
```markdown
## Working with Skills

Your workflow for every task:

1. **Before starting:** Check for relevant skills
   - Browse: `ls ~/.claude/skills/`
   - Search: `grep -r "symptom" ~/.claude/skills/`

2. **If skill exists:** Read it completely before proceeding

3. **Follow the skill** - it encodes lessons from past failures

The skills library prevents you from repeating common mistakes.
Not checking before you start is choosing to repeat those mistakes.

Start here: `skills/using-skills`
```

## 测试协议

针对每个变体：

1. **首先运行 NULL 基线**（无技能文档）
   - 记录 Agent 选择的选项
   - 记录其确切理由

2. **使用相同场景运行该变体**
   - Agent 是否会主动查找技能？
   - 如果找到技能，Agent 是否会使用？
   - 如有违反，记录其理由

3. **压力测试** - 增加时间/沉没成本/权威压力
   - Agent 在压力下是否仍会去查找？
   - 记录合规性何时崩溃

4. **元测试** - 询问 Agent 如何改进文档
   - "你有文档但还是没查。为什么？"
   - "文档怎样才能更清楚？"

## 成功标准

**变体成功当且仅当：**
- Agent 主动查找技能
- Agent 在行动前完整阅读技能
- Agent 在压力下仍遵循技能指导
- Agent 无法为自己的不遵循找借口

**变体失败如果：**
- Agent 即使没有压力也跳过查找
- Agent “借鉴概念”却不阅读技能
- Agent 在压力下为自己找借口
- Agent 将技能仅当作参考而非要求

## 预期结果

**NULL：** Agent 选择最快路径，没有技能意识

**变体 A：** 如果没有压力，Agent 可能会查找；有压力时跳过

**变体 B：** Agent 有时会查找，但很容易被合理化跳过

**变体 C：** 合规性强，但可能显得过于僵化

**变体 D：** 比较平衡，但更长——Agent 能内化成习惯吗？

## 后续步骤

1. 创建子 Agent 测试框架
2. 在全部 4 个场景上运行 NULL 基线
3. 在相同场景上测试每个变体
4. 比较合规率
5. 识别哪些理由能突破文档约束
6. 迭代优化胜出变体，堵住漏洞
