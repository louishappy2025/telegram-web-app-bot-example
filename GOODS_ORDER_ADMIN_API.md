# 商品、商品分类、订单后台管理API文档

## 基础信息

- **基础路径**: `/admin/api/v1`
- **认证方式**: JWT Token (需要在请求头中携带 `Authorization: Bearer <token>`)
- **响应格式**: 统一JSON格式

### 响应格式说明

```json
{
  "code": 200,           // 状态码：200成功，400参数错误，401未认证，404未找到，500服务器错误
  "message": "操作成功",  // 响应消息
  "data": {},           // 响应数据
  "total": 100,         // 总数（分页时）
  "pageNum": 1,         // 当前页码（分页时）
  "pageSize": 10,       // 每页大小（分页时）
  "totalPages": 10      // 总页数（分页时）
}
```

---

## 1. 商品管理API

### 1.1 获取商品列表（分页查询）

**接口地址**: `GET /admin/api/v1/goods/list`

**请求参数**:

- `pageNum` (可选): 页码，默认1
- `pageSize` (可选): 每页大小，默认10
- `goodsName` (可选): 商品名称，支持模糊查询
- `categoryId` (可选): 分类ID

**请求示例**:

```
GET /admin/api/v1/goods/list?pageNum=1&pageSize=10&goodsName=苹果&categoryId=1
```

**响应示例**:

```json
{
  "code": 200,
  "message": "获取成功",
  "data": [
    {
      "id": 1,
      "name": "红富士苹果",
      "icon": "apple.png",
      "price": 5.99,
      "stock": 100,
      "categoryId": 1,
      "description": "新鲜红富士苹果",
      "imageUrl": "https://example.com/apple.jpg",
      "createTime": "2025-01-01T10:00:00",
      "updateTime": "2025-01-01T10:00:00"
    }
  ],
  "total": 50,
  "pageNum": 1,
  "pageSize": 10,
  "totalPages": 5
}
```

### 1.2 根据ID获取商品详情

**接口地址**: `GET /admin/api/v1/goods/{id}`

**请求参数**:

- `id`: 商品ID

**请求示例**:

```
GET /admin/api/v1/goods/1
```

**响应示例**:

```json
{
  "code": 200,
  "message": "获取成功",
  "data": {
    "id": 1,
    "name": "红富士苹果",
    "icon": "apple.png",
    "price": 5.99,
    "stock": 100,
    "categoryId": 1,
    "description": "新鲜红富士苹果",
    "imageUrl": "https://example.com/apple.jpg",
    "createTime": "2025-01-01T10:00:00",
    "updateTime": "2025-01-01T10:00:00"
  }
}
```

### 1.3 新增商品

**接口地址**: `POST /admin/api/v1/goods`

**请求参数**:

```json
{
  "name": "红富士苹果",
  "icon": "apple.png",
  "price": 5.99,
  "stock": 100,
  "categoryId": 1,
  "description": "新鲜红富士苹果",
  "imageUrl": "https://example.com/apple.jpg"
}
```

**响应示例**:

```json
{
  "code": 200,
  "message": "创建成功",
  "data": {
    "id": 1,
    "name": "红富士苹果",
    "icon": "apple.png",
    "price": 5.99,
    "stock": 100,
    "categoryId": 1,
    "description": "新鲜红富士苹果",
    "imageUrl": "https://example.com/apple.jpg",
    "createTime": "2025-01-01T10:00:00",
    "updateTime": "2025-01-01T10:00:00"
  }
}
```

### 1.4 更新商品

**接口地址**: `PUT /admin/api/v1/goods/{id}`

**请求参数**:

```json
{
  "name": "红富士苹果（更新）",
  "price": 6.99,
  "stock": 80,
  "description": "新鲜红富士苹果，品质优良"
}
```

**响应示例**:

```json
{
  "code": 200,
  "message": "更新成功"
}
```

### 1.5 删除商品

**接口地址**: `DELETE /admin/api/v1/goods/{id}`

**请求示例**:

```
DELETE /admin/api/v1/goods/1
```

**响应示例**:

```json
{
  "code": 200,
  "message": "删除成功"
}
```

### 1.6 根据分类ID获取商品列表

**接口地址**: `GET /admin/api/v1/goods/category/{categoryId}`

**请求示例**:

```
GET /admin/api/v1/goods/category/1
```

**响应示例**:

