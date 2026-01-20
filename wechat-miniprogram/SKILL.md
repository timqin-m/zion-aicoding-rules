---
description: "WeChat Mini Program integration with Zion.app backend. Use when: (1) Developing WeChat Mini Programs, (2) Using CommonJS module system (require/module.exports), (3) Making GraphQL requests via wx.request, (4) Handling Mini Program file structure (.wxml, .wxss, .wxs, .js), (5) Integrating authentication, (6) Uploading files to Zion storage"
alwaysApply: false
applyToFiles:
  - "**/miniprogram/**"
  - "**/*.wxml"
  - "**/*.wxss"
  - "**/*.wxs"
  - "**/project.config.json"
  - "**/app.json"
---

# 微信小程序 + Zion.app 开发规则

## Overview

本文档规定了微信小程序与 Zion.app 后端集成时的开发规则和最佳实践。所有开发必须严格遵循本文档中的规则。

**相关文档**：
- [Zion.app 后端架构](./zion-backend-architecture.mdc)
- [Zion.app 二进制资源上传](./zion-binary-asset-upload-rules.mdc)
- [Zion.app 数据库操作](./zion-database-gql-api-rules.mdc)

---

## 模块系统

### 使用 CommonJS [MUST]

**必须使用 CommonJS 模块系统**：

```javascript
// ✅ 正确
const { functionName } = require('./utils/file.js')
module.exports = { functionName }

// ❌ 错误
import { functionName } from './utils/file.js'
```

---

## 网络请求

### 使用 wx.request [MUST]

**必须使用 `wx.request` 进行网络请求**：

```javascript
function graphqlRequest(query, variables = {}, token = null) {
  return new Promise((resolve, reject) => {
    wx.request({
      url: 'https://zion-app.functorz.com/zero/{projectExId}/api/graphql-v2',
      method: 'POST',
      data: { query, variables },
      header: {
        'Content-Type': 'application/json',
        ...(token ? { 'Authorization': `Bearer ${token}` } : {})
      },
      success: (res) => {
        if (res.data.errors) {
          reject(new Error(res.data.errors[0].message))
        } else {
          resolve(res.data.data)
        }
      },
      fail: reject
    })
  })
}
```

---

## 用户认证

### 静默登录实现 [MUST]

**实现微信小程序静默登录的步骤**：

```javascript
// 1. 获取微信登录 code
const loginRes = await new Promise((resolve, reject) => {
  wx.login({ success: resolve, fail: reject })
})

// 2. 调用 Zion GraphQL mutation 进行静默登录
const query = `
  mutation LoginWithWechatMiniApp($code: String!) {
    loginWithWechatMiniApp(code: $code) {
      account {
        id
        username
        profileImageUrl  // 必须包含
      }
      jwt { token }
    }
  }
`

const result = await graphqlRequest(query, { code: loginRes.code })
const { account, jwt } = result.loginWithWechatMiniApp

// 3. 保存 token 和账户信息
wx.setStorageSync('token', jwt.token)
wx.setStorageSync('account', account)
```

**开发要求**：
- **必须请求 `profileImageUrl` 字段**：用于后续头像显示
- `wx.login` 返回的 `code` 只能使用一次，5 分钟内有效

---

## 文件上传到 OSS

### 上传流程 [MUST]

上传文件到 Zion.app 的 OSS 存储必须遵循以下流程。详细协议请参考 [Zion.app 二进制资源上传规则](./zion-binary-asset-upload-rules.mdc)。

#### Step 1: 计算文件的 MD5 Base64

**必须使用 `wx.getFileInfo` API 计算 MD5**：

```javascript
const fileInfo = await new Promise((resolve, reject) => {
  wx.getFileInfo({ filePath: filePath, digestAlgorithm: 'md5', success: resolve, fail: reject })
})
// fileInfo.digest 是十六进制字符串，需要转换为 Base64
// 转换逻辑：将十六进制字符串转换为字节数组，再转换为 Base64
```

#### Step 2: 获取 Presigned Upload URL

