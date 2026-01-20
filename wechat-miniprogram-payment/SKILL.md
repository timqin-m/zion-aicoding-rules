---
description: WeChat Pay integration in Mini Programs via Zion backend. Use when: (1) Creating payment orders in Mini Programs, (2) Getting WeChat Pay parameters via GraphQL, (3) Calling wx.requestPayment API, (4) Querying payment status after completion, (5) Handling payment errors and retries, (6) Configuring order tables in Zion editor
alwaysApply: false
applyToFiles:
  - "**/miniprogram/**"
  - "**/payment/**"
---

# 微信小程序支付

## 概述

Zion 支持微信小程序原生支付集成，使得基于 Zion 构建的微信小程序可以使用微信支付完成订单支付。

## 前置条件

在使用微信支付功能之前，必须在 Zion 编辑器内完成以下配置：

1. **选择订单表**：项目的数据库中必须存在一个概念上的订单表，且必须在 Zion 编辑器内将该表绑定为项目的"订单表"设置。每个支付都必须关联一个订单 ID。
2. **配置微信支付密钥和证书**：必须在 Zion 编辑器内配置微信支付的 AppID、商户号、API 密钥等。

完成上述配置后，才能通过 GraphQL API 调用微信支付接口。支付流程包括：创建订单、获取微信支付参数、调用微信支付 API、支付完成后查询订单状态。

## 订单创建

每个项目的概念订单表都有自己的结构，不一定需要命名为 "order"，因为技术上每个项目的任何一个表都可以在 Zion 编辑器内绑定到项目的"订单表"设置。不过通常它们应该包含以下信息：订单属于哪个账户、订单金额是多少、订单包含哪些商品（通常通过 1:n 关系）。

在调用微信支付接口之前，必须先创建订单。订单创建的具体实现方式取决于项目的架构设计。关于订单创建的安全最佳实践，请参考 `zion-development-best-practices.mdc` 规则文件。

订单创建后应该返回订单 ID（或订单标识符）以及支付所需的订单相关信息（如订单金额、商品描述等）。

### 创建订单示例（使用 Actionflow）

**重要说明**：
- `fz_invoke_action_flow` 返回的是 `Json` 类型（叶子类型），**不能进行子选择**
- 必须直接返回 Json，然后在代码中解析

```javascript
// 调用创建订单 Actionflow
const query = `
  mutation CreateOrder($args: Json!) {
    fz_invoke_action_flow(
      actionFlowId: "0e4de7f6-2ef2-49ee-8b2f-2b16afc6767d"
      versionId: -1
      args: $args
    )
  }
`

const variables = {
  args: {
    amount: 100  // 订单金额（单位：分）
  }
}

const result = await graphqlRequest(query, variables, token)
// fz_invoke_action_flow 返回的是 Json 类型，直接解析
const orderData = result.fz_invoke_action_flow
if (!orderData) {
  throw new Error('创建订单失败：返回数据为空')
}
const orderId = orderData.order_id
const amount = orderData.amount
```

## 创建微信支付

前置条件：订单 ID（或订单标识符）、商品描述、订单总金额（单位：分）和认证 token。订单 ID 和金额应该来自订单创建的返回值。

**重要说明**：
- 必须使用创建订单 ActionFlow 输出的订单 ID 和金额（不要有自定义订单逻辑）
- 调用 Zion 提供的 `createWechatPayment` 接口时，确保请求包含 Bearer token
- `amount` 参数类型是 `BigDecimal!`，**单位是元**，需要将分转换为元（除以 100）
- `type` 参数必须设置为 `WECHATPAY_MINIPROGRAM`
- 返回的 `SignResult.message` 字段包含 JSON 格式的支付参数，需要解析

向项目的 GraphQL API 发送 mutation：

查询：
```gql
mutation CreateWechatPayment(
  $orderId: Long!
  $amount: BigDecimal!
  $description: String!
  $type: PaymentType!
) {
  createWechatPayment(
    orderId: $orderId
    amount: $amount
    description: $description
    type: $type
  ) {
    message
    status
  }
}
```