```json
{
  "code": 200,
  "message": "获取成功",
  "data": [
    {
      "id": 1,
      "name": "红富士苹果",
      "price": 5.99,
      "stock": 100,
      "categoryId": 1
    }
  ],
  "total": 1
}
```

### 1.7 获取所有商品（用于下拉选择）

**接口地址**: `GET /admin/api/v1/goods/all`

**响应示例**:

```json
{
  "code": 200,
  "message": "获取成功",
  "data": [
    {
      "id": 1,
      "name": "红富士苹果",
      "price": 5.99,
      "categoryId": 1
    }
  ]
}
```

---

## 2. 商品分类管理API

### 2.1 获取商品分类列表（分页查询）

**接口地址**: `GET /admin/api/v1/goods-category/list`

**请求参数**:

- `pageNum` (可选): 页码，默认1
- `pageSize` (可选): 每页大小，默认10
- `categoryName` (可选): 分类名称，支持模糊查询

**请求示例**:

```
GET /admin/api/v1/goods-category/list?pageNum=1&pageSize=10&categoryName=水果
```

**响应示例**:

```json
{
  "code": 200,
  "message": "获取成功",
  "data": [
    {
      "id": 1,
      "name": "水果",
      "description": "新鲜水果",
      "createTime": "2025-01-01T10:00:00",
      "updateTime": "2025-01-01T10:00:00"
    }
  ],
  "total": 10,
  "pageNum": 1,
  "pageSize": 10,
  "totalPages": 1
}
```

### 2.2 根据ID获取商品分类详情

**接口地址**: `GET /admin/api/v1/goods-category/{id}`

**请求示例**:

```
GET /admin/api/v1/goods-category/1
```

**响应示例**:

```json
{
  "code": 200,
  "message": "获取成功",
  "data": {
    "id": 1,
    "name": "水果",
    "description": "新鲜水果",
    "createTime": "2025-01-01T10:00:00",
    "updateTime": "2025-01-01T10:00:00"
  }
}
```

### 2.3 新增商品分类

**接口地址**: `POST /admin/api/v1/goods-category`

**请求参数**:

```json
{
  "name": "蔬菜",
  "description": "新鲜蔬菜"
}
```

**响应示例**:

```json
{
  "code": 200,
  "message": "创建成功",
  "data": {
    "id": 2,
    "name": "蔬菜",
    "description": "新鲜蔬菜",
    "createTime": "2025-01-01T10:00:00",
    "updateTime": "2025-01-01T10:00:00"
  }
}
```

### 2.4 更新商品分类

**接口地址**: `PUT /admin/api/v1/goods-category/{id}`

**请求参数**:

```json
{
  "name": "新鲜蔬菜",
  "description": "有机新鲜蔬菜"
}
```

**响应示例**:

```json
{
  "code": 200,
  "message": "更新成功"
}
```

### 2.5 删除商品分类

**接口地址**: `DELETE /admin/api/v1/goods-category/{id}`

**请求示例**:

```
DELETE /admin/api/v1/goods-category/1
```

**响应示例**:

```json
{
  "code": 200,
  "message": "删除成功"
}
```

**注意**: 如果分类下还有商品，会返回错误：

```json
{
  "code": 400,
  "message": "该分类下还有商品，无法删除"
}
```

### 2.6 获取所有商品分类（用于下拉选择）

**接口地址**: `GET /admin/api/v1/goods-category/all`

**响应示例**:

```json
{
  "code": 200,
  "message": "获取成功",
  "data": [
    {
      "id": 1,
      "name": "水果"
    },
    {
      "id": 2,
      "name": "蔬菜"
    }
  ]
}
```

---

## 3. 订单管理API

**排序规则**: 所有订单列表接口均按照创建时间倒序排列（最新订单在前），确保管理员能够优先看到最新的订单信息。

### 3.1 后台分页查询订单（包含订单详情）

**接口地址**: `POST /admin/api/v1/order/query`

**请求参数**:

```json
{
  "pageNum": 1,
  "pageSize": 10,
  "orderNo": "ORD202501010001",
  "orderStatus": 1,
  "payStatus": 1,
  "telegramId": 123456789
}
```

**响应示例**:

