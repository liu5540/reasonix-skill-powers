# reasonix-skill-powers

面向 Reasonix 的技能集合，受 [superpowers-zh](https://github.com/jnMetaCode/superpowers-zh) 启发，简化后适配到 Reasonix 子代理工作流。

## 这是什么

一个纯技能文档仓库，没有代码、没有构建系统、没有依赖。

每个技能都是一组提示词模板和工作流规范，存放在 `skills/<skill-name>/SKILL.md` 中。Reasonix 通过技能名称自动加载对应内容。

## 参考与简化

- **参考：** superpowers-zh（收集 AI 编程的提示词与技巧）
- **简化：** 只保留 SKILL.md 主文件，去掉复杂目录结构；统一用中文；适配 Reasonix 的子代理调用格式（YAML frontmatter + `runAs: subagent`）。

## 目录结构

```
skills/
  <skill-name>/
    SKILL.md              # 主文档（必需）
    supporting-file.*     # 仅在需要时
```

## 使用方式

1. 将本仓库克隆到本地
2. 在项目中使用 ,将skills 复制到 `.reasonix/skills`
3. /skill use-superpowers 加载 `superpowers` 到提示词
4. /skill brainstorming xxxx 时, 包含了 `审核` 之类的提示词会跳过计划编写，直接执行操作

## 技能列表

本仓库包含以下核心技能：

- **use-superpowers** - 执行经典 superpowers 7 步标准工作流
- 更多技能见 `skills/` 目录

### 使用示例

```
# 加载 use-superpowers 技能，执行 7 步标准工作流
/skill use-superpowers
```

## 标准工作流（7 步）

严格遵循 superpowers 官方 7 步流程，每一步对应本仓库技能：

| 步骤 | 技能 | 说明 |
|------|------|------|
| 1. **理解需求** | `brainstorming` | 澄清需求边界和验收标准，不确定的地方必须提问 |
| 2. **探索仓库** | （内置 `explore` 子代理） | 定位相关文件，理解架构和实现模式 |
| 3. **拆分任务** | `brainstorming` | 将大任务拆分为可独立实现的小任务单元 |
| 4. **编写计划** | `writing-plans` | 按依赖顺序列出每个任务的具体实现步骤 |
| 5. **执行实现** | `subagent-driven-development` + `implement` + `test-driven-development` | 使用子代理逐个分派任务，TDD 开发，一次一个不并行 |
| 6. **验证调试** | `verification-before-completion` + `systematic-debugging` + `flow-review` | 运行测试验证功能，系统调试问题，两轮审查（规格合规 + 代码质量） |
| 7. **提交准备** | `finishing-a-development-branch` + `chinese-commit-conventions` | 整理提交信息，准备代码审查 |

### 例外情况

两种小情况无需完整执行 7 步：

1. **单行/文档级修改**：直接修改 + 验证 + 提交，跳过拆分/计划/子代理分派
2. **已有问题调试**：从第 6 步开始直接使用 `systematic-debugging`

## 许可证

MIT License

```
MIT License

Copyright (c) 2025 liu5540

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```
