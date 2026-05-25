---
name: chinese-git-workflow
description: Use when configuring domestic Git platforms — Gitee, Coding.net, Jihu GitLab, CNB — covering SSH/HTTPS/credentials/CI differences and mirror sync. Only invoke when user explicitly requests.
---

# 国内 Git 工作流规范

## 概述

适配国内平台和团队习惯的 Git 工作流。**核心原则：** 工作流服务团队效率，选适合团队规模的，不硬套大厂方案。

## 国内 Git 平台适配

### 平台对比

| 特性 | Gitee | Coding.net | 极狐 GitLab | CNB | GitHub |
|------|-------|------------|-------------|-----|--------|
| 国内访问 | 快 | 快 | 快 | 快 | 不稳定 |
| CI/CD | Gitee Go | Coding CI | GitLab CI | .cnb.yml | Actions |
| 代码审查 | PR | MR | MR | MR | PR |
| 适合场景 | 开源/小团队 | 中大型团队 | 企业私有化 | 云原生 | 国际项目 |

### 各平台配置

**Gitee：** `git remote add origin https://gitee.com/<org>/<repo>.git`。SSH: `~/.ssh/config` 配置 `Host gitee.com`。双推：`git remote set-url --add --push origin https://github.com/<org>/<repo>.git`。

**Coding.net：** `git remote add origin https://e.coding.net/<team>/<project>/<repo>.git`。

**极狐 GitLab：** `git remote add origin https://jihulab.com/<group>/<repo>.git`。

**CNB（仅 HTTPS）：** `git remote add origin https://cnb.cool/<org>/<repo>.git`。用户名固定 `cnb`，密码为个人访问令牌。

## 工作流选择

### 方案一：主干开发（Trunk-Based）

适合小团队（2-8 人）、迭代快、有自动化测试。`main` 保持可发布，功能分支 1-2 天内合回，用 Feature Flag 控制未完成功能。

### 方案二：Git Flow

适合中大团队、固定版本节奏、需维护多版本。`main`（生产）+ `develop`（开发）+ `release/*` + `hotfix/*`。

### 方案三：国内简化流程（推荐）

适合大多数国内中小团队：
- `main` — 生产（受保护，仅 PR/MR）
- `dev` — 测试环境（自动部署）
- `feat/x` — 从 dev 拉出，合回 dev
- dev 通过后合并到 main 发布

## 分支命名规范

全部小写，`-` 连接。前缀明确类型：
```
feat/user-login              # 新功能
feat/JIRA-1234-order-refund  # 关联任务编号
fix/payment-callback         # Bug 修复
release/v2.1.0               # 版本发布
hotfix/v2.0.1                # 紧急修复
```

## CI/CD 平台示例

**Gitee Go（.gitee/pipelines/pipeline.yml）：**
```yaml
name: 构建与测试
triggers:
  push:
    branches:
      include: [main, dev]
stages:
  - name: 测试
    jobs:
      - name: 单元测试
        steps:
          - step: npmbuild@1
            inputs:
              nodeVersion: 20
              commands:
                - npm ci
                - npm test
```

**极狐 GitLab CI（.gitlab-ci.yml）：**
```yaml
stages: [test, build, deploy]
variables:
  NPM_REGISTRY: https://registry.npmmirror.com

单元测试:
  stage: test
  script:
    - npm config set registry $NPM_REGISTRY
    - npm ci && npm test

部署生产:
  stage: deploy
  script: [./scripts/deploy-production.sh]
  only: [main]
  when: manual
```

**CNB（.cnb.yml — branch-first 结构）：**
```yaml
main:
  push:
    - docker:
        image: node:20
      stages:
        - npm ci
        - npm test
        - npm run build
  pull_request:
    - docker:
        image: node:20
      stages:
        - npm run lint
        - npm test
```

## PR/MR 描述模板

```markdown
## 变更说明
<!-- 简要描述做了什么、解决什么问题 -->

## 变更类型
- [ ] 新功能（feat）
- [ ] Bug 修复（fix）
- [ ] 重构（refactor）

## 关联信息
- 需求/Bug 链接：
- 设计文档：

## 测试情况
- [ ] 单元测试通过
- [ ] 手动测试通过

## 部署注意事项
- [ ] 需要执行数据库迁移
- [ ] 无特殊注意事项
```

## 常用 Git 配置

```bash
git config --global user.name "张三"
git config --global user.email "zhangsan@company.com"
git config --global core.quotepath false       # 中文文件名正常显示
git config --global init.defaultBranch main
git config --global http.https://github.com.proxy socks5://127.0.0.1:7890  # GitHub 代理
npm config set registry https://registry.npmmirror.com  # NPM 国内镜像
```

.gitignore 国内常见追加：`.coding/`、`.gitee/`。

## 检查清单

- [ ] 分支命名符合团队规范
- [ ] commit message 格式正确
- [ ] 关联了需求/Bug 编号
- [ ] PR/MR 描述填写完整
- [ ] CI 流水线通过
- [ ] 已请求相关同事 Review