```json
{
  "code": 200,
  "message": "查询成功",
  "data": [
    {
      "order": {
        "id": 1,
        "orderNo": "ORD202501010001",
        "telegramId": 123456789,
        "totalAmount": 25.99,
        "orderStatus": 1,
        "payStatus": 1,
        "payMethod": 1,
        "createTime": "2025-01-01T10:00:00",
        "updateTime": "2025-01-01T10:00:00"
      },
      "orderItems": [
        {
          "id": 1,
          "orderId": 1,
          "goodsId": 1,
          "goodsName": "红富士苹果",
          "goodsIcon": "apple.png",
          "price": 5.99,
          "quantity": 2,
          "subtotal": 11.98
        }
      ]
    }
  ],
  "hasMore": true,
  "lastId": 1,
  "count": 10
}
```

### 3.2 获取订单列表（分页查询）

**接口地址**: `GET /admin/api/v1/order/list`

**请求参数**:

- `pageNum` (可选): 页码，默认1
- `pageSize` (可选): 每页大小，默认10
- `orderNo` (可选): 订单号，支持模糊查询
- `orderStatus` (可选): 订单状态（0下单成功，1正在备货，2已配送，3已完成，4已取消）
- `payStatus` (可选): 支付状态（0未支付，1已支付）
- `telegramId` (可选): Telegram用户ID

**请求示例**:

```
GET /admin/api/v1/order/list?pageNum=1&pageSize=10&orderStatus=1&payStatus=1
```

**响应示例**:

```json
{
  "code": 200,
  "message": "获取成功",
  "data": [
    {
      "id": 1,
      "orderNo": "ORD202501010001",
      "telegramId": 123456789,
      "totalAmount": 25.99,
      "orderStatus": 1,
      "payStatus": 1,
      "payMethod": 1,
      "createTime": "2025-01-01T10:00:00",
      "updateTime": "2025-01-01T10:00:00"
    }
  ],
  "total": 50,
  "pageNum": 1,
  "pageSize": 10,
  "totalPages": 5
}
```

### 3.3 根据ID获取订单详情（包含订单项）

**接口地址**: `GET /admin/api/v1/order/{id}`

**请求示例**:

```
GET /admin/api/v1/order/1
```

**响应示例**:

```json
{
  "code": 200,
  "message": "获取成功",
  "data": {
    "order": {
      "id": 1,
      "orderNo": "ORD202501010001",
      "telegramId": 123456789,
      "totalAmount": 25.99,
      "orderStatus": 1,
      "payStatus": 1,
      "payMethod": 1,
      "phone": "13800138000",
      "address": "北京市朝阳区xxx街道xxx号",
      "remark": "请尽快配送",
      "createTime": "2025-01-01T10:00:00",
      "updateTime": "2025-01-01T10:00:00"
    },
    "orderItems": [
      {
        "id": 1,
        "orderId": 1,
        "goodsId": 1,
        "goodsName": "红富士苹果",
        "goodsIcon": "apple.png",
        "goodsImageUrl": "https://example.com/apple.jpg",
        "price": 5.99,
        "quantity": 2,
        "subtotal": 11.98,
        "createTime": "2025-01-01T10:00:00"
      },
      {
        "id": 2,
        "orderId": 1,
        "goodsId": 2,
        "goodsName": "新鲜橙子",
        "goodsIcon": "orange.png",
        "goodsImageUrl": "https://example.com/orange.jpg",
        "price": 3.99,
        "quantity": 3,
        "subtotal": 11.97,
        "createTime": "2025-01-01T10:00:00"
      }
    ]
  }
}
```

**字段说明**:

- `order`: 订单基本信息
    - `id`: 订单ID
    - `orderNo`: 订单号
    - `telegramId`: Telegram用户ID
    - `totalAmount`: 订单总金额
    - `orderStatus`: 订单状态（0下单成功，1正在备货，2已配送，3已完成，4已取消）
    - `payStatus`: 支付状态（0未支付，1已支付）
    - `payMethod`: 支付方式（1微信支付，2支付宝，3现金）
    - `phone`: 收货手机号
    - `address`: 收货地址
    - `remark`: 订单备注
    - `createTime`: 创建时间
    - `updateTime`: 更新时间

- `orderItems`: 订单项列表
    - `id`: 订单项ID
    - `orderId`: 订单ID
    - `goodsId`: 商品ID
    - `goodsName`: 商品名称
    - `goodsIcon`: 商品图标
    - `goodsImageUrl`: 商品图片URL（用于显示商品图片）
    - `price`: 商品单价
    - `quantity`: 购买数量
    - `subtotal`: 小计金额
    - `createTime`: 创建时间

