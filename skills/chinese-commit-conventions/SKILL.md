---
name: chinese-commit-conventions
description: 中文 commit 与 changelog 配置参考——Conventional Commits 中文适配、commitlint/husky/commitizen 中文模板、conventional-changelog 中文配置。仅在用户显式 /chinese-commit-conventions 时调用，不要根据上下文自动触发。
---

# 中文 Git 提交规范

## 1. Conventional Commits 中文适配

基于 Conventional Commits 1.0.0 规范，针对中文团队的实际使用习惯进行适配。

### 类型（type）定义

| 类型       | 说明                         | 示例场景                   |
| ---------- | ---------------------------- | -------------------------- |
| `feat`     | 新功能                       | 添加用户注册模块           |
| `fix`      | 修复缺陷                     | 修复登录页白屏问题         |
| `docs`     | 文档变更                     | 更新 API 接口文档          |
| `style`    | 代码格式（不影响逻辑）       | 调整缩进、补充分号         |
| `refactor` | 重构（非新功能、非修复）     | 拆分过长的服务类           |
| `perf`     | 性能优化                     | 优化首页列表查询速度       |
| `test`     | 测试相关                     | 补充用户模块单元测试       |
| `chore`    | 构建/工具/依赖变更           | 升级 webpack 到 v5         |
| `ci`       | 持续集成配置                 | 修改 GitHub Actions 流程   |
| `revert`   | 回滚提交                     | 回滚 v2.1.0 的登录重构     |

### 原则

- type 保留英文关键字（工具链兼容性好）
- scope 和 description 使用中文
- body 使用中文完整描述

## 2. 中文 commit message 模板

```
<type>(<scope>): <subject>

<body>

<footer>
```

### 完整示例

```
feat(用户模块): 添加手机号一键登录功能

- 接入运营商一键登录 SDK
- 支持移动、联通、电信三网
- 登录失败自动降级到短信验证码

Closes #128
```

## 3. Subject 行规范

### 格式

```
<type>(<scope>): <description>
```

### 规则

- **type**: 必填，从上方类型表中选取
- **scope**: 选填，表示影响范围，使用中文模块名（如 `用户模块`、`订单`、`支付`）
- **description**: 必填，中文简述，不超过 50 个字符
  - 使用动宾短语：「添加 xxx」「修复 xxx」「优化 xxx」
  - 不加句号结尾
  - 不要写「修改了代码」这种无意义描述

### 示例

```
feat(权限): 添加基于 RBAC 的细粒度权限控制
fix(支付): 修复微信支付回调签名验证失败的问题
perf(列表页): 优化大数据量表格的虚拟滚动渲染
```

### 反面示例

```
fix: 修了一个 bug
feat: 更新代码
chore: 改了点东西
```

## 4. Body 编写规范

- 说明**为什么**要做这个改动（背景/原因）
- 说明**怎么做**的（技术方案摘要）
- 说明**影响范围**（哪些模块、接口受影响）
- 每行不超过 72 个字符（中文约 36 个汉字）
- 正文与标题之间空一行

**Body 模板：**
```
<改动背景和原因>

技术方案：
- <方案要点 1>
- <方案要点 2>

影响范围：<受影响的模块或服务>
```

## 5. Breaking Changes 标注

当提交包含不兼容变更时，必须在 footer 中标注。

```
feat(接口): 重构用户信息返回结构

将用户接口返回的扁平结构改为嵌套结构，前端需同步调整字段取值路径。

BREAKING CHANGE: /api/user/info 返回结构变更
- avatar 字段移入 profile 对象
- 移除已废弃的 nickname 字段，统一使用 displayName
```

也可用简写：`feat(接口)!: 重构用户信息返回结构`

**必须标注的情况：** 数据库表结构变更、公共 API 参数/返回值变更、配置文件格式变更。

## 6. Issue 关联

```
# GitHub
Closes #128
Refs #129, #130

# Gitee
Closes #I5ABC1

# Coding
关联 Coding 缺陷 #12345

# 多平台混合
Closes #128
Jira: PROJ-456
禅道: #789
```

## 7. Changelog 自动生成配置

```bash
npm install -D conventional-changelog-cli conventional-changelog-conventionalcommits
```

**package.json 脚本：**
```json
{
  "scripts": {
    "changelog": "conventional-changelog -p conventionalcommits -i CHANGELOG.md -s",
    "changelog:all": "conventional-changelog -p conventionalcommits -i CHANGELOG.md -s -r 0"
  }
}
```

**.versionrc.js 中文配置：**
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

## 8. commitlint 中文配置

```bash
npm install -D @commitlint/cli @commitlint/config-conventional
```

**commitlint.config.js：**
```javascript
module.exports = {
  extends: ['@commitlint/config-conventional'],
  rules: {
    'type-enum': [2, 'always', [
      'feat', 'fix', 'docs', 'style', 'refactor',
      'perf', 'test', 'chore', 'ci', 'revert'
    ]],
    'type-case': [2, 'always', 'lower-case'],
    'type-empty': [2, 'never'],
    'subject-empty': [2, 'never'],
    'subject-max-length': [2, 'always', 100],
    'subject-case': [0],  // 允许中文字符
    'header-max-length': [2, 'always', 120],
    'body-max-line-length': [1, 'always', 200],
    'footer-max-line-length': [1, 'always', 200]
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

## 9. husky + lint-staged 集成

```bash
npm install -D husky lint-staged
npx husky init
```

**.husky/commit-msg**
```bash
npx --no -- commitlint --edit "$1"
```

**.husky/pre-commit**
```bash
npx lint-staged
```

**package.json 中 lint-staged 配置：**
```json
{
  "lint-staged": {
    "*.{js,ts,jsx,tsx,vue}": ["eslint --fix", "prettier --write"],
    "*.{css,scss,less}": ["stylelint --fix", "prettier --write"],
    "*.md": ["prettier --write"]
  }
}
```

## 10. 团队规范检查清单

### 提交前自查

- [ ] type 是否正确选择（feat/fix/docs/...）
- [ ] scope 是否准确描述了影响模块
- [ ] subject 是否为动宾短语且不超过 50 字符
- [ ] subject 末尾是否去掉了句号
- [ ] body 是否说明了变更原因和方案
- [ ] 不兼容变更是否标注了 BREAKING CHANGE
- [ ] 相关 Issue 是否已关联
- [ ] 一次提交是否只做了一件事（原子性）

### 团队落地步骤

1. **工具链配置**：配置 commitlint + husky，让规范可执行
2. **模板共享**：将 `.commitlintrc`、`.husky/` 等配置提交到仓库
3. **团队培训**：组织 15 分钟规范说明会
4. **Code Review**：Review 时关注 commit message 质量
5. **持续迭代**：每季度回顾规范执行情况，根据反馈调整