```javascript
const query = `mutation GetImagePresignedUrl($md5: String!, $suffix: MediaFormat!) {
  imagePresignedUrl(imgMd5Base64: $md5, imageSuffix: $suffix, acl: PRIVATE) {
    imageId uploadUrl uploadHeaders
  }
}`
const result = await graphqlRequest(query, { md5: md5Base64, suffix: 'JPEG' }, token)
const { imageId, uploadUrl, uploadHeaders } = result.imagePresignedUrl
```

#### Step 3: 上传文件

**必须遵循以下规则**：
1. **必须保留所有 uploadHeaders**：presigned URL 的签名是基于这些 headers 计算的，移除任何 header 都会导致 403 错误
2. **必须确保上传的数据和计算 MD5 时使用的数据完全一致**
3. **必须使用真正的 ArrayBuffer**：不能依赖 `instanceof ArrayBuffer` 检查

```javascript
// 读取文件数据并转换为 ArrayBuffer
const fileManager = wx.getFileSystemManager()
let fileData = fileManager.readFileSync(filePath)
let originalFileData = fileData instanceof ArrayBuffer ? fileData : 
  (() => {
    const uint8Array = new Uint8Array(fileData)
    const buffer = new ArrayBuffer(uint8Array.length)
    new Uint8Array(buffer).set(uint8Array)
    return buffer
  })()

// 上传文件（必须保留所有 uploadHeaders）
await new Promise((resolve, reject) => {
  wx.request({
    url: uploadUrl,
    method: 'PUT',
    data: originalFileData,
    header: { 'Content-Type': 'image/jpeg', ...uploadHeaders },
    responseType: 'text',
    success: (res) => {
      if (res.statusCode === 200 || res.statusCode === 204) {
        resolve(imageId)
      } else {
        reject(new Error(`上传失败: ${res.statusCode}`))
      }
    },
    fail: reject
  })
})
```

### 常见错误处理

- **`InvalidDigest`**：必须使用 `wx.getFileInfo` API 计算 MD5，确保上传数据与计算 MD5 时使用的数据完全一致
- **`SignatureDoesNotMatch`**：必须保留所有 uploadHeaders，不要移除任何 header（特别是 Content-MD5）
- **`文件数据格式不正确`**：使用属性检查而不是 `instanceof`，强制转换为真正的 ArrayBuffer

### 文件下载（处理网络 URL）[MUST]

**处理网络资源时必须先下载到本地**：

```javascript
if (avatarUrl.startsWith('http://') || avatarUrl.startsWith('https://')) {
  const downloadRes = await new Promise((resolve, reject) => {
    wx.downloadFile({
      url: avatarUrl,
      success: (res) => {
        if (res.statusCode === 200 && res.tempFilePath) {
          resolve(res)
        } else {
          reject(new Error('下载失败'))
        }
      },
      fail: reject
    })
  })
  localFilePath = downloadRes.tempFilePath
}
```

---

## 用户信息编辑

### 头像选择实现 [MUST]

**使用微信小程序原生组件实现头像选择**：

```xml
<button open-type="chooseAvatar" bindchooseavatar="onChooseAvatar">
  <view class="avatar-wrapper">
    <image class="avatar" src="{{userInfo.avatar}}" mode="aspectFill" wx:if="{{userInfo.avatar}}"></image>
    <view class="avatar-placeholder" wx:else>
      <text>{{userInfo.nickname ? userInfo.nickname.charAt(0) : '👤'}}</text>
    </view>
  </view>
</button>
```

```javascript
async onChooseAvatar(e) {
  let localFilePath = e.detail.avatarUrl
  // 处理网络 URL（微信默认头像需要先下载）
  if (localFilePath.startsWith('http://') || localFilePath.startsWith('https://')) {
    const downloadRes = await new Promise((resolve, reject) => {
      wx.downloadFile({ url: localFilePath, success: resolve, fail: reject })
    })
    localFilePath = downloadRes.tempFilePath
  }
  const imageId = await uploadImage(localFilePath, token)
  await updateAccount(accountId, imageId, null, token)
}
```

### 昵称编辑实现 [MUST]

```xml
<input type="nickname" value="{{userInfo.nickname}}" bindblur="onNicknameBlur" />
```

