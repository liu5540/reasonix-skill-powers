# AGENTS.md

## 仓库性质

这是一个 **Reasonix 技能仓库**（reasonix-skill-powers），**不是代码项目**。没有构建系统、没有测试套件、没有包管理器、没有 CI。

所有内容使用 **中文**。

## 目录结构

```
skills/
  <skill-name>/
    SKILL.md              # 主文档（必需）
    supporting-file.*     # 仅在需要时（参考内容 >100 行、可复用脚本）
```

## 编写技能的强制规范

编写或编辑任何技能前，必须先加载 `skills/writing-skills/SKILL.md`。

### YAML Frontmatter

```yaml
---
name: skill-name-with-hyphens    # 只使用字母、数字、连字符
description: |
  以第三人称"Use when..."开头，描述**触发条件和症状**。
  不总结工作流。包含具体场景和搜索关键词。
  尽量控制在 500 字符以内，最多 1024 字符。
runAs: subagent                  # 可选：需要 spawn 子代理时添加
---
```

### 内容约束

- **原则/概念/代码模式**（< 50 行）：内联在 SKILL.md 中
- **大量参考内容**（> 100 行）：分离到支撑文件
- **交叉引用其他技能**：只使用技能名称（如 `test-driven-development`），**绝不使用文件路径**
- **Token 效率**：频繁加载的技能 < 200 词，其他 < 500 词

### 技能测试铁律

**没有失败的测试就不写技能。**

新技能或编辑现有技能时：
1. 先运行压力场景观察智能体失败（红）
2. 编写技能解决具体失败（绿）
3. 堵住新的合理化借口（重构）
4. 重新测试直到无懈可击

**无例外**——不适用于"简单添加"、"只是加一节"、"文档更新"。参见 `skills/writing-skills/testing-skills-with-subagents.md` 了解完整方法。

## 常用参考

- `skills/writing-skills/SKILL.md` — 技能编写规范
- `skills/writing-skills/testing-skills-with-subagents.md` — 技能测试方法
- `skills/test-driven-development/SKILL.md` — 红-绿-重构循环
- `skills/dispatching-parallel-agents/SKILL.md` — 并行子代理调用
- `skills/subagent-driven-development/SKILL.md` — 同会话子代理开发流程
