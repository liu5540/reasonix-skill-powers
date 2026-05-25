---
name: chinese-commit-conventions
description: Use when setting up commit conventions for Chinese-speaking teams — Conventional Commits adaptation, commitlint/husky/commitizen Chinese templates, conventional-changelog config. Only invoke when user explicitly requests.
---

# 中文 Git 提交规范

## 类型（type）定义

| 类型 | 说明 | 示例 |
|------|------|------|
| `feat` | 新功能 | 添加用户注册模块 |
| `fix` | 修复缺陷 | 修复登录页白屏问题 |
| `docs` | 文档变更 | 更新 API 接口文档 |
| `style` | 代码格式（不影响逻辑） | 调整缩进、补充分号 |
| `refactor` | 重构 | 拆分过长的服务类 |
| `perf` | 性能优化 | 优化首页列表查询速度 |
| `test` | 测试相关 | 补充用户模块单元测试 |
| `chore` | 构建/工具/依赖变更 | 升级 webpack 到 v5 |
| `ci` | 持续集成配置 | 修改 GitHub Actions 流程 |
| `revert` | 回滚提交 | 回滚 v2.1.0 的登录重构 |

原则：type 保留英文（工具链兼容），scope 和 description 用中文，body 用中文完整描述。

## 中文 commit message 模板

```
<type>(<scope>): <subject>

<body>

<footer>
```

完整示例：
```
feat(用户模块): 添加手机号一键登录功能

- 接入运营商一键登录 SDK
- 支持移动、联通、电信三网
- 登录失败自动降级到短信验证码

Closes #128
```

## Subject 行规范

格式：`<type>(<scope>): <description>`

规则：
- type 必填；scope 选填，使用中文模块名（如 `用户模块`、`订单`、`支付`）
- description 必填，中文动宾短语，不超过 50 字符，不加句号
- 不要写"修改了代码"这种无意义描述

示例：
```
feat(权限): 添加基于 RBAC 的细粒度权限控制
fix(支付): 修复微信支付回调签名验证失败的问题
perf(列表页): 优化大数据量表格的虚拟滚动渲染
```

反面：`fix: 修了一个 bug` / `feat: 更新代码` / `chore: 改了点东西`

## Body 编写规范

说明为什么做（背景/原因）、怎么做（技术方案摘要）、影响范围（哪些模块/接口）。每行不超过 72 字符（中文约 36 汉字），正文与标题间空一行。

## Breaking Changes 标注

不兼容变更必须在 footer 标注：
```
feat(接口): 重构用户信息返回结构

将用户接口返回的扁平结构改为嵌套结构，前端需同步调整。

BREAKING CHANGE: /api/user/info 返回结构变更
- avatar 字段移入 profile 对象
- 移除已废弃的 nickname 字段，统一使用 displayName
```

简写：`feat(接口)!: 重构用户信息返回结构`。必须标注的情况：数据库表结构变更、公共 API 参数/返回值变更、配置文件格式变更。

## Issue 关联

GitHub: `Closes #128`、`Refs #129`。Gitee: `Closes #I5ABC1`。Coding: `关联 Coding 缺陷 #12345`。多平台：可混合使用。

## Changelog 自动生成

```bash
npm install -D conventional-changelog-cli conventional-changelog-conventionalcommits
```

.versionrc.js：
```javascript
module.exports = {
  types: [
    { type: 'feat', section: '新功能' },
    { type: 'fix', section: '缺陷修复' },
    { type: 'perf', section: '性能优化' },
    { type: 'refactor', section: '代码重构' },
    { type: 'docs', section: '文档更新' },
    { type: 'test', section: '测试' },
    { type: 'chore', section: '构建/工具', hidden: true },
    { type: 'ci', section: '持续集成', hidden: true },
    { type: 'style', section: '代码格式', hidden: true }
  ]
}
```

## commitlint 中文配置

```bash
npm install -D @commitlint/cli @commitlint/config-conventional
```

commitlint.config.js：
```javascript
module.exports = {
  extends: ['@commitlint/config-conventional'],
  rules: {
    'type-enum': [2, 'always', ['feat','fix','docs','style','refactor','perf','test','chore','ci','revert']],
    'type-case': [2, 'always', 'lower-case'],
    'subject-empty': [2, 'never'],
    'subject-max-length': [2, 'always', 100],
    'subject-case': [0],  // 允许中文
    'header-max-length': [2, 'always', 120]
  },
  prompt: {
    messages: {
      type: '选择提交类型:',
      scope: '输入影响范围（可选）:',
      subject: '填写简短描述:',
      body: '填写详细描述（可选，使用 "|" 换行）:',
      breaking: '列出不兼容变更（可选）:',
      footer: '关联的 Issue（可选，例如 #123）:',
      confirmCommit: '确认提交以上信息？'
    }
  }
}
```

## husky + lint-staged 集成

```bash
npm install -D husky lint-staged
npx husky init
```

.husky/commit-msg: `npx --no -- commitlint --edit "$1"`
.husky/pre-commit: `npx lint-staged`

lint-staged 配置（package.json）：
```json
{
  "lint-staged": {
    "*.{js,ts,jsx,tsx,vue}": ["eslint --fix", "prettier --write"],
    "*.{css,scss,less}": ["stylelint --fix", "prettier --write"],
    "*.md": ["prettier --write"]
  }
}
```

## 提交前自查

- [ ] type 正确选择
- [ ] scope 准确描述影响模块
- [ ] subject 为动宾短语且不超过 50 字符，末尾无句号
- [ ] body 说明了变更原因和方案
- [ ] 不兼容变更标注了 BREAKING CHANGE
- [ ] 相关 Issue 已关联
- [ ] 一次提交只做一件事（原子性）
