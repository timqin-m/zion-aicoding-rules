# Zion Cursor Skills - 使用 Zion 后端构建自定义前端应用

> **预构建的 Cursor Skills，用于开发基于 Zion（[functorz.com](https://www.functorz.com)）作为后端即服务（BaaS）的自定义前端应用**

本仓库包含一套**生产就绪的 Cursor Skills**，使 AI Coding 工具（推荐Cursor）能够无缝集成 Zion 强大的后端基础设施。使用这些 Skills 可以快速构建全栈应用，同时利用 Zion 的企业级 PostgreSQL 数据库、GraphQL API、Actionflow、AI Agent 等功能。

> **注意**：本仓库适用于 Cursor 2.3.41 及以上版本，使用新的 Skills 系统（`.cursor/skills/` 目录结构）。

## 🎯 本仓库提供的内容

本仓库包含 **12 个专业 Skills 文件**：

1. **理解 Zion 的架构** - 后端结构、GraphQL 端点、身份验证
2. **查询和变更数据库** - 从数据模型自动生成的 GraphQL schema
3. **执行后端逻辑** - 用于复杂、多步骤操作的 Actionflow
4. **集成第三方 API** - 使用导入的 API 定义作为后端中继
5. **利用 AI Agent** - RAG、工具使用、多模态 I/O、结构化 JSON 输出
6. **处理支付** - 支付宝、微信支付等国内支付方式集成
7. **管理二进制资源** - 图片/文件上传和管理
8. **获取项目 Schema** - MCP 服务器集成，实时访问 schema
9. **UI 设计规范** - 基于有机/自然设计风格（Organic/Natural）的 React UI 设计规则
10. **Zeabur 部署规范** - 在 Zeabur 平台部署 React + TypeScript + Vite 项目的最佳实践
11. **微信小程序开发** - 微信小程序与 Zion 后端集成的特有规则和最佳实践
12. **微信小程序支付** - 微信小程序中使用 Zion 后端进行微信支付的方法

## 📦 包含的 Skills 文件

本仓库包含 **12 个 Skills 文件**，均为 `SKILL.md` 格式（Markdown with YAML frontmatter），每个 Skill 位于独立的目录中：

```
zion-aicoding-rules/
├── zion-backend-architecture/
│   └── SKILL.md                          # 核心架构和 GraphQL 设置
├── zion-database-gql-api/
│   ├── SKILL.md                          # 数据库 CRUD 操作
│   └── references/
│       └── detailed-reference.md         # 详细参考文档
├── zion-actionflow-gql-api/
│   └── SKILL.md                          # 后端工作流和业务逻辑
├── zion-tpa-gql-api/
│   └── SKILL.md                          # 第三方 API 集成
├── zion-ai-agent-gql-api/
│   └── SKILL.md                          # AI Agent 功能
├── zion-payment/
│   └── SKILL.md                          # 支付处理（支付宝、微信支付等）
├── zion-binary-asset-upload/
│   └── SKILL.md                          # 文件管理
├── zion-development-best-practices/
│   └── SKILL.md                          # 开发最佳实践
├── ui-design-rules/
│   └── SKILL.md                          # UI 设计规范（有机/自然风格）
├── zeabur-deployment/
│   └── SKILL.md                          # Zeabur 平台部署规范
├── wechat-miniprogram/
│   └── SKILL.md                          # 微信小程序开发规则
└── wechat-miniprogram-payment/
    └── SKILL.md                          # 微信小程序支付规则
```

所有 Skills 文件都包含 YAML frontmatter 元数据（`description` 和 `alwaysApply`），会被 Cursor 2.3.41+ 正确识别和应用。

## 🚀 快速开始

### 前置要求

* Cursor 编辑器 2.3.41 及以上版本（支持 Cursor Skills 系统）
* Zion（[functorz.com](https://www.functorz.com)）账号和项目

### 步骤 1: 在 Zion 中构建后端

在使用这些 Skills 之前，你需要在 Zion 中创建后端基础设施：

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

### 步骤 3: 导入 Skills 到项目

#### 通过 Cursor 聊天框下载（推荐）

这是最简单且可靠的方式：

1. 在 Cursor 的聊天框中，直接请求 AI 助手下载 Skills：
   ```
   请帮我下载并安装 zion-aicoding-rules Skills，GitHub 仓库地址是：
   https://github.com/functorz-tech/zion-aicoding-rules
   ```
   
   AI 助手会自动将 Skills 文件下载到项目的 `.cursor/skills/` 目录中。

#### 手动安装

1. 克隆或下载本仓库
2. 将各个 Skills 目录复制到你的项目根目录下的 `.cursor/skills/` 目录中
3. 确保目录结构为：`.cursor/skills/技能名/SKILL.md`
   
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

## 📚 Skills 文件详解

### 1. `zion-backend-architecture`

**用途**：核心架构和 GraphQL 设置  
**教 AI**：
* Zion 的 BaaS 架构
* GraphQL HTTP 和 WebSocket 端点
* Apollo Client + subscriptions-transport-ws 设置
* 身份验证令牌处理
* 项目结构和约定


### 2. `zion-database-gql-api`

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

### 3. `zion-actionflow-gql-api`

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

### 4. `zion-tpa-gql-api`

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

### 5. `zion-ai-agent-gql-api`

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

### 6. `zion-payment`

**用途**：支付处理集成  
**教 AI**：
* 一次性支付流程
* 订阅管理
* 支付事件的 Webhook 处理
* 订单创建和状态跟踪
* 支付 UI 集成

**支持的支付方式**：
* 支付宝
### 7. `zion-binary-asset-upload`

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
* 基于有机/自然设计风格（Organic/Natural），强调 wabi-sabi 美学
* 大地色调调色板（苔藓绿、赤陶土、米白色等）
* 有机形状和 blob 设计，拒绝 90 度直角
* 彩色阴影系统（苔藓绿、粘土橙色调）
* 自然纹理叠加和纸张质感
* 卡片设计、按钮设计、输入框设计
* 动画规则和性能优化
* 排版规则（Fraunces + Nunito/Quicksand）和响应式设计

**设计风格**：
* wabi-sabi 美学：接受不完美，追求真实和自然
* 有机形状：软质 blob 形状，复杂的百分比 border-radius
* 自然纹理：全局颗粒纹理叠加（3-4% 不透明度）
* 温暖质感：触感、接地、平静的视觉体验
* 彩色阴影：柔和阴影带自然色彩色调，禁止纯黑色
* 不对称设计：通过旋转图像、偏移元素创造有机真实感

### 10. `zeabur-deployment`

**用途**：Zeabur 平台部署规范和最佳实践  
**教 AI**：
* Zeabur 平台部署要求
* 必需配置文件（tsconfig.json、vite.config.ts 等）
* TypeScript 配置要求（兼容 TypeScript 4.9.5）
* 构建和部署流程
* 环境变量配置
* 常见部署问题和解决方案

**核心要求**：
* 必须包含所有必需的配置文件
* TypeScript 配置必须兼容 TypeScript 4.9.5
* 必须使用正确的 moduleResolution 设置
* 必须配置正确的构建脚本

### 11. `wechat-miniprogram`

**用途**：微信小程序与 Zion 后端集成的特有规则和最佳实践  
**教 AI**：
* 微信小程序特有的模块系统（CommonJS）
* 使用 `wx.request` 进行 GraphQL 请求
* 微信小程序文件结构（.wxml、.wxss、.wxs、.js）
* 小程序生命周期和页面管理
* 与 Zion 后端的身份验证集成
* 文件上传到 Zion 存储
* 小程序特有的错误处理和网络请求封装

**核心规则**：
* 必须使用 CommonJS 模块系统（`require`/`module.exports`）
* 必须使用 `wx.request` 而非 `fetch` 或 `axios`
* 必须遵循微信小程序的文件结构和命名规范
* 必须正确处理小程序的生命周期和页面路由

**适用场景**：
* 开发微信小程序前端
* 小程序与 Zion 后端 GraphQL API 集成
* 小程序中的文件上传和管理
* 小程序用户身份验证

### 12. `wechat-miniprogram-payment`

**用途**：微信小程序中使用 Zion 后端进行微信支付的方法  
**教 AI**：
* 微信小程序原生支付集成
* 订单创建和支付流程
* 调用微信支付 API（`wx.requestPayment`）
* 支付完成后查询订单状态
* 支付错误处理和重试机制
* 订单表配置和绑定

**前置条件**：
* 必须在 Zion 编辑器内配置订单表
* 必须配置微信支付的 AppID、商户号、API 密钥等
* 每个支付必须关联一个订单 ID

**支付流程**：
1. 创建订单（通过 Actionflow 或 GraphQL mutation）
2. 获取微信支付参数（通过 GraphQL mutation）
3. 调用 `wx.requestPayment` 发起支付
4. 支付完成后查询订单状态
5. 处理支付结果和错误

**适用场景**：
* 微信小程序中的商品购买
* 小程序中的服务订阅
* 小程序中的虚拟商品支付
* 小程序中的会员充值

## 💡 使用示例

### 示例 1: 使用 AI 助手构建博客（Web 应用）

**你**："基于 Zion 项目后端创建一个 WEB 应用，项目 exId 是 xxxx"

**AI 将**：
* 使用 MCP 获取项目 Schema
* 生成类型安全的 GraphQL 查询
* 配置 Apollo Client
* 创建博客列表和详情页面
* 实现实时更新（订阅）

### 示例 2: 使用 AI 助手构建微信小程序

**你**："基于 Zion 项目后端创建一个微信小程序，项目 exId 是 xxxx"

**AI 将**：
* 使用 MCP 获取项目 Schema
* 创建微信小程序项目结构（app.json、pages、components）
* 使用 `wx.request` 封装 GraphQL 请求
* 实现小程序页面和组件（.wxml、.wxss、.js）
* 集成 Zion 后端身份验证
* 实现文件上传功能
* 配置小程序支付流程（如需要）



## 🛠️ 高级工作流

### 组合多个 Skills

Skills 可以智能地协同工作。例如：

**你**："构建一个社交媒体帖子创建流程，包含图片上传和高级功能的支付"

**AI 将使用**：
* `zion-backend-architecture` → 设置 Apollo Client（Web）或 wx.request（小程序）
* `zion-database-gql-api` → 创建帖子变更
* `zion-binary-asset-upload` → 处理图片上传
* `zion-actionflow-gql-api` → 多步骤帖子创建工作流
* `zion-payment` → 支付宝支付（Web）
* `wechat-miniprogram-payment` → 微信支付（小程序）
* `zion-ai-agent-gql-api` → AI 驱动的内容审核

### 微信小程序开发工作流

**你**："构建一个微信小程序电商应用，包含商品展示、购物车和微信支付"

**AI 将使用**：
* `wechat-miniprogram` → 创建小程序项目结构和页面
* `zion-backend-architecture` → 配置 GraphQL 请求封装
* `zion-database-gql-api` → 查询商品数据和创建订单
* `wechat-miniprogram-payment` → 实现微信支付流程
* `zion-binary-asset-upload` → 处理商品图片上传

## 🎯 最佳实践

### 1. 保持 Schema 更新

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
* Cursor Skills 最佳实践
* [模型上下文协议 (MCP)](https://modelcontextprotocol.io)

### 社区和支持

* [Zion 官网](https://www.functorz.com)
* 技术支持：通过 Zion 平台联系


**准备构建你的下一个应用？**

1. ⭐ Star 本仓库
2. 📋 通过 Cursor 聊天框下载 Skills 到你的项目（推荐）或手动复制到 `.cursor/skills/` 目录
3. 🔌 配置 MCP 服务器
4. 🚀 开始使用 AI 构建！

**有问题？** 提交 issue 或通过 Zion 平台联系

### Skills 类型说明

安装的 Skills 会根据其配置自动应用：

- **Always Apply**：每个聊天会话都会应用（`alwaysApply: true`）
- **Apply Intelligently**：当 AI 判断相关时自动应用（`alwaysApply: false`）
- **Apply to Specific Files**：当文件匹配指定模式时应用（`applyToFiles` 配置）
- **Apply Manually**：在聊天中使用 `@技能名` 手动应用

### 更新 Skills

当本仓库更新 Skills 后，你可以再次通过聊天框请求 AI 助手更新 Skills，或者手动下载最新的 Skills 文件到 `.cursor/skills/` 目录。

### ⚠️ 关于 Cursor 版本要求

本仓库的 Skills 适用于 **Cursor 2.3.41 及以上版本**，使用新的 Skills 系统（`.cursor/skills/` 目录结构）。如果你使用的是旧版本的 Cursor，请升级到最新版本。

---

### 关于 Zion

Zion（[functorz.com](https://www.functorz.com)）是一款不用写代码也可以开发出微信小程序、WEB、AI Agent 的全栈无代码开发工具。Zion 拥有全栈无代码开发的能力，你可以在 Zion 上通过可视化的配置完成一个包含前后端的完整项目。作为一个高自由度的开发工具，Zion 的服务端接口也拥有标准的规范（包括不限于用户鉴权、支付服务、AI Agent服务、对象存储服务、数据库服务），结合用户在 Zion 上的自定义配置（数据库配置、AI Agent配置、支付配置）以及各项服务的规范，无需在Zion上搭建前端页面即可让 Cursor 结合 Zion 的后端开发一个可商用的完整软件产品！

---

## 📱 支持的开发场景

本仓库的 Skills 支持以下开发场景：

### Web 应用开发
- React + TypeScript + Vite 项目
- Apollo Client 集成
- 实时订阅（WebSocket）
- 支付宝支付集成
- Zeabur 平台部署

### 微信小程序开发
- 微信小程序原生开发
- CommonJS 模块系统
- wx.request 网络请求
- 微信支付集成
- 小程序文件上传

### AI Agent 开发
- RAG 检索增强生成
- 多模态输入输出
- 工具使用和函数调用
- 结构化 JSON 输出

---

_最后更新：2026年1月20日_


