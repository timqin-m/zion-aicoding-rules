---
description: "Payment processing via Zion backend (Alipay, WeChat Pay, etc.). Use when: (1) Creating payment orders, (2) Processing one-time payments, (3) Managing subscriptions, (4) Handling payment webhooks, (5) Tracking order status, (6) Integrating payment UI"
alwaysApply: false
---

# 概述
Zion 支持原生支付集成，使得基于 Zion 构建的项目的最终用户可以使用支付宝、微信支付等方式支付订单。

## 支付宝支付

### 前置条件
在使用支付宝支付功能之前，必须在 Zion 编辑器内完成以下配置：
1. **选择订单表**：项目的数据库中必须存在一个概念上的订单表，且必须在 Zion 编辑器内将该表绑定为项目的"订单表"设置。每个支付都必须关联一个订单 ID。
2. **配置支付密钥和证书**：必须在 Zion 编辑器内配置支付宝的支付密钥和证书。

完成上述配置后，才能通过 GraphQL API 调用支付宝支付接口。支付流程包括：创建订单、获取支付宝支付表单、跳转到支付宝支付页面完成支付、支付完成后根据返回 URL 返回并查询支付状态。

### 订单创建
每个项目的概念订单表都有自己的结构，不一定需要命名为 "order"，因为技术上每个项目的任何一个表都可以在 Zion 编辑器内绑定到项目的"订单表"设置。不过通常它们应该包含以下信息：订单属于哪个账户、订单金额是多少、订单包含哪些商品（通常通过 1:n 关系）。  

在调用支付宝支付接口之前，必须先创建订单。订单创建的具体实现方式取决于项目的架构设计。关于订单创建的安全最佳实践，请参考 `zion-development-best-practices.mdc` 规则文件。

订单创建后应该返回订单 ID（或订单标识符）以及支付所需的订单相关信息（如订单金额、主题/商品名称等）。

### 创建支付宝支付
前置条件：订单 ID（或订单标识符）、支付金额、主题（商品名称或订单描述）和返回 URL。订单 ID 和主题应该来自订单创建的返回值。  

**重要说明**：
- 必须使用创建订单 ActionFlow 输出的订单 ID 和 `goods_name`（不要有自定义订单逻辑）
- 调用 Zion 提供的 `alipayTrade` 接口时，确保请求包含 Bearer token
- `returnUrl` **必须**使用 `window.location.origin`（根域名），不要使用固定路径（如 `/payment/callback`）
- `totalAmount` 在前端以 `Float` 类型传递，后端会将其转换为字符串格式传给支付宝（支付宝要求 `total_amount` 必须是字符串类型，如 `"88.88"`）

向项目的 GraphQL API 发送 mutation：  
查询：
```gql
mutation AliPayTrade(
  $outTradeNo: String!
  $productCode: String!
  $subject: String!
  $totalAmount: Float!
  $returnUrl: String!
) {
  alipayTrade(
    bizContent: {
      out_trade_no: $outTradeNo
      product_code: $productCode
      subject: $subject
      total_amount: $totalAmount
    }
    returnUrl: $returnUrl
  )
}
```

变量示例：
```json
{
  "outTradeNo": "{orderId}",
  "productCode": "FAST_INSTANT_TRADE_PAY",
  "subject": "{orderSubject}",
  "totalAmount": 0.01,
  "returnUrl": "{window.location.origin}"
}
```

输出示例：
```json
{
  "data": {
    "alipayTrade": [
      "<form>...</form>",
      "{tradeNo}"
    ]
  }
}
```

该 mutation 返回一个包含两个元素的数组：
- 第一个元素：支付宝支付表单 HTML 字符串，需要提交该表单以跳转到支付宝支付页面
- 第二个元素：支付宝交易号（可选，供参考）
  - **注意**：根据支付宝规范，`alipay.trade.page.pay` 首次响应不应包含支付宝交易号（`trade_no`）
  - 实际的支付宝交易号应通过异步通知（`notify_url`）或主动查询接口获取
  - 如果返回数组的第二个元素存在，可能是商户订单号或交易号，仅供参考，不应依赖

### 跳转到支付宝支付页面

**⚠️ 关键警告：表单提交的正确方式**