```javascript
async onNicknameBlur(e) {
  const nickname = e.detail.value.trim()
  if (nickname && nickname !== this.data.userInfo.nickname) {
    await updateAccount(accountId, null, nickname, token)
  }
}
```

---

## 数据模型操作

### 创建记录 [MUST]

**当表不支持直接设置外键字段时，必须使用两步操作**：

```javascript
// Step 1: 先创建空记录
const createResult = await graphqlRequest(`
  mutation CreateRecord {
    insert_record_one(object: {}) {
      id
    }
  }
`, {}, token)

// Step 2: 更新记录，设置关联字段
await graphqlRequest(`
  mutation UpdateRecord($id: bigint!, $img_id: bigint!, $user_id: bigint) {
    update_record_by_pk(
      pk_columns: {id: $id}
      _set: {img_id: $img_id, user_id: $user_id}
    ) { id }
  }
`, { id: createResult.insert_record_one.id, img_id: imageId, user_id: userId }, token)
```

---

## 开发要求

### 头像上传和读取 [MUST]

**必须遵循以下要求**：
- **必须使用 `wx.getFileInfo` API 计算 MD5**，不要手动实现或使用第三方库
- **必须保留所有 uploadHeaders**（特别是 Content-MD5），presigned URL 的签名依赖所有 headers
- **必须确保上传的数据和计算 MD5 时使用的数据完全一致**
- **必须处理网络 URL**：微信默认头像等网络资源需要先使用 `wx.downloadFile` 下载到本地
- **登录时必须请求 `profileImageUrl` 字段**：`loginWithWechatMiniApp` 的 GraphQL 查询必须包含 `profileImageUrl`
- **必须从服务器获取最新信息**：页面加载时优先从服务器获取账户信息，不能只依赖本地存储

### 自定义 TabBar [MUST]

**必须完成以下配置**：
- **必须在 `app.json` 中设置 `"custom": true`**
- **必须在页面 JSON 配置中引用组件**：`"usingComponents": { "custom-tab-bar": "/custom-tab-bar/index" }`
- **必须在页面 `onLoad` 中更新选中状态**：使用 `this.getTabBar().setData({ selected: index })`
- **必须为页面添加底部 padding**：避免内容被 tabBar 遮挡，建议 `padding-bottom: calc(160rpx + env(safe-area-inset-bottom))`

```javascript
// app.json
"tabBar": { "custom": true, "list": [...] }

// 页面 JSON
{ "usingComponents": { "custom-tab-bar": "/custom-tab-bar/index" } }

// 页面 JS
onLoad() {
  if (typeof this.getTabBar === 'function' && this.getTabBar()) {
    this.getTabBar().setData({ selected: 0 })
  }
}
```

### 变量作用域 [MUST]

**必须遵循以下规则**：
- **循环内使用的变量必须在循环外声明**：避免作用域问题导致 `ReferenceError`
- **必须在使用前初始化变量**：特别是可能为 `null` 或 `undefined` 的变量

```javascript
// ❌ 错误：在循环内声明，循环外使用
while (attempts < maxAttempts) {
  let imageId = null  // 在循环内声明
}
this.saveToHistory(imageUrl, imageId)  // 会报错

// ✅ 正确：在循环外声明
let imageId = null
while (attempts < maxAttempts) {
  if (content.image.id) {
    imageId = content.image.id
  }
}
this.saveToHistory(imageUrl, imageId)  // 正常使用
```

### ArrayBuffer 处理 [MUST]

**必须遵循以下规则**：
- **不能依赖 `instanceof ArrayBuffer` 检查**：小程序环境中可能返回 `false`
- **必须使用属性检查**：检查 `byteLength`、`buffer`、`byteOffset` 等属性
- **必须转换为真正的 ArrayBuffer**：确保上传时使用真正的 ArrayBuffer

### 页面布局 [MUST]

**必须遵循以下规则**：
- **自定义 tabBar 页面必须添加底部 padding**：避免内容被遮挡
- **必须考虑安全区域**：使用 `env(safe-area-inset-bottom)` 适配不同设备

```css
.container {
  padding-bottom: calc(160rpx + env(safe-area-inset-bottom));
}
```

### 自定义组件中加载远程 Icon（SVG/图片）[MUST]

