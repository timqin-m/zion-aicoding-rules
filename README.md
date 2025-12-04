# Zion Cursor Rules - 使用 Zion 后端构建自定义前端应用

> **预构建的 Cursor 规则，用于开发基于 Zion.app 作为后端即服务（BaaS）的自定义前端应用**

本仓库包含一套**生产就绪的 Cursor 规则**，使 AI Coding 工具（推荐Cursor）能够无缝集成 Zion 强大的后端基础设施。使用这些规则可以快速构建全栈应用，同时利用 Zion 的企业级 PostgreSQL 数据库、GraphQL API、Actionflow、AI Agent 等功能。

## 🎯 本仓库提供的内容

本仓库包含 **8 个专业规则文件**：

1. **理解 Zion 的架构** - 后端结构、GraphQL 端点、身份验证
2. **查询和变更数据库** - 从数据模型自动生成的 GraphQL schema
3. **执行后端逻辑** - 用于复杂、多步骤操作的 Actionflow
4. **集成第三方 API** - 使用导入的 API 定义作为后端中继
5. **利用 AI Agent** - RAG、工具使用、多模态 I/O、结构化 JSON 输出
6. **处理支付** - 支付宝、微信支付等国内支付方式集成
7. **管理二进制资源** - 图片/文件上传和管理
8. **获取项目 Schema** - MCP 服务器集成，实时访问 schema

## 📦 包含的规则文件

```
├── zion-backend-architecture.mdc      # 核心架构和 GraphQL 设置
├── zion-database-gql-api-rules.mdc    # 数据库 CRUD 操作
├── zion-actionflow-gql-api-rules.mdc  # 后端工作流和业务逻辑
├── zion-tpa-gql-api-rules.mdc         # 第三方 API 集成
├── zion-ai-agent-gql-api-rules.mdc    # AI Agent 功能
├── zion-payment-rules.mdc             # 支付处理（支付宝、微信支付等）
├── zion-binary-asset-upload-rules.mdc # 文件管理
└── zion-development-best-practices.mdc # 开发最佳实践
```

## 🚀 快速开始

### 前置要求

* Cursor 编辑器或任何支持 Cursor 规则的 AI 助手
* Zion.app 账号和项目
* GraphQL 和 TypeScript/JavaScript 的基础知识

### 步骤 1: 在 Zion 中构建后端

在使用这些规则之前，你需要在 Zion 中创建后端基础设施：

1. **注册** Zion.app 并创建新项目
2. **设计数据库** - 创建表、定义关系、设置数据模型（可以是用Zion AI数据库助手）
3. **构建后端逻辑** - 为复杂业务逻辑创建 Actionflow
4. **配置集成** - 根据需要设置支付、AI Agent、第三方 API
### 步骤 2: 配置 MCP 服务器

为了实时获取项目 Schema，需要配置 Zion MCP 服务器：

1. **安装 MCP 服务器**：
   在 Cursor 的 MCP 配置文件中添加：

```json
{
  "mcpServers": {
    "zion": {
      "command": "npx",
      "args": [
        "-y",
        "zion-mcp@latest"
      ]
    }
  }
}
```

2. **配置项目**：
   - 首次使用时，MCP 服务器会引导你完成 OAuth 认证
   - 认证后，可以设置当前项目上下文
   - 使用 `mcp_zion_get_project_schema` 获取项目的最新 Schema

3. **使用 MCP 功能**：
   - `mcp_zion_get_projects` - 列出所有项目
   - `mcp_zion_set_current_project` - 设置当前项目
   - `mcp_zion_get_project_schema` - 获取项目 Schema

### 步骤 3: 复制规则到项目

1. **克隆或下载本仓库**
2. **复制规则文件**到你的项目：

```bash
# 将规则文件复制到项目的 .cursor/rules/ 目录
cp *.mdc /path/to/your/project/.cursor/rules/
```

或者直接在项目根目录使用这些规则文件。

### 步骤 4: 开始构建

现在你可以使用 AI 助手来构建应用了！

**示例提示**：
```
使用 Zion 后端规则，创建一个 Next.js 博客应用，项目 ID 是 abc91xyY
```

AI 助手将：
* 自动使用正确的 GraphQL 端点
* 生成类型安全的查询和变更
* 配置 Apollo Client 和 WebSocket
* 实现身份验证流程
* 处理文件上传
* 集成支付功能

## 📚 规则文件详解

### 1. `zion-backend-architecture.mdc`

**用途**：核心架构和 GraphQL 设置  
**教 AI**：
* Zion 的 BaaS 架构
* GraphQL HTTP 和 WebSocket 端点
* Apollo Client + subscriptions-transport-ws 设置
* 身份验证令牌处理
* 项目结构和约定

**关键概念**：

```typescript
// HTTP 端点
https://zion-app.functorz.com/zero/{projectExId}/api/graphql-v2

// WebSocket 端点
wss://zion-app.functorz.com/zero/{projectExId}/api/graphql-subscription
```

### 2. `zion-database-gql-api-rules.mdc`

**用途**：通过自动生成的 GraphQL schema 进行数据库操作  
**教 AI**：
* 数据库表如何映射到 GraphQL 类型
* 获取数据的查询模式
* CRUD 操作的变更模式
* 关系处理（1:1、1:N、N:N）
* 过滤、排序和分页语法

**示例 AI 可以生成**：

```graphql
query GetPostsWithAuthors($limit: Int) {
  post(limit: $limit, order_by: {created_at: desc}) {
    id
    title
    content
    author {  # 关系自动处理
      id
      name
      email
    }
  }
}
```

### 3. `zion-actionflow-gql-api-rules.mdc`