支付宝返回的 HTML 表单包含已经格式化好的字段值，特别是 `biz_content` 字段包含 JSON 字符串。**绝对不要重新构建表单**，否则会导致字段值被错误转义，从而引发签名验证失败。

**正确做法：直接使用原始 HTML 表单（在当前页面打开）**

```typescript
// ✅ 正确：直接使用原始 HTML 表单
const formMatch = alipayFormHtml.match(/<form[^>]*>[\s\S]*?<\/form>/i);
const scriptMatch = alipayFormHtml.match(/<script[^>]*>[\s\S]*?<\/script>/i);

const extractedFormHtml = formMatch?.[0] || '';
const extractedScriptHtml = scriptMatch?.[0] || '';

// 组合完整的 HTML
const completeHtml = `
  <html>
    <head>
      <meta charset="utf-8">
      <title>正在跳转到支付宝...</title>
    </head>
    <body>
      ${extractedFormHtml}
      ${extractedScriptHtml}
    </body>
  </html>
`;

// 在当前页面直接写入原始 HTML（会跳转到支付宝，支付完成后会重定向回 returnUrl）
document.open();
document.write(completeHtml);
document.close();
```

**关键点**：
- 使用正则表达式提取 `form` 和 `script` 标签，保持原始格式
- 不进行任何 HTML 解析或修改
- 在当前页面直接写入 HTML，让浏览器自然处理表单提交
- 支付完成后，支付宝会根据创建支付时传递的 `returnUrl` 参数，将用户重定向回指定的返回 URL

### 支付返回处理
当用户完成支付后，支付宝会将用户重定向到创建支付时指定的 `returnUrl`（即 `window.location.origin`）。在该返回 URL 对应的页面中，需要：

1. **检测支付返回**：检查 URL 参数中是否包含 `out_trade_no`（订单号）
2. **查询订单状态**：使用订单 ID 查询订单状态，确认支付是否完成
3. **更新用户权限**：如果订单状态为"已支付"，更新用户权限（如 `can_use_ai`）
4. **处理支付结果**：根据订单状态显示相应的提示信息（支付成功、支付失败、支付取消等）
5. **页面跳转**：如果支付成功且用户已获得相应权限，可以跳转到应用主页或其他页面

**实现示例**：

```typescript
useEffect(() => {
  // 检查是否是支付返回
  const params = new URLSearchParams(window.location.search);
  const outTradeNo = params.get('out_trade_no');
  
  if (outTradeNo) {
    // 支付返回，查询订单状态并更新用户权限
    checkOrderAndUpdateAccess(outTradeNo);
  }
}, []);

const checkOrderAndUpdateAccess = async (orderId: string) => {
  // 等待几秒让后端 webhook 处理完成
  await new Promise(resolve => setTimeout(resolve, 2000));
  
  // 查询订单状态
  const { data } = await apolloClient.mutate({
    mutation: QUERY_ORDER_STATUS,
    variables: {
      args: {
        order_id: parseInt(orderId),
      },
    },
  });

  const status = data?.fz_invoke_action_flow?.status;
  
  if (status === '已支付') {
    // 更新用户权限
    const userId = localStorage.getItem('userId');
    if (userId) {
      const { data: userData } = await apolloClient.query({
        query: GET_USER,
        variables: { userId: parseInt(userId) },
        fetchPolicy: 'network-only',
      });
      
      if (userData?.account_by_pk) {
        useAuthStore.getState().setUser(userData.account_by_pk);
        
        // 如果用户有相应权限，跳转到主页
        if (userData.account_by_pk.can_use_ai) {
          setTimeout(() => {
            navigate('/');
          }, 2000);
        }
      }
    }
  }
};
```

### Webhook 处理
支付宝可能会通过 webhook 向 Zion 项目的后端发送支付通知，相应的 actionflow 用于处理这些 webhook 请求。因此前端不需要特殊处理 webhook 处理器的逻辑。

由于 webhook 是异步过程，在支付返回页面查询订单状态前，应该等待 2 秒左右，确保后端 webhook 有足够时间处理支付通知并更新订单状态。如果查询时订单状态尚未更新，可以提示用户稍后刷新页面或稍等片刻。