### 3.4 获取订单详情（管理后台专用）

**接口地址**: `GET /admin/api/v1/order/detail/{id}`

**说明**: 此接口专门为管理后台设计，提供更丰富的订单详情信息，包括状态文本说明和用户信息。

**请求示例**:

```
GET /admin/api/v1/order/detail/1
```

**响应示例**:

```json
{
  "code": 200,
  "message": "获取成功",
  "data": {
    "order": {
      "id": 1,
      "orderNo": "ORD202501010001",
      "telegramId": 123456789,
      "totalAmount": 25.99,
      "orderStatus": 1,
      "payStatus": 1,
      "payMethod": 1,
      "phone": "13800138000",
      "address": "北京市朝阳区xxx街道xxx号",
      "remark": "请尽快配送",
      "createTime": "2025-01-01T10:00:00",
      "updateTime": "2025-01-01T10:00:00"
    },
    "orderItems": [
      {
        "id": 1,
        "orderId": 1,
        "goodsId": 1,
        "goodsName": "红富士苹果",
        "goodsIcon": "apple.png",
        "goodsImageUrl": "https://example.com/apple.jpg",
        "price": 5.99,
        "quantity": 2,
        "subtotal": 11.98,
        "createTime": "2025-01-01T10:00:00"
      },
      {
        "id": 2,
        "orderId": 1,
        "goodsId": 2,
        "goodsName": "新鲜橙子",
        "goodsIcon": "orange.png",
        "goodsImageUrl": "https://example.com/orange.jpg",
        "price": 3.99,
        "quantity": 3,
        "subtotal": 11.97,
        "createTime": "2025-01-01T10:00:00"
      }
    ],
    "userInfo": {
      "telegramId": 123456789,
      "userType": "Telegram用户"
    },
    "orderStatusText": "已完成",
    "payStatusText": "已支付",
    "payMethodText": "微信支付"
  }
}
```

**字段说明**:

- `order`: 订单基本信息（同上）
- `orderItems`: 订单项列表（同上）
- `userInfo`: 用户信息
    - `telegramId`: Telegram用户ID
    - `userType`: 用户类型
- `orderStatusText`: 订单状态文本说明
- `payStatusText`: 支付状态文本说明
- `payMethodText`: 支付方式文本说明

### 3.5 更新订单状态

**接口地址**: `PUT /admin/api/v1/order/{id}/status`

**请求参数**:

```json
{
  "orderStatus": 1,
  "payStatus": 1
}
```

**响应示例**:

```json
{
  "code": 200,
  "message": "更新成功",
  "data": {
    "id": 1,
    "orderNo": "ORD202501010001",
    "orderStatus": 1,
    "payStatus": 1,
    "updateTime": "2025-01-01T10:00:00"
  }
}
```

### 3.6 更新订单备注

**接口地址**: `PUT /admin/api/v1/order/{id}/remark`

**功能描述**: 管理员专用接口，用于添加、修改或清空指定订单的备注信息。

**请求参数**:

```json
{
  "remark": "客户要求加急配送，请优先处理"
}
```

**请求体字段说明**:
- `remark` (可选): 订单备注，字符串类型
  - 可以为空字符串（清空备注）
  - 可以为null（清空备注）
  - 最大长度：500个字符

**响应示例**:

**成功响应**:
```json
{
  "code": 200,
  "message": "备注更新成功",
  "data": {
    "id": 123,
    "orderNo": "ORD20250101001",
    "remark": "客户要求加急配送，请优先处理",
    "updateTime": "2025-01-01T15:30:00"
  }
}
```

**订单不存在**:
```json
{
  "code": 404,
  "message": "订单不存在"
}
```

**参数错误**:
```json
{
  "code": 400,
  "message": "订单备注长度不能超过500个字符"
}
```

### 3.7 删除订单（包含订单项）

**接口地址**: `DELETE /admin/api/v1/order/{id}`

**请求示例**:

```
DELETE /admin/api/v1/order/1
```

**响应示例**:

```json
{
  "code": 200,
  "message": "删除成功"
}
```

### 3.8 根据用户ID获取订单列表

**接口地址**: `GET /admin/api/v1/order/user/{userId}`

**请求示例**:

```
GET /admin/api/v1/order/user/1
```

**响应示例**:

```json
{
  "code": 200,
  "message": "获取成功",
  "data": [
    {
      "id": 1,
      "orderNo": "ORD202501010001",
      "totalAmount": 25.99,
      "orderStatus": 1,
      "payStatus": 1
    }
  ],
  "total": 1
}
```

### 3.9 根据Telegram用户ID获取订单列表

**接口地址**: `GET /admin/api/v1/order/telegram/{telegramId}`

**请求示例**:

```
GET /admin/api/v1/order/telegram/123456789
```

**响应示例**:

```json
{
  "code": 200,
  "message": "获取成功",
  "data": [
    {
      "id": 1,
      "orderNo": "ORD202501010001",
      "totalAmount": 25.99,
      "orderStatus": 1,
      "payStatus": 1
    }
  ],
  "total": 1
}
```

### 3.10 根据订单状态获取订单列表

**接口地址**: `GET /admin/api/v1/order/status/{orderStatus}`

**请求示例**:

```
GET /admin/api/v1/order/status/1
```

**响应示例**:

```json
{
  "code": 200,
  "message": "获取成功",
  "data": [
    {
      "id": 1,
      "orderNo": "ORD202501010001",
      "totalAmount": 25.99,
      "orderStatus": 1,
      "payStatus": 1
    }
  ],
  "total": 10
}
```

### 3.11 获取订单统计信息

**接口地址**: `GET /admin/api/v1/order/statistics`

**响应示例**:

```json
{
  "code": 200,
  "message": "获取成功",
  "data": {
    "totalOrders": 100,
    "newOrders": 20,
    "preparingOrders": 15,
    "deliveringOrders": 10,
    "completedOrders": 50,
    "cancelledOrders": 5,
    "totalAmount": 2599.99
  }
}
```

### 3.12 订单流转操作（接单/送货/送货完成）

#### 3.12.1 客服接单（订单进入"正在备货"）

**接口地址**: `PUT /admin/api/v1/order/{id}/accept`

**请求参数**:
```json
{
  "remark": "可选，操作备注"
}
```

**响应示例**:
```json
{
  "code": 200,
  "message": "接单成功，订单进入正在备货状态",
  "data": {
    "id": 1,
    "orderNo": "ORD202501010001",
    "orderStatus": 1,
    "updateTime": "2025-01-01T10:00:00"
  }
}
```

#### 3.12.2 客服送货（订单进入"已配送"）

**接口地址**: `PUT /admin/api/v1/order/{id}/deliver`

**请求参数**:
```json
{
  "remark": "可选，操作备注"
}
```

**响应示例**:
```json
{
  "code": 200,
  "message": "送货操作成功，订单进入已配送状态",
  "data": {
    "id": 1,
    "orderNo": "ORD202501010001",
    "orderStatus": 2,
    "updateTime": "2025-01-01T10:10:00"
  }
}
```

#### 3.12.3 用户确认收货（订单进入"已完成"）

**接口地址**: `PUT /order/{id}/complete`

**请求参数**:
```json
{
  "remark": "可选，操作备注"
}
```

**响应示例**:
```json
{
  "id": 1,
  "orderNo": "ORD202501010001",
  "orderStatus": 3,
  "updateTime": "2025-01-01T10:20:00"
}
```

**说明**:
- 所有流转接口均支持 remark 字段，便于记录操作备注。
- 状态流转顺序：下单成功 → 正在备货 → 已配送 → 已完成。
- 管理端和用户端接口风格与现有API保持一致。

---

## 状态码说明

| 状态码 | 说明          |
|-----|-------------|
| 200 | 操作成功        |
| 400 | 参数错误或业务逻辑错误 |
| 401 | 未认证或token无效 |
| 404 | 资源不存在       |
| 500 | 服务器内部错误     |

## 订单状态码与含义

| 状态码 | 说明         |
|--------|--------------|
| 0      | 下单成功     |
| 1      | 正在备货     |
| 2      | 已配送       |
| 3      | 已完成       |
| 4      | 已取消       |
| 5      | 异常         |

> 所有接口返回的 orderStatus 字段均以此表为准，前端展示时请使用本表说明。

## 支付状态说明

| 状态值 | 说明  |
|-----|-----|
| 0   | 未支付 |
| 1   | 已支付 |

## 支付方式说明

| 方式值 | 说明   |
|-----|------|
| 1   | 微信支付 |
| 2   | 支付宝  |
| 3   | 现金   |

---

（下方所有涉及订单状态的API响应示例、参数说明，均已与上表保持一致） 