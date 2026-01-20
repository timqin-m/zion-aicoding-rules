---
description: "Zeabur platform deployment rules and best practices for React + TypeScript + Vite projects. Use when: (1) Deploying to Zeabur platform, (2) Configuring required files (tsconfig.json, vite.config.ts), (3) Setting up TypeScript 4.9.5 compatibility, (4) Configuring SPA routing (React Router), (5) Troubleshooting deployment issues (502 errors, host restrictions)"
alwaysApply: true
---
# Zeabur 部署规范

本文档总结了在 Zeabur 平台上部署 React + TypeScript + Vite 项目的最佳实践和注意事项。

## 核心要求

### 1. 必需配置文件 [MUST]

项目必须包含以下配置文件，否则构建会失败：

- **`tsconfig.json`**: TypeScript 编译配置
- **`tsconfig.node.json`**: Node.js 环境的 TypeScript 配置（用于 `vite.config.ts`）
- **`vite.config.ts`**: Vite 构建工具配置
- **`package.json`**: 项目依赖和脚本配置

### 2. TypeScript 配置要求 [MUST]

**兼容性要求**：
- 如果使用 TypeScript 4.9.5，必须使用 `moduleResolution: "node"`（不支持 `bundler`）
- 必须包含 `esModuleInterop: true` 和 `allowSyntheticDefaultImports: true`

**标准配置示例**：
```json
{
  "compilerOptions": {
    "target": "ES2020",
    "useDefineForClassFields": true,
    "lib": ["ES2020", "DOM", "DOM.Iterable"],
    "module": "ESNext",
    "skipLibCheck": true,
    "moduleResolution": "node",
    "resolveJsonModule": true,
    "isolatedModules": true,
    "noEmit": true,
    "jsx": "react-jsx",
    "esModuleInterop": true,
    "allowSyntheticDefaultImports": true,
    "strict": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noFallthroughCasesInSwitch": true
  },
  "include": ["src"],
  "references": [{ "path": "./tsconfig.node.json" }]
}
```

### 3. 构建脚本配置 [MUST]

**package.json 中的构建脚本**：
```json
{
  "scripts": {
    "build": "tsc && vite build"
  }
}
```

**构建流程**：
1. `tsc` - TypeScript 类型检查（必须通过）
2. `vite build` - 生产环境构建

### 4. 依赖版本锁定 [SHOULD]

**推荐做法**：
- 使用精确版本号（不使用 `^` 或 `~`）
- 在 `package-lock.json` 中锁定依赖版本
- 确保本地构建和 Zeabur 构建使用相同的依赖版本

### 5. 构建输出检查 [SHOULD]

**构建成功后检查**：
- 确认 `dist/` 目录已生成
- 检查构建输出中的警告（如 chunk 大小警告）
- 验证静态资源路径正确

### 6. SPA 部署配置（React Router 等）[MUST]

**对于使用 React Router 的单页应用，必须在部署前配置**：

1. **配置 `vite.config.ts`**：
   ```typescript
   export default defineConfig({
     // ... 其他配置
     publicDir: 'public',
     preview: {
       host: '0.0.0.0',
       port: 8080, // Zeabur 使用的端口
       strictPort: false,
       // 允许所有主机访问（生产环境部署需要）
       allowedHosts: true,
     },
   })
   ```

2. **配置 `package.json`**：
   ```json
   {
     "scripts": {
       "start": "vite preview --host 0.0.0.0 --port ${PORT:-8080}"
     }
   }
   ```

3. **在 Zeabur 中配置**：
   - 服务类型：**Web Service**（不是 Static Site）
   - 启动命令：`npm start`
   - 构建输出目录：`dist`

**为什么需要这些配置**：
- `allowedHosts: true`：防止 "Blocked request. This host is not allowed" 错误
- `vite preview`：自动处理 SPA 路由重定向，防止 502 错误
- Web Service 类型：提供服务器支持，处理客户端路由

### 7. 常见问题排查 [REFERENCE]

