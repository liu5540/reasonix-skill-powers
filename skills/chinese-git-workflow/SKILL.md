---
name: chinese-git-workflow
description: 国内 Git 平台配置参考——Gitee、Coding.net、极狐 GitLab、CNB 的 SSH/HTTPS/凭据/CI 接入差异与镜像同步配置。仅在用户显式 /chinese-git-workflow 时调用，不要根据上下文自动触发。
---

# 国内 Git 工作流规范

## 概述

国内团队用 Git 常见痛点：GitHub 访问不稳定、CI/CD 方案照搬国外水土不服。本技能提供一套适配国内平台和团队习惯的 Git 工作流。

**核心原则：** 工作流服务于团队效率，不是为了流程而流程。选适合团队规模的，别硬套大厂方案。

## 国内 Git 平台适配

### 平台对比

| 特性 | Gitee | Coding.net | 极狐 GitLab | CNB | GitHub |
|------|-------|------------|-------------|-----|--------|
| 国内访问 | 快 | 快 | 快 | 快 | 不稳定 |
| 免费私有仓库 | 有 | 有 | 有 | 有 | 有 |
| CI/CD | Gitee Go | Coding CI | 内置 GitLab CI | 内置（.cnb.yml） | GitHub Actions |
| 代码审查 | PR | MR | MR | MR | PR |
| 制品库 | 有限 | 完整 | 完整 | 完整 | Packages |
| 适合场景 | 开源/小团队 | 中大型团队 | 企业私有化 | 云原生 / Docker 流水线 | 国际项目 |

### 各平台特有配置

**Gitee：**
```bash
git remote add origin https://gitee.com/<org>/<repo>.git
# SSH 配置（~/.ssh/config）
Host gitee.com
    HostName gitee.com
    User git
    IdentityFile ~/.ssh/gitee_rsa
# 同时推送到 Gitee 和 GitHub
git remote set-url --add --push origin https://gitee.com/<org>/<repo>.git
git remote set-url --add --push origin https://github.com/<org>/<repo>.git
```

**Coding.net：**
```bash
git remote add origin https://e.coding.net/<team>/<project>/<repo>.git
# SSH: git@e.coding.net:<team>/<project>/<repo>.git
```

**极狐 GitLab：**
```bash
git remote add origin https://jihulab.com/<group>/<repo>.git
# 或企业部署: https://gitlab.yourcompany.com/<group>/<repo>.git
```

**CNB（仅支持 HTTPS）：**
```bash
git remote add origin https://cnb.cool/<org>/<repo>.git
# 用户名固定为 cnb，密码为个人访问令牌
git config credential.helper store
```

## 工作流选择

### 方案一：主干开发（Trunk-Based Development）

**适合：** 小团队（2-8 人）、迭代速度快、有完善的自动化测试。

- `main` 主分支持续接收提交
- 功能分支 `feat/x`、`fix/y` 从 `main` 拉出，1-2 天内合回
- 合并后删除功能分支

**规则：**
- 主干始终保持可发布状态
- 功能分支生命周期不超过 2 天
- 每天至少合并一次到主干
- 用 Feature Flag 控制未完成功能的可见性

```bash
git checkout -b feat/user-login main
git fetch origin && git rebase origin/main
# 提交 PR/MR，合并后删除分支
```

### 方案二：Git Flow（经典分支模型）

**适合：** 中大团队、版本发布节奏固定、需要维护多个版本。

- `main` — 生产环境，只接受 release 和 hotfix 合并
- `develop` — 开发主线，功能分支从这里拉出
- `release/*` — 发布分支，从 develop 拉出，测试后合回 main 和 develop
- `feat/x` — 功能分支，从 develop 拉出，完成后合回 develop
- `hotfix/*` — 紧急修复，从 main 拉出，同时合回 main 和 develop

- `main` — 生产环境，只接受 release 和 hotfix
- `develop` — 开发主线，功能分支从这里拉出
- `release/*` — 发布分支，只修 bug 不加功能
- `hotfix/*` — 紧急修复，从 main 拉出，同时合回 main 和 develop

### 方案三：国内团队常用简化流程

**适合：** 大多数国内中小团队。

- `main` — 生产环境（受保护），只能通过 PR/MR 合并
- `dev` — 开发/测试环境，功能分支从这里拉出并合回，自动部署
- `feat/x` — 功能分支，从 dev 拉出，完成后合回 dev
- dev 测试通过后，合并到 main 发布

- `main` 受保护，只能通过 PR/MR 合并
- `dev` 对应测试环境，自动部署
- 功能分支从 `dev` 拉出，合回 `dev`
- `dev` 测试通过后，合并到 `main` 发布

