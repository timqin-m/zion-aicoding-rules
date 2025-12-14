# Zion Cursor Rules - 使用 Zion 后端构建自定义前端应用

> **预构建的 Cursor 规则，用于开发基于 Zion（[functorz.com](https://www.functorz.com)）作为后端即服务（BaaS）的自定义前端应用**

本仓库包含一套**生产就绪的 Cursor 规则**，使 AI Coding 工具（推荐Cursor）能够无缝集成 Zion 强大的后端基础设施。使用这些规则可以快速构建全栈应用，同时利用 Zion 的企业级 PostgreSQL 数据库、GraphQL API、Actionflow、AI Agent 等功能。

## 🎯 本仓库提供的内容

本仓库包含 **9 个专业规则文件**：

1. **理解 Zion 的架构** - 后端结构、GraphQL 端点、身份验证
2. **查询和变更数据库** - 从数据模型自动生成的 GraphQL schema
3. **执行后端逻辑** - 用于复杂、多步骤操作的 Actionflow
4. **集成第三方 API** - 使用导入的 API 定义作为后端中继
5. **利用 AI Agent** - RAG、工具使用、多模态 I/O、结构化 JSON 输出
6. **处理支付** - 支付宝、微信支付等国内支付方式集成
7. **管理二进制资源** - 图片/文件上传和管理
8. **获取项目 Schema** - MCP 服务器集成，实时访问 schema
9. **UI 设计规范** - 基于 Apple 设计风格的 React UI 设计规则

## 📦 包含的规则文件

本仓库包含 **9 个规则文件**，均为 `.mdc` 格式（Markdown with frontmatter），位于仓库根目录：

```
zion-aicoding-rules/
├── zion-backend-architecture.mdc         # 核心架构和 GraphQL 设置
├── zion-database-gql-api-rules.mdc        # 数据库 CRUD 操作
├── zion-actionflow-gql-api-rules.mdc      # 后端工作流和业务逻辑
├── zion-tpa-gql-api-rules.mdc            # 第三方 API 集成
├── zion-ai-agent-gql-api-rules.mdc       # AI Agent 功能
├── zion-payment-rules.mdc                # 支付处理（支付宝、微信支付等）
├── zion-binary-asset-upload-rules.mdc    # 文件管理
├── zion-development-best-practices.mdc   # 开发最佳实践
└── ui-design-rules.mdc                   # UI 设计规范（Apple 风格）
```

所有规则文件都包含 frontmatter 元数据（`description` 和 `alwaysApply`），会被 Cursor 正确识别和应用。

## 🚀 快速开始

### 前置要求

* Cursor 编辑器或任何支持 Cursor 规则的 AI Coding 工具
* Zion（[functorz.com](https://www.functorz.com)）账号和项目

### 步骤 1: 在 Zion 中构建后端

在使用这些规则之前，你需要在 Zion 中创建后端基础设施：

1. **注册** Zion（[functorz.com](https://www.functorz.com)）并创建新项目
2. **设计数据库** - 创建表、定义关系、设置数据模型（可以使用 Zion AI 数据库助手）
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

3. **MCP 服务器提供的工具**：
   - `set_current_working_directory` - 设置当前工作目录
   - `get_projects` - 列出所有项目
   - `set_current_project` - 设置当前项目
   - `get_current_project` - 获取当前项目上下文
   - `get_project_schema` - 获取项目 Schema
   - `reauth` - 重新进行 OAuth 认证

### 步骤 3: 导入规则到项目

有两种方式可以将规则导入到你的项目中：

#### 方式 1: 通过 Cursor 聊天框下载（推荐）⭐

这是最简单且可靠的方式：

1. 在 Cursor 的聊天框中，直接请求 AI 助手下载规则：
   ```
   请帮我下载并安装 zion-aicoding-rules 规则，GitHub 仓库地址是：
   https://github.com/your-username/zion-aicoding-rules
   ```

2. AI 助手会自动：
   - 从 GitHub 仓库下载所有 `.mdc` 规则文件
   - 创建 `.cursor/rules/` 目录结构
   - 将规则文件转换为正确的格式（`规则名/RULE.md`）
   - 安装到你的项目中

**优势**：
- ✅ 简单快捷：只需在聊天框中请求即可
- ✅ 自动处理：AI 助手会处理所有文件转换和安装
- ✅ 可靠稳定：避免了 Cursor 官方 GitHub 导入功能的 bug

**注意**：目前 Cursor 官方的 GitHub 直接导入功能存在 bug，不推荐使用。建议通过聊天框让 AI 助手下载安装。

#### 方式 2: 手动复制规则文件

如果你需要离线使用或自定义规则：

1. **克隆或下载本仓库**
2. **创建规则目录结构**并复制规则文件：

### 步骤 4: 开始构建

现在你可以使用 AI Coding 工具来构建应用了！

**示例提示**：
```
连接 Zion 平台后端，创建一个博客应用，项目 ID 是 xxxx
```

AI 助手将：
* 自动使用正确的 GraphQL 端点
* 生成类型安全的查询和变更
* 配置 Apollo Client 和 WebSocket
* 实现身份验证流程
* 处理文件上传
* 集成支付功能

## 📚 规则文件详解

### 1. `zion-backend-architecture`

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

### 2. `zion-database-gql-api-rules`

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

### 3. `zion-actionflow-gql-api-rules`

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

### 4. `zion-tpa-gql-api-rules`

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

### 5. `zion-ai-agent-gql-api-rules`

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

### 6. `zion-payment-rules`

**用途**：支付处理集成  
**教 AI**：
* 一次性支付流程
* 订阅管理
* 支付事件的 Webhook 处理
* 订单创建和状态跟踪
* 支付 UI 集成

**支持的支付方式**：
* 支付宝
### 7. `zion-binary-asset-upload-rules`

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

### 8. `zion-development-best-practices`

**用途**：开发最佳实践和安全规范  
**教 AI**：
* TypeScript 编码标准
* Apollo Client 配置要求
* 代码质量检查（ESLint、TypeScript）
* 架构决策（GraphQL CRUD vs Actionflow）
* 安全实践建议

### 9. `ui-design-rules`

**用途**：React 项目 UI 设计规则和 AI 执行规范  
**教 AI**：
* 基于 Apple 设计风格（Human Interface Guidelines）
* 颜色系统、间距系统、圆角系统
* 卡片设计、按钮设计、输入框设计
* 动画规则和性能优化
* 排版规则和响应式设计

**设计风格**：
* 毛玻璃效果（Frosted Glass）和深度层次
* 柔和的阴影和微妙的视觉反馈
* 流畅自然的动画过渡
* 系统化、语义化的颜色体系
* 充足的留白和清晰的层次结构

## 💡 使用示例

### 示例 1: 使用 AI 助手构建博客

**你**："基于 Zion 项目后端创建一个 WEB 应用，项目 exId 是 xxxx"

**AI 将**：
* 使用 MCP 获取项目 Schema
* 生成类型安全的 GraphQL 查询
* 配置 Apollo Client
* 创建博客列表和详情页面
* 实现实时更新（订阅）

### 示例 2: 构建电商应用

**你**："创建一个电商应用，包含商品列表、购物车和支付功能"

**AI 将使用**：
* `zion-backend-architecture` → 设置 Apollo Client
* `zion-database-gql-api-rules` → 创建商品查询
* `zion-actionflow-gql-api-rules` → 订单创建工作流
* `zion-payment-rules` → 支付集成
* `zion-binary-asset-upload-rules` → 商品图片上传

## 🛠️ 高级工作流

### 组合多个规则

规则可以智能地协同工作。例如：

**你**："构建一个社交媒体帖子创建流程，包含图片上传和高级功能的支付"

**AI 将使用**：
* `zion-backend-architecture` → 设置 Apollo Client
* `zion-database-gql-api-rules` → 创建帖子变更
* `zion-binary-asset-upload-rules` → 处理图片上传
* `zion-actionflow-gql-api-rules` → 多步骤帖子创建工作流
* `zion-payment-rules` → 支付宝支付
* `zion-ai-agent-gql-api-rules` → AI 驱动的内容审核

## 🎯 最佳实践

### 1. 规则选择

规则有 `alwaysApply` 元数据：
* **true**：AI 始终考虑此规则（例如，架构）
* **false**：AI 在相关时上下文应用

### 2. 保持 Schema 更新

使用 MCP 服务器在 Zion 项目结构更改时刷新 Schema：

**你**："刷新我的 Zion 项目 Schema"  
**AI**：获取项目的最新 Schema 到上下文



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
2. 📋 通过 Cursor 聊天框下载规则到你的项目（推荐）或手动复制
3. 🔌 配置 MCP 服务器
4. 🚀 开始使用 AI 构建！

**有问题？** 提交 issue 或通过 Zion 平台联系

### 规则类型说明

安装的规则会根据其配置自动应用：

- **Always Apply**：每个聊天会话都会应用
- **Apply Intelligently**：当 AI 判断相关时自动应用
- **Apply to Specific Files**：当文件匹配指定模式时应用
- **Apply Manually**：在聊天中使用 `@规则名` 手动应用

### 更新规则

当本仓库更新规则后，你可以再次通过聊天框请求 AI 助手更新规则，或者手动下载最新的规则文件。

### ⚠️ 关于 GitHub 直接导入

目前 Cursor 官方的 GitHub 直接导入功能（Remote Rule (GitHub)）存在 bug，可能导致规则无法正确导入。因此我们推荐使用聊天框下载的方式，这是目前最可靠的方法。

---

### 关于 Zion

Zion（[functorz.com](https://www.functorz.com)）是一款不用写代码也可以开发出微信小程序、WEB、AI Agent 的全栈无代码开发工具。Zion 拥有全栈无代码开发的能力，你可以在 Zion 上通过可视化的配置完成一个包含前后端的完整项目。作为一个高自由度的开发工具，Zion 的服务端接口也拥有标准的规范（包括不限于用户鉴权、支付服务、AI Agent服务、对象存储服务、数据库服务），结合用户在 Zion 上的自定义配置（数据库配置、AI Agent配置、支付配置）以及各项服务的规范，无需在Zion上搭建前端页面即可让 Cursor 结合 Zion 的后端开发一个可商用的完整软件产品！

---

_最后更新：2025年12月12日_