**必须遵循以下规则**：
- **自定义组件中不能直接 require 其他模块**：会导致 `can not find module` 错误
- **必须直接使用 `wx.request` 调用 GraphQL API**：避免 require 依赖问题
- **必须提供 fallback（emoji 或占位符）**：确保即使 icon 加载失败，组件也能正常显示
- **必须在 `ready` 生命周期中加载**：不阻塞组件初始化，确保组件立即显示

```javascript
// ✅ 正确：硬编码 GraphQL URL，直接使用 wx.request
const app = getApp()
const ZION_GRAPHQL_URL = 'https://zion-app.functorz.com/zero/{projectExId}/api/graphql-v2'

Component({
  data: { iconUrl: '' },
  lifetimes: {
    attached() {}, // 立即显示组件（使用 fallback）
    ready() {
      this.loadIcon().catch(err => console.error('加载 icon 失败:', err))
    }
  },
  methods: {
    loadIcon() {
      return new Promise((resolve, reject) => {
        const token = app.getToken() || null
        wx.request({
          url: ZION_GRAPHQL_URL,
          method: 'POST',
          header: {
            'Content-Type': 'application/json',
            ...(token ? { 'Authorization': `Bearer ${token}` } : {})
          },
          data: {
            query: `query GetImageById($imageId: bigint!) {
              getImageById(imageId: $imageId) { id url }
            }`,
            variables: { imageId: 7000000007117943 }
          },
          success: (res) => {
            if (res.statusCode === 200 && res.data.data?.getImageById?.url) {
              this.setData({ iconUrl: res.data.data.getImageById.url })
              resolve(res.data.data.getImageById.url)
            } else {
              reject(new Error('获取 icon 失败'))
            }
          },
          fail: reject
        })
      })
    }
  }
})
```

```xml
<view class="icon-wrapper">
  <image class="icon-image" src="{{iconUrl}}" mode="aspectFit" wx:if="{{iconUrl}}"></image>
  <text class="icon-emoji" wx:if="{{!iconUrl}}">🎨</text>
</view>
```

---

## Lottie 动画集成

### 库文件引入 [MUST]

**从 npm/unpkg 下载的 `lottie-miniprogram` 是 webpack 打包的 UMD 格式，必须进行以下处理**：

1. **初始化 exports 对象**：在文件开头添加 `var exports = typeof exports !== 'undefined' ? exports : {};`
2. **添加 module.exports**：在文件末尾添加 CommonJS 导出：

```javascript
if (typeof module !== 'undefined' && module.exports) {
  module.exports = {
    setup: exports.setup,
    loadAnimation: exports.loadAnimation,
    freeze: exports.freeze,
    unfreeze: exports.unfreeze
  }
}
```

### Canvas 2D 使用 [MUST]

**必须按以下方式使用 Canvas 2D**：

```javascript
const query = wx.createSelectorQuery().in(this)
query.select('#lottie-canvas').fields({ node: true, size: true }).exec((res) => {
  const canvas = res[0].node
  const ctx = canvas.getContext('2d')
  const dpr = wx.getSystemInfoSync().pixelRatio
  const width = res[0].width || 600
  const height = res[0].height || 600
  
  canvas.width = width * dpr
  canvas.height = height * dpr
  ctx.scale(dpr, dpr)
  
  lottie.setup(canvas)
  lottie.loadAnimation({
    loop: true,
    autoplay: true,
    animationData: lottieData,
    rendererSettings: { context: ctx, clearCanvas: true }
  })
})
```

**必须遵循的规则**：
- Canvas 必须使用 `type="2d"` 和 `id` 属性
- 必须设置 `canvas.width` 和 `canvas.height`（考虑 DPR）
- Lottie JSON 数据通过 `getFileById` 从 Zion OSS 获取，然后使用 `wx.request` 下载内容

---

## 参考资源

- [微信小程序官方文档](https://developers.weixin.qq.com/miniprogram/dev/framework/)
- [Zion.app 文档](https://www.functorz.com/)
- [Zion.app 二进制资源上传规则](./zion-binary-asset-upload-rules.mdc)
- [Zion.app 数据库操作规则](./zion-database-gql-api-rules.mdc)