变量示例：
```json
{
  "orderId": 1234567890,
  "amount": 0.01,
  "description": "积分充值",
  "type": "WECHATPAY_MINIPROGRAM"
}
```

输出示例：
```json
{
  "data": {
    "createWechatPayment": {
      "status": "SUCCESS",
      "message": "{\"appId\":\"wx1234567890abcdef\",\"timeStamp\":\"1234567890\",\"nonceStr\":\"5K8264ILTKCH16CQ2502SI8ZNMTM67VS\",\"package\":\"prepay_id=wx1234567890abcdef\",\"signType\":\"RSA\",\"paySign\":\"oR9d8PuhnIc+YZ8cBHFCwfgpaK9gd7vaRvKYDqLvM1COz2eymBHy39QbG3DLHnzGSUBXx3vc3817WHqScob37YqJAfCfiYyN44fXqPfGd0WfYzRj2R6HUbDmnTpNt4Z3LHKz03yXGOPd3xn0gEcwP3vKHyWGXgLY9vID3vXiQIJ0hqws5GQzdAlXgZlSRsbXnsjq8o6RvnTsfQxr9bYqod4KYlSA3J1Fz8r40jNHmqsoRVkF4ilaMd0NqR2bMnyzX11pLmgcWW15ftqpjHr8QxszrJk1Nn4nG4qPX5lxmLxzNXWxP4bHQ4s8SUj7NhdMDnmq2y1hJyxSpx\"}"
    }
  }
}
```

**关键点**：
- `status` 字段：`SUCCESS` 表示成功，`FAIL` 表示失败
- `message` 字段：包含 JSON 字符串格式的支付参数，需要 `JSON.parse()` 解析
- 解析后的 JSON 包含以下字段：
  - `appId`：小程序 AppID
  - `timeStamp`：时间戳（字符串）
  - `nonceStr`：随机字符串
  - `package`：统一下单接口返回的 prepay_id 参数值，格式为 `prepay_id=xxx`
  - `signType`：签名类型，通常为 `RSA`
  - `paySign`：签名

## 调用微信支付

**必须使用微信小程序原生 API `wx.requestPayment`**：

```javascript
// 获取支付参数后，调用微信支付
wx.requestPayment({
  appId: paymentParams.appId,
  timeStamp: paymentParams.timeStamp,
  nonceStr: paymentParams.nonceStr,
  package: paymentParams.package,
  signType: paymentParams.signType,
  paySign: paymentParams.paySign,
  success: (res) => {
    console.log('支付成功', res)
    // 支付成功后的处理逻辑
  },
  fail: (err) => {
    console.error('支付失败', err)
    // 支付失败后的处理逻辑
  }
})
```

**关键点**：
- 必须使用 `wx.requestPayment` API，这是微信小程序官方提供的支付接口
- 所有参数必须与后端返回的参数完全一致，不能修改
- 支付成功后，微信会自动调用后端的支付回调接口（webhook），更新订单状态

## 支付返回处理

当用户完成支付后（无论成功或失败），`wx.requestPayment` 的 `success` 或 `fail` 回调会被触发。在该回调中，需要：

1. **等待后端处理**：由于 webhook 是异步过程，应该等待 2-3 秒让后端 webhook 有足够时间处理支付通知并更新订单状态
2. **查询订单状态**：使用订单 ID 查询订单状态，确认支付是否完成
3. **更新用户权限**：如果订单状态为"已支付"，更新用户权限（如积分余额）
4. **处理支付结果**：根据订单状态显示相应的提示信息（支付成功、支付失败、支付取消等）
5. **页面跳转**：如果支付成功，可以跳转到应用主页或其他页面

## Webhook 处理

微信支付会通过 webhook 向 Zion 项目的后端发送支付通知，相应的 actionflow 用于处理这些 webhook 请求。因此前端不需要特殊处理 webhook 处理器的逻辑。

