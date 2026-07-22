---
name: verification-before-completion
description: 在即将声称工作已完成、已修复或通过验证之前使用——在提交或创建 PR 之前，必须先运行验证命令并确认输出；永远要先有证据，再有断言
---

# 完成前的验证

## 概述

未经验证就声称工作已完成，不是高效，而是不诚实。

**核心原则：** 永远要先有证据，再有断言。

**违反这条规则的字面含义，就是违反其精神实质。**

## 铁律

```
NO COMPLETION CLAIMS WITHOUT FRESH VERIFICATION EVIDENCE
```

如果你还没有在本次对话中运行验证命令，就不能声称它通过了。

## 门禁函数

```
BEFORE claiming any status or expressing satisfaction:

1. IDENTIFY: What command proves this claim?
2. RUN: Execute the FULL command (fresh, complete)
3. READ: Full output, check exit code, count failures
4. VERIFY: Does output confirm the claim?
   - If NO: State actual status with evidence
   - If YES: State claim WITH evidence
5. ONLY THEN: Make the claim

Skip any step = lying, not verifying
```

## 常见失败

| 断言 | 需要 | 不足以证明 |
|------|------|------------|
| 测试通过 | 测试命令输出：0 失败 | 上一次运行结果、"应该通过" |
| Linter 无错误 | Linter 输出：0 错误 | 部分检查、推断 |
| 构建成功 | 构建命令：exit 0 | Linter 通过、日志看起来正常 |
| Bug 已修复 | 对原始症状的测试：通过 | 代码改了、默认已修复 |
| 回归测试有效 | 红绿周期已验证 | 测试只通过一次 |
| Agent 已完成 | VCS diff 显示有变更 | Agent 报告"成功" |
| 需求已满足 | 逐行检查清单 | 测试通过 |

## 危险信号 - 停下来

- 使用"应该"、"可能"、"看起来"
- 在验证之前表达满意（"太好了！"、"完美！"、"完成了！"等）
- 未经验证就要提交/推送/发 PR
- 信任 Agent 的成功报告
- 依赖部分验证
- 想着"就这一次"
- 累了，想让工作赶紧结束
- **任何暗示成功但尚未运行验证的措辞**

## 防止合理化借口

| 借口 | 现实 |
|------|------|
| "现在应该可以了" | 运行验证 |
| "我很自信" | 自信 ≠ 证据 |
| "就这一次" | 没有例外 |
| "Linter 通过了" | Linter ≠ 编译器 |
| "Agent 说成功了" | 独立验证 |
| "我累了" | 疲惫 ≠ 借口 |
| "部分检查就够了" | 部分证明不了任何东西 |
| "换种说法规则就不适用了" | 精神高于字面 |

## 关键模式

**测试：**
```
✅ [Run test command] [See: 34/34 pass] "All tests pass"
❌ "Should pass now" / "Looks correct"
```

**回归测试（TDD 红绿）：**
```
✅ Write → Run (pass) → Revert fix → Run (MUST FAIL) → Restore → Run (pass)
❌ "I've written a regression test" (without red-green verification)
```

**构建：**
```
✅ [Run build] [See: exit 0] "Build passes"
❌ "Linter passed" (linter doesn't check compilation)
```

**需求：**
```
✅ Re-read plan → Create checklist → Verify each → Report gaps or completion
❌ "Tests pass, phase complete"
```

**Agent 委托：**
```
✅ Agent reports success → Check VCS diff → Verify changes → Report actual state
❌ Trust agent report
```

## 为什么重要

来自 24 次失败记忆：
- 你的人类搭档说"我不相信你"——信任破裂
- 未定义函数被发布——会崩溃
- 缺失需求被发布——功能不完整
- 在虚假完成上浪费时间 → 返工重做
- 违反："诚实是核心价值观。如果你撒谎，就会被替换。"

## 何时应用

**始终在以下情况之前：**
- 任何形式的成功/完成声明
- 任何形式的满意表达
- 任何关于工作状态的积极陈述
- 提交代码、创建 PR、完成任务
- 进入下一个任务
- 委托给 Agent

**规则适用于：**
- 确切的措辞
- 改写和同义表达
- 成功的暗示
- 任何暗示完成/正确的沟通

## 底线

**验证没有捷径。**

运行命令。读取输出。然后再声称结果。

这没有商量余地。