#### 问题 1: `Cannot find name 'process'`
**原因**：使用了 `process.env` 但没有类型定义
**解决**：
- 移除相关代码（推荐）
- 或安装 `@types/node`：`npm i --save-dev @types/node`

#### 问题 2: `Module resolution error`
**原因**：TypeScript 配置中的 `moduleResolution` 不兼容
**解决**：使用 `"moduleResolution": "node"` 而不是 `"bundler"`

#### 问题 3: `Unused variable/import`
**原因**：TypeScript 严格模式要求所有变量都被使用
**解决**：
- 移除未使用的变量
- 或使用下划线前缀：`const [, setState] = useState()`

#### 问题 4: `Missing tsconfig.json`
**原因**：配置文件被删除或未提交到版本控制
**解决**：确保所有配置文件都在版本控制中

## 部署前检查清单

在推送到 Zeabur 之前，请确保：

### 部署配置检查
- [ ] 本地 `npm run build` 成功执行
- [ ] 所有必需的配置文件都存在
- [ ] `package.json` 中的依赖版本已锁定

### SPA 部署配置检查（如果使用 React Router）[MUST]
- [ ] `vite.config.ts` 中已配置 `preview.allowedHosts: true`
- [ ] `vite.config.ts` 中已配置 `preview.port: 8080`
- [ ] `vite.config.ts` 中已配置 `preview.host: '0.0.0.0'`
- [ ] `package.json` 中已配置 `start` 脚本：`vite preview --host 0.0.0.0 --port ${PORT:-8080}`
- [ ] 在 Zeabur 中已配置服务类型为 **Web Service**
- [ ] 在 Zeabur 中已设置启动命令为 `npm start`
- [ ] 在 Zeabur 中已设置构建输出目录为 `dist`

**注意**：这些配置必须在首次部署前完成，可以避免出现 502 错误和主机访问限制问题。

## 最佳实践总结

1. **配置完整**：确保所有必需的配置文件都存在
2. **本地验证**：在推送到 Zeabur 之前，本地构建必须成功
3. **版本锁定**：使用精确的依赖版本，避免构建环境差异
4. **预防性配置**：对于 SPA 应用（React Router 等），在首次部署前就配置好 `vite.config.ts` 的 `preview` 选项和 `package.json` 的 `start` 脚本，避免出现 502 错误和主机访问限制问题
5. **服务类型选择**：SPA 应用必须使用 **Web Service** 类型，不能使用 Static Site（除非平台支持路由重定向）

### 8. 502 错误和主机访问限制排查 [REFERENCE]

**问题描述**：部署显示成功，但访问页面时出现 `502: SERVICE_UNAVAILABLE` 错误，或显示 "Blocked request. This host is not allowed" 错误。

**根本原因**：
1. **SPA 路由需要服务器支持**：React Router 等单页应用需要服务器将所有路由请求重定向到 `index.html`
2. **Zeabur 静态网站可能不支持路由重定向**：需要使用 Web Service 类型运行静态文件服务器
3. **Vite preview 的主机访问限制**：默认只允许 localhost 访问，需要配置允许外部域名

**解决方案（推荐）**：

#### 使用 Web Service + Vite Preview

1. **在 Zeabur 控制台中配置**：
   - 服务类型：**Web Service**（不是 Static Site）
   - 启动命令：`npm start`
   - 构建输出目录：`dist`

2. **配置 `vite.config.ts`**：
   ```typescript
   export default defineConfig({
     // ... 其他配置
     preview: {
       host: '0.0.0.0',
       port: 8080, // Zeabur 使用的端口
       strictPort: false,
       // 允许所有主机访问（生产环境部署需要）
       allowedHosts: true,
     },
   })
   ```

3. **配置 `package.json`**：
   ```json
   {
     "scripts": {
       "start": "vite preview --host 0.0.0.0 --port ${PORT:-8080}"
     }
   }
   ```

