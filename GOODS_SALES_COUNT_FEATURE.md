# 商品销量字段功能

## 概述

为了让前端能够显示商品的销量信息，并且让商品列表按照销量排序显示，我们为商品表添加了销量字段，并修改了相关的接口。

## 功能特性

- ✅ 商品表新增销量字段（`sales_count`）
- ✅ 管理员创建/编辑商品时可以设置销量
- ✅ 管理员商品列表按销量降序排序
- ✅ 前端查询接口按销量降序排序
- ✅ 销量字段在所有相关接口中返回
- ✅ 数据库索引优化排序性能

## 数据库变更

### 1. 添加销量字段

**执行脚本：** `src/main/resources/sql/add_goods_sales_count.sql`

```sql
-- 添加销量字段
ALTER TABLE `goods` 
ADD COLUMN `sales_count` int DEFAULT 0 COMMENT '销量' AFTER `status`;

-- 为现有商品设置默认销量
UPDATE `goods` SET `sales_count` = 0 WHERE `sales_count` IS NULL;

-- 创建销量字段的索引，用于排序优化
CREATE INDEX `idx_goods_sales_count` ON `goods` (`sales_count` DESC);
```

### 2. 字段说明

| 字段名 | 类型 | 默认值 | 说明 |
|--------|------|--------|------|
| sales_count | int | 0 | 商品销量，用于排序和展示 |

## 后端变更

### 1. 实体类修改

**文件：** `src/main/java/com/business/supermarket/entity/Goods.java`

```java
/**
 * 销量
 */
private Integer salesCount;
```

### 2. DTO修改

**文件：** `src/main/java/com/business/supermarket/dto/GoodsRequest.java`

```java
/**
 * 销量
 */
private Integer salesCount;
```

### 3. 服务层修改

**文件：** `src/main/java/com/business/supermarket/service/impl/GoodsServiceImpl.java`

#### 3.1 创建商品方法
```java
// 在createGoods方法中添加销量字段
.salesCount(request.getSalesCount() != null ? request.getSalesCount() : 0) // 默认销量为0
```

#### 3.2 更新商品方法
```java
// 在updateGoods方法中添加销量字段
.salesCount(request.getSalesCount() != null ? request.getSalesCount() : existingGoods.getSalesCount()) // 保持原销量或更新
```

#### 3.3 管理员列表查询
```java
// adminPageQuery方法中的排序逻辑
query.orderBy(GOODS.SALES_COUNT.desc(), GOODS.CREATE_TIME.desc());
```

#### 3.4 前端查询接口
```java
// queryGoods方法中的排序逻辑
query.orderBy(GOODS.SALES_COUNT.desc(), GOODS.CREATE_TIME.desc())
```

## API接口文档

### 1. 管理员商品列表接口

**接口地址：** `GET /admin/api/v1/goods/list`

**请求参数：**
| 参数名 | 类型 | 必填 | 默认值 | 说明 |
|--------|------|------|--------|------|
| pageNum | Integer | 否 | 1 | 页码 |
| pageSize | Integer | 否 | 10 | 每页数量 |
| goodsName | String | 否 | - | 商品名称（模糊搜索） |
| categoryId | Integer | 否 | - | 分类ID |
| status | Integer | 否 | - | 状态（1-上架，0-下架） |

**响应格式：**
```json
{
  "code": 200,
  "message": "获取成功",
  "data": [
    {
      "goods": {
        "id": 1,
        "name": "测试商品",
        "no": "sp1",
        "icon": "",
        "price": 2.00,
        "stock": 2,
        "description": "",
        "imageUrl": "https://example.com/image.jpg",
        "status": 1,
        "salesCount": 100,
        "createTime": "2025-01-28T10:00:00",
        "updateTime": "2025-01-28T10:00:00"
      },
      "categoryIds": [2, 3],
      "categoryNames": "饮料, 零食",
      "categoryNameList": ["饮料", "零食"]
    }
  ],
  "total": 1,
  "pageNum": 1,
  "pageSize": 10,
  "totalPages": 1
}
```

**重要变化：**
- ✅ 新增 `salesCount` 字段，表示商品销量
- ✅ 列表按销量降序排序（销量高的在前面）
- ✅ 销量相同时按创建时间降序排序

### 2. 管理员创建商品接口

**接口地址：** `POST /admin/api/v1/goods`

**请求参数：**
```json
{
  "name": "测试商品",
  "icon": "",
  "price": 2.00,
  "stock": 10,
  "categoryIds": [2, 3],
  "description": "商品描述",
  "imageUrl": "https://example.com/image.jpg",
  "status": 1,
  "salesCount": 50
}
```

**字段说明：**
| 字段名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| salesCount | Integer | 否 | 销量，不传则默认为0 |

### 3. 管理员更新商品接口

**接口地址：** `PUT /admin/api/v1/goods/{id}`

**请求参数：**
```json
{
  "name": "测试商品",
  "icon": "",
  "price": 2.00,
  "stock": 10,
  "categoryIds": [2, 3],
  "description": "商品描述",
  "imageUrl": "https://example.com/image.jpg",
  "status": 1,
  "salesCount": 150
}
```

