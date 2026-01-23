# Zion Backend Connect Skill

> **整合的 Cursor Skill，用于开发基于 Zion（[functorz.com](https://www.functorz.com)）作为后端即服务（BaaS）的自定义前端应用**

本仓库提供一个**整合的 Cursor Skill**，将所有 Zion 后端集成功能合并为一个统一的 skill，使 AI Coding 工具（推荐 Cursor）能够无缝集成 Zion 强大的后端基础设施。

> **注意**：本仓库适用于 Cursor 2.3.41 及以上版本，使用新的 Skills 系统（`.cursor/skills/` 目录结构）。

## 🚀 快速安装

### 通过 skills.sh 安装（推荐）

```bash
npx skills add timqin-m/zion-backend-connect-skill
```

### 手动安装

1. 克隆或下载本仓库
2. 将 `zion-backend-connect-skills` 目录复制到你的项目根目录下的 `.cursor/skills/` 目录中
3. 确保目录结构为：`.cursor/skills/zion-backend-connect-skills/SKILL.md`

## 📦 包含的内容

这个整合的 skill 包含所有 Zion 后端集成的完整功能：

1. **Zion 后端架构** - 后端结构、GraphQL 端点、身份验证
2. **数据库操作** - 从数据模型自动生成的 GraphQL schema，包含详细的参考文档（1340+ 行）
3. **Actionflows** - 用于复杂、多步骤操作的同步和异步工作流
4. **第三方 API 集成** - 使用导入的 API 定义作为后端中继
5. **AI Agents** - RAG、工具使用、多模态 I/O、结构化 JSON 输出
6. **支付处理** - 支付宝、微信支付等国内支付方式集成
7. **二进制资源管理** - 图片/文件上传和管理
8. **开发最佳实践** - TypeScript、Apollo Client、安全规范
9. **UI 设计规则** - 基于有机/自然设计风格（Organic/Natural）的 React UI 设计规则
10. **Zeabur 部署规范** - 在 Zeabur 平台部署 React + TypeScript + Vite 项目的最佳实践
11. **微信小程序开发** - 微信小程序与 Zion 后端集成的特有规则和最佳实践
12. **微信小程序支付** - 微信小程序中使用 Zion 后端进行微信支付的方法

**总计 4628 行**，包含所有详细参考文档、示例代码、工具函数、检查清单和错误处理指南。

## 🎯 优势

- ✅ **只需安装一个 skill**，而不是 12 个独立的 skills
- ✅ **所有功能整合在一个文档中**，便于查找和使用
- ✅ **减少技能目录的复杂性**
- ✅ **更好的上下文管理**，AI 可以同时访问所有相关功能
- ✅ **内容完整未精简**，包含所有详细参考和示例代码

## 📚 使用示例

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

## 🛠️ 前置要求

* Cursor 编辑器 2.3.41 及以上版本（支持 Cursor Skills 系统）
* Zion（functorz.com）账号和项目

### 配置 MCP 服务器（可选但推荐）

为了实时获取项目 Schema，可以配置 Zion MCP 服务器：

1. **安装 MCP 服务器**：在 Cursor 的 MCP 配置文件中添加：

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
   * 首次使用时，MCP 服务器会引导你完成 OAuth 认证
   * 认证后，可以设置当前项目上下文

## 📖 其他资源

### Zion 文档

* [Zion 官方文档](https://www.functorz.com/)
* [快速开始指南](https://www.functorz.com/)

### GraphQL 资源

* [Apollo Client 文档](https://www.apollographql.com/docs/react/)
* [GraphQL 最佳实践](https://graphql.org/learn/best-practices/)

### Cursor 和 AI 开发

* [Cursor 官方文档](https://cursor.sh/docs)
* [Cursor Skills 最佳实践](https://cursor.sh/docs)

## 🎯 支持的开发场景

### Web 应用开发

* React + TypeScript + Vite 项目
* Apollo Client 集成
* 实时订阅（WebSocket）
* 支付宝支付集成
* Zeabur 平台部署

### 微信小程序开发

* 微信小程序原生开发
* CommonJS 模块系统
* wx.request 网络请求
* 微信支付集成
* 小程序文件上传

### AI Agent 开发

* RAG 检索增强生成
* 多模态输入输出
* 工具使用和函数调用
* 结构化 JSON 输出

## ⚠️ 关于 Cursor 版本要求

本仓库的 Skill 适用于 **Cursor 2.3.41 及以上版本**，使用新的 Skills 系统（`.cursor/skills/` 目录结构）。如果你使用的是旧版本的 Cursor，请升级到最新版本。

---

### 关于 Zion

Zion（functorz.com）是一款不用写代码也可以开发出微信小程序、WEB、AI Agent 的全栈无代码开发工具。Zion 拥有全栈无代码开发的能力，你可以在 Zion 上通过可视化的配置完成一个包含前后端的完整项目。作为一个高自由度的开发工具，Zion 的服务端接口也拥有标准的规范（包括不限于用户鉴权、支付服务、AI Agent服务、对象存储服务、数据库服务），结合用户在 Zion 上的自定义配置（数据库配置、AI Agent配置、支付配置）以及各项服务的规范，无需在Zion上搭建前端页面即可让 Cursor 结合 Zion 的后端开发一个可商用的完整软件产品！

---

**准备构建你的下一个应用？**

1. ⭐ Star 本仓库
2. 📋 通过 `npx skills add timqin-m/zion-backend-connect-skill` 安装
3. 🔌 配置 MCP 服务器（可选）
4. 🚀 开始使用 AI 构建！

**有问题？** 提交 issue 或通过 Zion 平台联系

_最后更新：2026年1月23日_