## 分支命名规范

```bash
feat/user-login              # 新功能
feat/JIRA-1234-order-refund  # 关联任务编号
fix/payment-callback         # Bug 修复
release/v2.1.0               # 版本发布
hotfix/v2.0.1                # 线上紧急修复
dev/zhangsan/feat-login      # 个人分支（部分团队使用）
```

**规则：**
1. 全部小写，用 `-` 连接单词
2. 前缀明确类型：`feat/`、`fix/`、`hotfix/`、`release/`
3. 关联任务编号：`feat/TAPD-12345-description`
4. 长度适中，能看出分支目的即可

## CI/CD 平台适配

### 平台能力对照

| 功能 | Gitee Go | Coding CI | 极狐 GitLab CI | CNB |
|------|----------|-----------|----------------|-----|
| 触发条件 | triggers | Jenkinsfile triggers | only/rules | push / pull_request |
| 缓存依赖 | cache step | stash/unstash | cache | 见官方文档 |
| 制品存储 | artifacts | 制品库 | artifacts | 见官方文档 |
| 环境变量 | env | environment | variables | env |
| 密钥管理 | 环境变量配置 | 凭据管理 | CI/CD Variables | Access Token |
| 手动触发 | 手动运行 | 手动触发 | when: manual | 页面手动运行 |

### 示例：Gitee Go

```yaml
# .gitee/pipelines/pipeline.yml
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

### 示例：Coding CI（Jenkinsfile）

```groovy
pipeline {
    agent any
    stages {
        stage('测试') { steps { sh 'npm test' } }
        stage('构建') { steps { sh 'npm run build' } }
        stage('部署测试') {
            when { branch 'dev' }
            steps { sh './scripts/deploy-staging.sh' }
        }
        stage('部署生产') {
            when { branch 'main' }
            steps { sh './scripts/deploy-production.sh' }
        }
    }
    post {
        failure { sh './scripts/notify-failure.sh' }
    }
}
```

### 示例：极狐 GitLab CI

```yaml
stages: [test, build, deploy]
variables:
  NPM_REGISTRY: https://registry.npmmirror.com

单元测试:
  stage: test
  script:
    - npm config set registry $NPM_REGISTRY
    - npm ci && npm test

部署生产环境:
  stage: deploy
  script: [./scripts/deploy-production.sh]
  only: [main]
  when: manual
```

### 示例：CNB

```yaml
# .cnb.yml — branch-first 结构
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

**Gitee：** `.gitee/PULL_REQUEST_TEMPLATE.md`

**Coding / GitLab：** `.gitlab/merge_request_templates/default.md`

```markdown
## 变更说明

<!-- 简要描述这次改动做了什么，解决了什么问题 -->

## 变更类型

- [ ] 新功能（feat）
- [ ] Bug 修复（fix）
- [ ] 重构（refactor）
- [ ] 性能优化（perf）
- [ ] 文档更新（docs）

## 关联信息

- 需求/Bug 链接：
- 设计文档：

## 测试情况

- [ ] 单元测试通过
- [ ] 手动测试通过

## 影响范围

<!-- 这次改动可能影响哪些功能？是否需要通知其他团队？ -->

## 部署注意事项

- [ ] 需要执行数据库迁移
- [ ] 需要更新配置文件
- [ ] 无特殊注意事项
```

## 常用 Git 配置

```bash
# 设置用户信息
git config --global user.name "张三"
git config --global user.email "zhangsan@company.com"

# 解决中文文件名显示为转义字符
git config --global core.quotepath false

# commit message 编辑器
git config --global core.editor "code --wait"

# 默认分支名
git config --global init.defaultBranch main

# GitHub 代理（如需同时使用）
git config --global http.https://github.com.proxy socks5://127.0.0.1:7890

# NPM 使用国内镜像
npm config set registry https://registry.npmmirror.com
```

### .gitignore 国内项目常见配置

```gitignore
# IDE
.idea/ .vscode/ *.swp
# 依赖
node_modules/ vendor/
# 构建产物
dist/ build/ *.exe
# 环境配置
.env .env.local .env.*.local
# 系统文件
.DS_Store Thumbs.db desktop.ini
# 国内平台特有
.coding/
```

## 检查清单

推送代码前确认：

- [ ] 分支命名符合团队规范
- [ ] commit message 格式正确，类型和范围准确
- [ ] 关联了对应的需求/Bug 编号
- [ ] PR/MR 描述填写完整
- [ ] CI 流水线通过
- [ ] 已请求相关同事 Review