由于 webhook 是异步过程，在支付成功后查询订单状态前，应该等待 2-3 秒，确保后端 webhook 有足够时间处理支付通知并更新订单状态。如果查询时订单状态尚未更新，可以提示用户稍后刷新页面或稍等片刻。

## 工具函数示例

### 完整的支付工具函数

```javascript
// utils/payment.js
const { graphqlRequest } = require('./graphql.js')

/**
 * 创建订单
 * @param {number} amount - 订单金额（单位：分）
 * @param {string} token - 认证token
 * @returns {Promise<{order_id: number, amount: number}>} 返回订单ID和金额
 */
async function createOrder(amount, token) {
  const query = `
    mutation CreateOrder($args: Json!) {
      fz_invoke_action_flow(
        actionFlowId: "0e4de7f6-2ef2-49ee-8b2f-2b16afc6767d"
        versionId: -1
        args: $args
      )
    }
  `
  
  const variables = {
    args: {
      amount: amount
    }
  }
  
  const result = await graphqlRequest(query, variables, token)
  // fz_invoke_action_flow 返回的是 Json 类型，直接解析
  const orderData = result.fz_invoke_action_flow
  if (!orderData) {
    throw new Error('创建订单失败：返回数据为空')
  }
  return {
    order_id: orderData.order_id,
    amount: orderData.amount
  }
}

/**
 * 创建微信支付
 * @param {string} outTradeNo - 商户订单号（订单ID）
 * @param {string} description - 商品描述
 * @param {number} totalAmount - 订单总金额（单位：分）
 * @param {string} token - 认证token
 * @returns {Promise<object>} 返回微信支付参数
 */
async function createWechatPay(outTradeNo, description, totalAmount, token) {
  // amount 需要转换为 BigDecimal（元），所以除以100
  const amountInYuan = totalAmount / 100
  
  const query = `
    mutation CreateWechatPayment(
      $orderId: Long!
      $amount: BigDecimal!
      $description: String!
      $type: PaymentType!
    ) {
      createWechatPayment(
        orderId: $orderId
        amount: $amount
        description: $description
        type: $type
      ) {
        message
        status
      }
    }
  `
  
  const variables = {
    orderId: parseInt(outTradeNo),
    amount: amountInYuan,
    description: description,
    type: 'WECHATPAY_MINIPROGRAM'
  }
  
  const result = await graphqlRequest(query, variables, token)
  const signResult = result.createWechatPayment
  
  if (signResult.status !== 'SUCCESS') {
    throw new Error('创建微信支付失败：' + signResult.message)
  }
  
  // message 字段包含 JSON 格式的支付参数
  try {
    const paymentParams = JSON.parse(signResult.message)
    return paymentParams
  } catch (e) {
    // 如果不是 JSON，可能 message 就是错误信息
    throw new Error('解析支付参数失败：' + signResult.message)
  }
}

/**
 * 查询订单状态
 * @param {number} orderId - 订单ID
 * @param {string} token - 认证token
 * @returns {Promise<object>} 返回订单信息
 */
async function queryOrderStatus(orderId, token) {
  const query = `
    query GetOrder($id: bigint!) {
      order_by_pk(id: $id) {
        id
        status
        pay_amount
        account_account
        ud_jifenshuzhi_1f263d
      }
    }
  `
  
  const variables = { id: orderId }
  const result = await graphqlRequest(query, variables, token)
  return result.order_by_pk
}

/**
 * 完整的微信支付流程
 * @param {number} amount - 订单金额（单位：分）
 * @param {string} description - 商品描述
 * @param {string} token - 认证token
 * @returns {Promise<{orderId: number, success: boolean}>} 返回订单ID和支付结果
 */
async function processWechatPayment(amount, description, token) {
  try {
    // 1. 创建订单
    const order = await createOrder(amount, token)
    const orderId = order.order_id
    
    // 2. 创建微信支付
    const paymentParams = await createWechatPay(
      orderId.toString(),
      description,
      amount,
      token
    )
    
    // 3. 调用微信支付
    return await new Promise((resolve, reject) => {
      wx.requestPayment({
        appId: paymentParams.appId,
        timeStamp: paymentParams.timeStamp,
        nonceStr: paymentParams.nonceStr,
        package: paymentParams.package,
        signType: paymentParams.signType,
        paySign: paymentParams.paySign,
        success: async (res) => {
          // 支付成功，等待几秒后查询订单状态（让后端 webhook 处理完成）
          await new Promise(resolve => setTimeout(resolve, 2000))
          
          try {
            const orderStatus = await queryOrderStatus(orderId, token)
            resolve({
              orderId: orderId,
              success: orderStatus.status === '已支付',
              orderStatus: orderStatus
            })
          } catch (error) {
            // 查询订单状态失败，但支付已成功
            resolve({
              orderId: orderId,
              success: false,
              error: error.message
            })
          }
        },
        fail: (err) => {
          // 支付失败或取消
          reject(err)
        }
      })
    })
  } catch (error) {
    console.error('微信支付失败:', error)
    throw error
  }
}

module.exports = {
  createOrder,
  createWechatPay,
  queryOrderStatus,
  processWechatPayment
}
```