**用途**：执行复杂的后端工作流  
**教 AI**：
* 同步 vs 异步 Actionflow
* 通过 GraphQL 调用 Actionflow
* 处理 Actionflow 参数和返回值
* 轮询异步 Actionflow 完成状态
* 错误处理和重试

**使用场景**：
* 多步骤业务逻辑
* 长时间运行的操作（LLM API 调用）
* 带回滚的数据库事务
* 邮件发送、通知
* 复杂的数据转换

### 4. `zion-tpa-gql-api-rules.mdc`

**用途**：第三方 API 集成  
**教 AI**：
* 使用导入的 OpenAPI 定义
* 后端作为认证中继
* 无 CORS 问题
* 安全的凭证管理

**优势**：
* 将 API 密钥保存在服务器端
* 集中式错误处理
* 请求/响应日志
* 速率限制控制

### 5. `zion-ai-agent-gql-api-rules.mdc`

**用途**：利用内置 AI 功能  
**教 AI**：
* 创建和管理 AI Agent
* RAG（检索增强生成）
* 工具使用 / 函数调用
* 多模态输入（文本、图片、音频）
* 结构化 JSON 输出
* 流式响应

**AI Agent 功能示例**：
* 带向量搜索的文档问答
* 图片分析和生成
* 自动化决策
* 从非结构化文本中提取数据

### 6. `zion-payment-rules.mdc`

**用途**：支付处理集成  
**教 AI**：
* 一次性支付流程
* 订阅管理
* 支付事件的 Webhook 处理
* 订单创建和状态跟踪
* 支付 UI 集成

**支持的支付方式**：
* 支付宝
* 微信支付
* 其他国内支付方式

### 7. `zion-binary-asset-upload-rules.mdc`

**用途**：文件和图片管理  
**教 AI**：
* 直接文件上传到 Zion 存储
* 图片优化和转换
* CDN URL 生成
* 文件元数据管理
* 资源的访问控制

**支持的文件类型**：
* 图片（JPG、PNG、GIF、WebP）
* 文档（PDF、DOCX、XLSX）
* 视频（MP4、WebM）
* 音频文件
* 通用二进制数据

## 💡 使用示例

### 示例 1: 使用 AI 助手构建博客

**你**："基于 Zion 项目后端创建一个 WEB引用，项目 exId 是 xxxx"

**AI 将**：
* 使用 MCP 获取项目 Schema
* 生成类型安全的 GraphQL 查询
* 配置 Apollo Client
* 创建博客列表和详情页面
* 实现实时更新（订阅）

### 示例 2: 构建电商应用

**你**："创建一个电商应用，包含商品列表、购物车和支付功能"

**AI 将使用**：
* `zion-backend-architecture.mdc` → 设置 Apollo Client
* `zion-database-gql-api-rules.mdc` → 创建商品查询
* `zion-actionflow-gql-api-rules.mdc` → 订单创建工作流
* `zion-payment-rules.mdc` → 支付集成
* `zion-binary-asset-upload-rules.mdc` → 商品图片上传

## 🛠️ 高级工作流

### 组合多个规则

规则可以智能地协同工作。例如：

**你**："构建一个社交媒体帖子创建流程，包含图片上传和高级功能的支付"

**AI 将使用**：
* `zion-backend-architecture.mdc` → 设置 Apollo Client
* `zion-database-gql-api-rules.mdc` → 创建帖子变更
* `zion-binary-asset-upload-rules.mdc` → 处理图片上传
* `zion-actionflow-gql-api-rules.mdc` → 多步骤帖子创建工作流
* `zion-payment-rules.mdc` → 高级功能付费墙
* `zion-ai-agent-gql-api-rules.mdc` → AI 驱动的内容审核

## 🎯 最佳实践

### 1. 规则选择

规则有 `alwaysApply` 元数据：
* **true**：AI 始终考虑此规则（例如，架构）
* **false**：AI 在相关时上下文应用

### 2. 保持 Schema 更新

使用 MCP 服务器在 Zion 项目结构更改时刷新 Schema：

**你**："刷新我的 Zion 项目 Schema"  
**AI**：获取项目的最新 Schema 到上下文

### 3. 对复杂逻辑使用 Actionflow

不要尝试在前端实现多步骤业务逻辑：

❌ **不要**：

```typescript
// 前端做太多事情
const createOrderWithInventoryCheck = async () => {
  const inventory = await checkInventory(productId);
  if (inventory > 0) {
    const order = await createOrder(...);
    await reduceInventory(productId);
    await sendEmail(...);
  }
};
```

✅ **应该**：

```typescript
// 让后端 Actionflow 处理
const createOrder = async () => {
  await invokeActionflow('create_order_with_checks', {
    product_id: productId,
    quantity: 1
  });
};
```


## 📖 其他资源

### Zion 文档

* [Zion 官方文档](https://www.functorz.com)
* 快速开始指南
* 数据库配置
* Actionflow 指南
* AI Agent 教程
* 支付集成

### GraphQL 资源

* [Apollo Client 文档](https://www.apollographql.com/docs/react/)
* GraphQL 最佳实践
* 订阅指南

### Cursor 和 AI 开发

* [Cursor 官方文档](https://cursor.sh/docs)
* Cursor 规则最佳实践
* [模型上下文协议 (MCP)](https://modelcontextprotocol.io)

### 社区和支持

* [Zion 官网](https://www.functorz.com)
* 技术支持：通过 Zion 平台联系


**准备构建你的下一个应用？**

1. ⭐ Star 本仓库
2. 📋 复制规则到你的项目
3. 🔌 配置 MCP 服务器
4. 🚀 开始使用 AI 构建！

**有问题？** 提交 issue 或通过 Zion 平台联系

---

_最后更新：2025 年 12/04_

