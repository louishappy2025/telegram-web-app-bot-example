# 订单API完整指南

## 概述

本文档详细说明了订单相关的所有API接口，包括用户端和后台管理端的接口。

## 目录

1. [用户端订单API](#用户端订单api)
2. [后台管理订单API](#后台管理订单api)
3. [数据结构说明](#数据结构说明)
4. [错误码说明](#错误码说明)

## 用户端订单API

### 1. 创建订单

**接口地址：** `POST /order`

**请求体：**
```json
{
  "order": {
    "userId": 1,
    "telegramId": 123456789,
    "payMethod": 1,
    "paymentUrl": "https://example.com/payment.jpg",
    "phone": "85512345678",
    "address": "香港中环德辅道中1号",
    "remark": "请尽快配送"
  },
  "orderItems": [
    {
      "goodsId": 1,
      "quantity": 2,
      "attributes": {
        "温度": "冰",
        "糖度": "无糖",
        "大小": "大杯"
      }
    }
  ]
}
```

**响应体：**
```json
{
  "order": {
    "id": 1,
    "orderNo": "ORD20250108001",
    "userId": 1,
    "telegramId": 123456789,
    "payMethod": 1,
    "paymentUrl": "https://example.com/payment.jpg",
    "phone": "85512345678",
    "address": "香港中环德辅道中1号",
    "totalAmount": 25.50,
    "orderStatus": 0,
    "payStatus": 1,
    "remark": "请尽快配送",
    "createTime": "2025-01-08T10:30:00",
    "updateTime": "2025-01-08T10:30:00"
  },
  "orderItems": [
    {
      "orderItem": {
        "id": 1,
        "orderId": 1,
        "goodsId": 1,
        "goodsName": "奶茶",
        "goodsNo": "G001",
        "goodsIcon": "https://example.com/icon.jpg",
        "goodsImageUrl": "https://example.com/image.jpg",
        "price": 10.00,
        "actualPrice": 12.75,
        "quantity": 2,
        "subtotal": 25.50,
        "attributeAdjustment": 5.50,
        "attributeHash": "1:temperature=ice|sugar=none|size=large",
        "createTime": "2025-01-08T10:30:00"
      },
      "attributes": [
        {
          "id": 1,
          "orderItemId": 1,
          "attributeName": "温度",
          "attributeValue": "冰",
          "priceAdjustment": 0.00
        },
        {
          "id": 2,
          "orderItemId": 1,
          "attributeName": "糖度",
          "attributeValue": "无糖",
          "priceAdjustment": 0.00
        },
        {
          "id": 3,
          "orderItemId": 1,
          "attributeName": "大小",
          "attributeValue": "大杯",
          "priceAdjustment": 2.75
        }
      ],
      "attributeTexts": ["冰", "无糖", "大杯(+2.75)"]
    }
  ]
}
```

### 2. 获取订单详情

**接口地址：** `GET /order/{id}`

**路径参数：**
- `id`: 订单ID

**响应体：** 与创建订单响应相同

### 3. 获取订单列表

**接口地址：** `GET /order/list`

**响应体：**
```json
[
  {
    "id": 1,
    "orderNo": "ORD20250108001",
    "userId": 1,
    "telegramId": 123456789,
    "totalAmount": 25.50,
    "orderStatus": 0,
    "payStatus": 1,
    "createTime": "2025-01-08T10:30:00"
  }
]
```

### 4. 根据用户ID获取订单

**接口地址：** `GET /order/user/{userId}`

**路径参数：**
- `userId`: 用户ID

### 5. 根据Telegram用户ID获取订单

**接口地址：** `GET /order/telegram/{telegramId}`

**路径参数：**
- `telegramId`: Telegram用户ID

### 6. 根据订单状态获取订单

**接口地址：** `GET /order/status/{orderStatus}`

**路径参数：**
- `orderStatus`: 订单状态码

### 7. 多条件查询订单

**接口地址：** `POST /order/query`

**请求体：**
```json
{
  "userId": 1,
  "telegramId": 123456789,
  "orderStatus": 0,
  "payStatus": 1,
  "orderNo": "ORD20250108",
  "startTime": "2025-01-08T00:00:00",
  "endTime": "2025-01-08T23:59:59",
  "pageSize": 10,
  "lastId": null
}
```

**响应体：**
```json
{
  "orders": [
    {
      "order": { /* 订单信息 */ },
      "orderItems": [ /* 订单项详情数组 */ ]
    }
  ],
  "hasMore": true,
  "count": 10,
  "lastId": 100
}
```

### 8. 更新订单

**接口地址：** `PUT /order/{id}`

**路径参数：**
- `id`: 订单ID

**请求体：**
```json
{
  "orderStatus": 1,
  "remark": "更新后的备注"
}
```

### 9. 删除订单

**接口地址：** `DELETE /order/{id}`

**路径参数：**
- `id`: 订单ID

### 10. 确认收货

**接口地址：** `PUT /order/{id}/complete`

**路径参数：**
- `id`: 订单ID

**请求体：**
```json
{
  "remark": "商品已收到，满意"
}
```

### 11. 提交支付凭证

**接口地址：** `POST /order/{id}/payment-voucher`

**路径参数：**
- `id`: 订单ID

**请求体：**
```json
{
  "paymentUrl": "https://example.com/payment-proof.jpg",
  "remark": "已通过银行转账支付"
}
```

**响应体：**
```json
{
  "code": 200,
  "message": "支付凭证提交成功，订单已标记为已支付",
  "data": {
    "id": 1,
    "orderNo": "ORD20250108001",
    "payStatus": 1,
    "paymentUrl": "https://example.com/payment-proof.jpg",
    "updateTime": "2025-01-08T10:30:00"
  }
}
```

## 订单项相关API

### 1. 获取订单的所有订单项

**接口地址：** `GET /order/{orderId}/items`

**路径参数：**
- `orderId`: 订单ID

### 2. 获取特定订单项详情

**接口地址：** `GET /order/{orderId}/items/{itemId}`

**路径参数：**
- `orderId`: 订单ID
- `itemId`: 订单项ID

### 3. 为订单添加订单项

**接口地址：** `POST /order/{orderId}/items`

**路径参数：**
- `orderId`: 订单ID

**请求体：**
```json
{
  "goodsId": 1,
  "goodsName": "奶茶",
  "goodsNo": "G001",
  "price": 10.00,
  "actualPrice": 12.75,
  "quantity": 1,
  "attributeAdjustment": 2.75
}
```

### 4. 更新订单项

**接口地址：** `PUT /order/{orderId}/items/{itemId}`

**路径参数：**
- `orderId`: 订单ID
- `itemId`: 订单项ID

### 5. 删除订单项

**接口地址：** `DELETE /order/{orderId}/items/{itemId}`

**路径参数：**
- `orderId`: 订单ID
- `itemId`: 订单项ID

### 6. 批量添加订单项

**接口地址：** `POST /order/{orderId}/items/batch`

**路径参数：**
- `orderId`: 订单ID

**请求体：**
```json
[
  {
    "goodsId": 1,
    "goodsName": "奶茶",
    "quantity": 2,
    "price": 10.00,
    "actualPrice": 12.75
  },
  {
    "goodsId": 2,
    "goodsName": "咖啡",
    "quantity": 1,
    "price": 15.00,
    "actualPrice": 15.00
  }
]
```

## 后台管理订单API

### 1. 后台分页查询订单

**接口地址：** `POST /admin/api/v1/order/query`

**请求头：**
```
Authorization: Bearer <admin_token>
```

**请求体：**
```json
{
  "userId": 1,
  "telegramId": 123456789,
  "orderStatus": 0,
  "payStatus": 1,
  "orderNo": "ORD20250108",
  "startTime": "2025-01-08T00:00:00",
  "endTime": "2025-01-08T23:59:59",
  "pageSize": 10,
  "lastId": null
}
```

**响应体：**
```json
{
  "code": 200,
  "message": "查询成功",
  "data": [
    {
      "order": { /* 订单信息 */ },
      "orderItems": [ /* 订单项详情数组 */ ]
    }
  ],
  "hasMore": true,
  "lastId": 100,
  "count": 10
}
```

### 2. 获取订单列表（分页）

**接口地址：** `GET /admin/api/v1/order/list`

**请求参数：**
- `page`: 页码（默认1）
- `size`: 每页大小（默认10）

### 3. 获取订单详情

**接口地址：** `GET /admin/api/v1/order/{id}`

**路径参数：**
- `id`: 订单ID

**响应体：**
```json
{
  "code": 200,
  "message": "获取成功",
  "data": {
    "order": { /* 订单信息 */ },
    "orderItems": [ /* 订单项详情数组 */ ]
  }
}
```

### 4. 更新订单状态

**接口地址：** `PUT /admin/api/v1/order/{id}/status`

**路径参数：**
- `id`: 订单ID

**请求体：**
```json
{
  "orderStatus": 1,
  "payStatus": 1
}
```

**响应体：**
```json
{
  "code": 200,
  "message": "更新成功",
  "data": {
    "id": 1,
    "orderNo": "ORD20250108001",
    "orderStatus": 1,
    "payStatus": 1,
    "updateTime": "2025-01-08T10:30:00"
  }
}
```

### 5. 更新订单备注

**接口地址：** `PUT /admin/api/v1/order/{id}/remark`

**路径参数：**
- `id`: 订单ID

**请求体：**
```json
{
  "remark": "客户要求加急处理"
}
```

### 6. 删除订单

**接口地址：** `DELETE /admin/api/v1/order/{id}`

**路径参数：**
- `id`: 订单ID

### 7. 根据用户ID获取订单

**接口地址：** `GET /admin/api/v1/order/user/{userId}`

**路径参数：**
- `userId`: 用户ID

### 8. 根据Telegram用户ID获取订单

**接口地址：** `GET /admin/api/v1/order/telegram/{telegramId}`

**路径参数：**
- `telegramId`: Telegram用户ID

### 9. 根据订单状态获取订单

**接口地址：** `GET /admin/api/v1/order/status/{orderStatus}`

**路径参数：**
- `orderStatus`: 订单状态码

### 10. 获取订单统计信息

**接口地址：** `GET /admin/api/v1/order/statistics`

**响应体：**
```json
{
  "code": 200,
  "message": "获取成功",
  "data": {
    "totalOrders": 1000,
    "newOrders": 50,
    "preparingOrders": 30,
    "deliveringOrders": 20,
    "completedOrders": 800,
    "cancelledOrders": 80,
    "exceptionOrders": 20,
    "totalAmount": 50000.00
  }
}
```

### 11. 获取订单详情（管理后台专用）

**接口地址：** `GET /admin/api/v1/order/detail/{id}`

**路径参数：**
- `id`: 订单ID

**响应体：**
```json
{
  "code": 200,
  "message": "获取成功",
  "data": {
    "order": { /* 订单信息 */ },
    "orderItems": [ /* 订单项详情数组 */ ],
    "userInfo": {
      "telegramId": 123456789,
      "userType": "Telegram用户"
    },
    "orderStatusText": "下单成功",
    "payStatusText": "已支付",
    "payMethodText": "ABA银行转账"
  }
}
```

### 12. 客服接单

**接口地址：** `PUT /admin/api/v1/order/{id}/accept`

**路径参数：**
- `id`: 订单ID

**请求体：**
```json
{
  "remark": "已接单，正在备货"
}
```

### 13. 客服送货

**接口地址：** `PUT /admin/api/v1/order/{id}/deliver`

**路径参数：**
- `id`: 订单ID

**请求体：**
```json
{
  "remark": "商品已送达"
}
```

### 14. 设置订单为异常状态

**接口地址：** `PUT /admin/api/v1/order/{id}/exception`

**路径参数：**
- `id`: 订单ID

**请求体：**
```json
{
  "reason": "商品缺货，暂时无法配送"
}
```

## 数据结构说明

### 订单状态码

| 状态码 | 状态名称 | 说明 |
|--------|----------|------|
| 0 | 下单成功 | 订单刚创建 |
| 1 | 正在备货 | 客服已接单 |
| 2 | 已配送 | 商品已送达 |
| 3 | 已完成 | 用户确认收货 |
| 4 | 已取消 | 订单被取消 |
| 5 | 异常状态 | 订单异常 |

### 支付状态码

| 状态码 | 状态名称 | 说明 |
|--------|----------|------|
| 0 | 未支付 | 待支付 |
| 1 | 已支付 | 已完成支付 |

### 订单项属性结构

```json
{
  "id": 1,
  "orderItemId": 1,
  "attributeName": "温度",
  "attributeValue": "冰",
  "priceAdjustment": 0.00
}
```

### 订单项详情结构

```json
{
  "orderItem": {
    "id": 1,
    "orderId": 1,
    "goodsId": 1,
    "goodsName": "奶茶",
    "goodsNo": "G001",
    "goodsIcon": "https://example.com/icon.jpg",
    "goodsImageUrl": "https://example.com/image.jpg",
    "price": 10.00,
    "actualPrice": 12.75,
    "quantity": 2,
    "subtotal": 25.50,
    "attributeAdjustment": 5.50,
    "attributeHash": "1:temperature=ice|sugar=none|size=large",
    "createTime": "2025-01-08T10:30:00"
  },
  "attributes": [
    {
      "id": 1,
      "orderItemId": 1,
      "attributeName": "温度",
      "attributeValue": "冰",
      "priceAdjustment": 0.00
    }
  ],
  "attributeTexts": ["冰", "无糖", "大杯(+2.75)"]
}
```

## 错误码说明

### 通用错误码

| 错误码 | 说明 |
|--------|------|
| 200 | 成功 |
| 400 | 请求参数错误 |
| 401 | 未授权 |
| 403 | 权限不足 |
| 404 | 资源不存在 |
| 500 | 服务器内部错误 |

### 业务错误码

| 错误码 | 说明 |
|--------|------|
| 1001 | 订单不存在 |
| 1002 | 商品不存在 |
| 1003 | 商品已下架 |
| 1004 | 商品库存不足 |
| 1005 | 商品属性组合无效 |
| 1006 | 支付方式错误 |
| 1007 | 订单状态不允许此操作 |
| 1008 | 支付凭证URL不能为空 |
| 1009 | 订单已支付，无需重复提交支付凭证 |

## 注意事项

1. **订单项属性集成**：订单详情查询时会自动包含订单项属性信息，无需额外调用属性接口
2. **价格计算**：商品价格由后端从数据库获取，前端不允许传递价格信息
3. **属性价格调整**：商品属性可能包含价格调整，最终价格 = 基础价格 + 属性调整
4. **支付凭证**：非货到付款订单必须提供支付凭证URL
5. **状态流转**：订单状态有固定的流转规则，不能随意跳转
6. **权限控制**：后台管理接口需要管理员权限
7. **分页查询**：使用游标分页，通过lastId实现无限滚动 