## 页面使用示例

```javascript
// pages/payment/payment.ts
const { processWechatPayment } = require('../../utils/payment.js')

Page({
  data: {
    amount: 0, // 订单金额（单位：分）
    description: '', // 商品描述
    loading: false
  },

  onLoad(options) {
    if (options.amount) {
      this.setData({
        amount: parseInt(options.amount),
        description: options.description || '积分充值'
      })
    }
  },

  async handlePayment() {
    const token = wx.getStorageSync('token')
    if (!token) {
      wx.showToast({
        title: '请先登录',
        icon: 'none'
      })
      return
    }

    this.setData({ loading: true })

    try {
      const result = await processWechatPayment(
        this.data.amount,
        this.data.description,
        token
      )

      if (result.success) {
        wx.showToast({
          title: '支付成功',
          icon: 'success'
        })
        
        // 支付成功，跳转回上一页或首页
        setTimeout(() => {
          wx.navigateBack()
        }, 1500)
      } else {
        wx.showToast({
          title: '支付未完成',
          icon: 'none'
        })
      }
    } catch (error) {
      console.error('支付失败:', error)
      wx.showToast({
        title: error.message || '支付失败',
        icon: 'none',
        duration: 2000
      })
    } finally {
      this.setData({ loading: false })
    }
  }
})
```

## 注意事项

1. **金额单位转换**：
   - 订单金额在前端使用**分**为单位（如 100 表示 1.00 元）
   - 调用 `createWechatPayment` 时，`amount` 参数类型是 `BigDecimal!`，**单位是元**，需要除以 100 转换
   - 例如：100 分 → 1.00 元

2. **订单 ID**：必须使用创建订单 ActionFlow 返回的订单 ID，不要自定义

3. **支付参数解析**：
   - `createWechatPayment` 返回 `SignResult` 类型
   - `status` 字段必须为 `SUCCESS` 才表示成功
   - `message` 字段包含 JSON 字符串，需要 `JSON.parse()` 解析后才能获取支付参数

4. **支付参数**：所有支付参数必须与后端返回的参数完全一致，不能修改

5. **异步处理**：支付成功后需要等待 2-3 秒再查询订单状态，确保后端 webhook 处理完成

6. **错误处理**：需要处理支付取消、支付失败等各种情况

7. **用户提示**：支付过程中应该显示加载状态，支付完成后应该显示明确的成功或失败提示

8. **Actionflow 返回类型**：
   - `fz_invoke_action_flow` 返回 `Json` 类型（叶子类型），**不能进行子选择**
   - 必须直接返回 Json，然后在代码中解析

## 参考资源

- [微信小程序支付官方文档](https://developers.weixin.qq.com/miniprogram/dev/api/payment/wx.requestPayment.html)
- [Zion.app 支付规则](./zion-payment-rules.mdc)
- [微信小程序开发规则](./wechat-miniprogram-rules.mdc)
