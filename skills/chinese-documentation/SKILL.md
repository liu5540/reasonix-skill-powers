---
name: chinese-documentation
description: Use when writing Chinese technical documentation or README — Chinese-English spacing, full/half-width punctuation, term handling, link formatting. Only invoke when user explicitly requests.
---

# 中文技术文档写作规范

## 概述

中文技术文档最常见问题不是内容不够，而是读起来别扭——中英文挤在一起无空格、全角半角混用。

**核心原则：** 排版服务于阅读体验，规范服务于一致性。

参考标准：[中文文案排版指北](https://github.com/sparanoid/chinese-copywriting-guidelines)

## 中文排版规范

### 空格

中英文之间加空格（`使用 Git 进行版本管理` ✓，`使用Git进行版本管理` ✗）。中文与数字之间加空格（`包含 3 个新功能`）。数字与单位之间加空格（`5 MB`）。链接前后加空格。

例外：度数/百分比不加空格（`32°C`、`95%`）。

### 标点符号

中文语境用全角标点：`注意：该接口需要鉴权。`。全角标点与英文/数字间不加空格。

括号：中文语境用全角括号 `（详见说明）`；括号内有英文/数字用半角 `基于 Spring Boot (v3.2.0) 开发`；纯英文用半角。

引号：中文推荐直角引号 `「确定」按钮`，嵌套用 `『』`。

### 数字

技术文档统一用半角阿拉伯数字：`100 个并发连接`、`v2.1.0`、`端口号 8080`。

## 中英混排

### 术语处理

保留英文：专有名词（React、Kubernetes）、行业缩写（API、SDK、CI/CD）、命令和代码（`npm install`）、协议标准（HTTP、TCP/IP）、无公认中文翻译的术语（debounce、middleware）。

翻译为中文：有公认翻译的通用概念（数据库、服务器、负载均衡）、文档标题和章节名。

首次出现标注翻译：`本系统采用消息队列（Message Queue）实现异步通信。` 后续直接用中文。

避免过度翻译：`在 Controller 层做参数校验` ✓，`在控制器层做参数校验` ✗。`使用 Redis 做缓存` ✓，`使用"远程字典服务"做"会话"缓存` ✗。

## API 文档模板

```markdown
## 创建订单 / Create Order

### 基本信息
- **请求方式 (Method):** POST
- **请求路径 (Path):** `/api/v1/orders`
- **鉴权方式 (Auth):** Bearer Token

### 请求参数 (Request Parameters)

| 参数名 (Field) | 类型 (Type) | 必填 (Required) | 说明 (Description) |
|----------------|-------------|-----------------|-------------------|
| product_id | string | 是 | 商品 ID |
| quantity | integer | 是 | 购买数量，最小值为 1 |
| coupon_code | string | 否 | 优惠券码 |

### 请求示例
```json
{ "product_id": "prod_abc123", "quantity": 2 }
```

### 错误码 (Error Codes)

| 错误码 (Code) | 说明 (Description) | 处理建议 (Suggestion) |
|---------------|--------------------|----------------------|
| 40001 | 商品不存在 | 检查 product_id 是否正确 |
```

金额必须明确单位：`total_amount: 9900  // 单位：分`。不写 `99.00`（分还是元？浮点数精度问题）。

## README.md 中文模板

````markdown
# 项目名称

简短一句话介绍。

## 特性
- 特性一
- 特性二

## 快速开始

### 环境要求
- Node.js >= 20

### 安装
```bash
npm install your-package
```

### 基本用法
```typescript
import { YourPackage } from 'your-package';
const result = await client.doSomething();
```

## 文档
- [使用指南](./docs/guide.md)
- [API 参考](./docs/api.md)

## 许可证
[MIT](./LICENSE)
````

## 常见问题

**机翻味：** 避免被动语态（"被用来"→"用于"）、冗余代词、直译英文句式。

**句式欧化：** 长句拆短句，定语从句改并列句，一句话只说一件事。

**中英标点混用：** 中文句子用全角标点。`请先安装依赖，然后运行测试。` ✓，`请先安装依赖,然后运行测试.` ✗。

**缺乏结构化：** 用列表和表格组织信息，不要一大段文字堆砌。

## 写作检查清单

排版：中英文间有空格、中文与数字间有空格、中文语境全角标点、无全半角混用。

术语：专有名词保留英文、首次出现标注中英对照、不过度翻译、术语前后一致。

内容：句子简短无欧化长句、无多余被动语态、用列表/表格结构化、代码可直接运行。