**字段说明：**
- ✅ `salesCount` 可以更新，不传则保持原值

### 4. 前端商品查询接口

**接口地址：** `POST /goods/query`

**请求参数：**
```json
{
  "name": "商品名称",
  "categoryId": 1,
  "minPrice": 0,
  "maxPrice": 100,
  "status": 1,
  "pageSize": 20,
  "lastId": null
}
```

**响应格式：**
```json
{
  "goods": [
    {
      "id": 1,
      "name": "测试商品",
      "no": "sp1",
      "icon": "",
      "price": 2.00,
      "stock": 2,
      "description": "",
      "imageUrl": "https://example.com/image.jpg",
      "status": 1,
      "salesCount": 100,
      "createTime": "2025-01-28T10:00:00",
      "updateTime": "2025-01-28T10:00:00"
    }
  ],
  "hasMore": false,
  "count": 1,
  "lastId": 1
}
```

**重要变化：**
- ✅ 新增 `salesCount` 字段
- ✅ 按销量降序排序返回

### 5. 商品详情接口

**接口地址：** `GET /admin/api/v1/goods/{id}` 或 `GET /goods/{id}`

**响应格式：**
```json
{
  "code": 200,
  "message": "获取成功",
  "data": {
    "goods": {
      "id": 1,
      "name": "测试商品",
      "no": "sp1",
      "salesCount": 100,
      // ... 其他字段
    },
    "categoryIds": [2, 3],
    "categories": [
      {"id": 2, "name": "饮料"},
      {"id": 3, "name": "零食"}
    ]
  }
}
```

## 前端集成指南

### 1. 列表显示销量

```javascript
// 商品列表展示销量
<td>销量: {item.goods.salesCount || 0}</td>

// 或者用图标显示热销商品
{item.goods.salesCount > 100 && (
  <span className="hot-sale-badge">热销</span>
)}
```

### 2. 表单字段

```javascript
// 管理员创建/编辑商品表单
<FormItem label="销量">
  <InputNumber
    value={formData.salesCount}
    onChange={(value) => setFormData({...formData, salesCount: value})}
    min={0}
    placeholder="请输入销量"
  />
</FormItem>
```

### 3. 排序说明

```javascript
// 列表页面说明
<div className="sort-info">
  商品列表已按销量排序，销量高的商品显示在前面
</div>
```

### 4. 数据处理

```javascript
// 处理商品数据时确保销量字段存在
const processGoodsData = (goods) => ({
  ...goods,
  salesCount: goods.salesCount || 0, // 确保有默认值
  salesCountDisplay: goods.salesCount ? `${goods.salesCount}件` : '暂无销量'
});
```

## 数据验证规则

### 后端验证
- `salesCount` 必须是非负整数
- 创建时不传则默认为0
- 更新时不传则保持原值

### 前端验证建议
```javascript
const validateSalesCount = (value) => {
  if (value !== undefined && value !== null) {
    if (!Number.isInteger(value) || value < 0) {
      return '销量必须是非负整数';
    }
  }
  return null;
};
```

## 性能优化

### 1. 数据库层面
- ✅ 已创建 `idx_goods_sales_count` 降序索引
- ✅ 查询性能优化，支持大量数据的快速排序

### 2. 应用层面
- ✅ 排序逻辑：销量降序 → 创建时间降序
- ✅ 避免了复杂的聚合查询，直接使用字段排序

## 兼容性说明

### 向后兼容
- ✅ 现有接口返回格式保持不变，只是新增了 `salesCount` 字段
- ✅ 不传 `salesCount` 的请求仍然正常工作
- ✅ 老版本前端可以忽略新字段

### 数据迁移
- ✅ 现有商品销量默认设置为0
- ✅ 不影响现有业务逻辑
- ✅ 可以逐步更新真实销量数据

## 测试建议

### 1. 功能测试
- 测试创建商品时设置销量
- 测试更新商品时修改销量
- 测试列表按销量排序显示
- 测试销量为0和null的处理

### 2. 性能测试
- 测试大量商品时的排序性能
- 测试销量字段的查询效率

### 3. 兼容性测试
- 测试不传salesCount字段的请求
- 测试老版本前端的兼容性

## 注意事项

1. **数据一致性**：
   - 销量字段应该与实际订单数据保持一致
   - 建议定期从订单表同步真实销量

2. **业务逻辑**：
   - 销量可以手动设置，也可以从系统自动计算
   - 考虑是否需要销量历史记录

3. **前端展示**：
   - 可以根据销量设置不同的展示样式
   - 考虑添加"热销"、"新品"等标签

## 更新日志

**2025-01-28：**
- ✅ 数据库添加 `sales_count` 字段
- ✅ 实体类和DTO添加销量字段
- ✅ 管理员接口支持销量的增删改
- ✅ 所有列表查询接口按销量排序
- ✅ 创建数据库索引优化性能
- ✅ 保持向后兼容性 