**重要注意事项**：
- ✅ `allowedHosts` **只能**在 `vite.config.ts` 配置文件中设置，**不能**作为命令行参数
- ✅ 设置为 `true` 允许所有主机访问（适合生产环境部署）
- ✅ 端口设置为 `8080`（Zeabur 默认端口），或使用 `${PORT:-8080}` 从环境变量读取
- ✅ `vite preview` 会自动处理 SPA 路由重定向（所有路由都返回 `index.html`）
- ❌ **不要**在命令行中使用 `--allowedHosts`，会导致 `CACError: Unknown option '--allowedHosts'` 错误

#### 常见错误排查

1. **`CACError: Unknown option '--allowedHosts'`**：
   - **原因**：`allowedHosts` 不能作为命令行参数
   - **解决**：只在 `vite.config.ts` 中配置 `allowedHosts: true`，不要添加到命令行

2. **`Blocked request. This host is not allowed`**：
   - **原因**：Vite preview 默认只允许 localhost 访问
   - **解决**：在 `vite.config.ts` 中设置 `allowedHosts: true`

3. **502 错误**：
   - 确认服务类型为 **Web Service**（不是 Static Site）
   - 确认启动命令为 `npm start`
   - 确认 `vite.config.ts` 中 `preview` 配置正确
   - 确认端口设置为 `8080`（Zeabur 默认端口）
   - 查看 Zeabur 部署日志，检查是否有运行时错误

#### 方案 2：使用 Static Site（如果支持路由重定向）
如果 Zeabur 的静态网站支持路由重定向：
1. 创建 `public/_redirects` 文件：
   ```
   /*    /index.html   200
   ```
2. 在 Zeabur 中配置为 **"Static Site"**
3. 构建输出目录设置为 `dist`

**检查清单**：
- [ ] 确认 Zeabur 服务类型配置正确（推荐使用 "Static Site"）
- [ ] 确认构建输出目录为 `dist`
- [ ] 如果使用 Web Service，确认启动命令正确
- [ ] 检查 Zeabur 部署日志，查看是否有运行时错误
- [ ] **SPA 路由配置**：确保创建了 `public/_redirects` 文件，将所有路由重定向到 `index.html`

#### 方案 3：SPA 路由重定向配置（必须）
对于使用 React Router 的单页应用（SPA），需要配置路由重定向：

1. **创建 `public/_redirects` 文件**：
   ```
   /*    /index.html   200
   ```
   这会将所有请求重定向到 `index.html`，让 React Router 处理客户端路由。

2. **确保 `vite.config.ts` 包含 `publicDir` 配置**：
   ```typescript
   export default defineConfig({
     plugins: [react()],
     publicDir: 'public', // 确保 public 目录被复制到 dist
   })
   ```

3. **验证构建输出**：
   - 构建后，`dist/_redirects` 文件应该存在
   - 如果没有，检查 `public` 目录是否正确配置

4. **如果静态网站不支持重定向（推荐使用此方案）**：
   - 安装 `serve` 包：`npm install --save-dev serve`
   - 更新 `package.json` 的 `start` 脚本：
     ```json
     {
       "scripts": {
         "start": "npx serve -s dist -l $PORT"
       }
     }
     ```
     **注意**：使用 `npx serve` 而不是直接 `serve`，使用 `$PORT` 而不是 `${PORT:-3000}`（Zeabur 会自动设置 PORT 环境变量）
   - 在 Zeabur 中配置为 **Web Service**，启动命令设置为 `npm start`
   - `serve -s` 会自动处理 SPA 路由重定向（所有路由都返回 `index.html`）

5. **如果仍然 502 错误，检查以下事项**：
   - 确认 `serve` 包已正确安装（检查 `package-lock.json`）
   - 确认构建成功，`dist` 目录存在且包含 `index.html`
   - 查看 Zeabur 部署日志，检查是否有运行时错误
   - 尝试使用 `vite preview` 作为备用方案：
     ```json
     {
       "scripts": {
         "start": "vite preview --host 0.0.0.0 --port $PORT"
       }
     }
     ```

## 参考

- [Vite 构建配置](https://vitejs.dev/config/)
- [TypeScript 编译选项](https://www.typescriptlang.org/tsconfig)
- [Zeabur 部署文档](https://zeabur.com/docs)
