# 商品分类单独显示功能API文档

## 概述

为商品分类添加了"是否单独显示"字段（`separate_display`），用于前端区分不同类型的分类展示方式。该字段为TINYINT(1)类型，默认值为0。

前端可以通过该字段自行判断分类的显示方式，无需额外的查询接口。

## 数据库变更

### 1. 字段说明
- **字段名**: `separate_display`
- **类型**: `TINYINT(1)`
- **默认值**: `0`
- **注释**: `是否单独显示：1-是，0-否（默认）`

### 2. 数据库变更SQL
```sql
-- 添加separate_display字段
ALTER TABLE goods_category 
ADD COLUMN separate_display TINYINT(1) DEFAULT 0 COMMENT '是否单独显示：1-是，0-否（默认）';

-- 更新现有数据，默认都不单独显示
UPDATE goods_category 
SET separate_display = 0 
WHERE separate_display IS NULL;
```

### 3. 验证SQL
```sql
-- 查看字段添加结果
SELECT id, name, no, separate_display, create_time 
FROM goods_category 
ORDER BY id ASC 
LIMIT 10;

-- 查看表结构
DESC goods_category;
```

## 实体类变更

### GoodsCategory实体类
```java
/**
 * 是否单独显示：1-是，0-否
 */
private Integer separateDisplay;
```

## API接口

### 管理员接口

#### 1. 获取所有商品分类（包含separateDisplay字段）

**接口地址**: `GET /admin/api/v1/goods-category/all`

**请求头**:
```
Authorization: Bearer {admin_token}
```

**响应示例**:
```json
{
  "code": 200,
  "message": "获取成功",
  "data": [
    {
      "id": 1,
      "name": "特色推荐",
      "no": "featured",
      "imageUrl": "https://example.com/featured.jpg",
      "separateDisplay": 1,
      "createTime": "2025-01-01T10:00:00",
      "updateTime": "2025-01-01T11:00:00"
    },
    {
      "id": 2,
      "name": "日用品",
      "no": "daily",
      "imageUrl": "https://example.com/daily.jpg",
      "separateDisplay": 0,
      "createTime": "2025-01-01T10:00:00",
      "updateTime": "2025-01-01T11:00:00"
    }
  ]
}
```

#### 2. 更新分类的单独显示状态

**接口地址**: `PUT /admin/api/v1/goods-category/{id}/separate-display`

**请求头**:
```
Authorization: Bearer {admin_token}
Content-Type: application/json
```

**请求参数**:
```json
{
  "separateDisplay": 1
}
```

**参数说明**:
- `separateDisplay`: 是否单独显示，必填，只能为0或1
  - `0`: 不单独显示（正常分类）
  - `1`: 单独显示（特殊展示）

**成功响应**:
```json
{
  "code": 200,
  "message": "更新成功"
}
```

**失败响应**:
```json
{
  "code": 400,
  "message": "单独显示状态必须为0（否）或1（是）"
}
```

```json
{
  "code": 404,
  "message": "商品分类不存在"
}
```

#### 3. 创建分类（支持separateDisplay字段）

**接口地址**: `POST /admin/api/v1/goods-category`

**请求参数**:
```json
{
  "name": "分类名称",
  "no": "category_code",
  "imageUrl": "https://example.com/image.jpg",
  "separateDisplay": 0
}
```

**字段说明**:
- `separateDisplay`: 可选字段，不传默认为0

#### 4. 更新分类（支持separateDisplay字段）

**接口地址**: `PUT /admin/api/v1/goods-category/{id}`

**请求参数**:
```json
{
  "name": "更新后的分类名称",
  "no": "updated_code",
  "imageUrl": "https://example.com/updated.jpg",
  "separateDisplay": 1
}
```

### 前端用户接口

#### 获取所有分类列表（包含separateDisplay字段）

**接口地址**: `GET /goodscategory/list`

**响应示例**:
```json
[
  {
    "id": 1,
    "name": "特色推荐",
    "no": "featured",
    "imageUrl": "https://example.com/featured.jpg",
    "separateDisplay": 1,
    "createTime": "2025-01-01T10:00:00",
    "updateTime": "2025-01-01T11:00:00"
  },
  {
    "id": 2,
    "name": "日用品",
    "no": "daily",
    "imageUrl": "https://example.com/daily.jpg",
    "separateDisplay": 0,
    "createTime": "2025-01-01T10:00:00",
    "updateTime": "2025-01-01T11:00:00"
  }
]
```

## 业务场景

### 1. 单独显示分类 (`separateDisplay = 1`)
- **用途**: 首页特色推荐、热门分类、促销分类等
- **展示位置**: 首页顶部推荐区、特殊展示区域

### 2. 正常显示分类 (`separateDisplay = 0`)
- **用途**: 常规商品分类浏览
- **展示位置**: 分类列表页、商品筛选

## 前端集成建议

### 1. 分类数据处理
```javascript
// 获取所有分类
const allCategories = await fetch('/goodscategory/list');

// 前端自行过滤单独显示的分类
const featuredCategories = allCategories.filter(cat => cat.separateDisplay === 1);

// 前端自行过滤正常显示的分类
const normalCategories = allCategories.filter(cat => cat.separateDisplay === 0);
```

### 2. 管理后台
```javascript
// 切换分类显示状态
const toggleSeparateDisplay = async (categoryId, separateDisplay) => {
  await fetch(`/admin/api/v1/goods-category/${categoryId}/separate-display`, {
    method: 'PUT',
    headers: {
      'Content-Type': 'application/json',
      'Authorization': `Bearer ${adminToken}`
    },
    body: JSON.stringify({ separateDisplay })
  });
};
```

### 3. 前端展示组件示例
```javascript
// React示例
const CategoryDisplay = ({ categories }) => {
  const featuredCategories = categories.filter(cat => cat.separateDisplay === 1);
  const normalCategories = categories.filter(cat => cat.separateDisplay === 0);
  
  return (
    <div>
      {/* 特色分类区域 */}
      <div className="featured-categories">
        <h2>特色推荐</h2>
        {featuredCategories.map(cat => <FeaturedCategoryCard key={cat.id} category={cat} />)}
      </div>
      
      {/* 常规分类区域 */}
      <div className="normal-categories">
        <h2>商品分类</h2>
        {normalCategories.map(cat => <NormalCategoryCard key={cat.id} category={cat} />)}
      </div>
    </div>
  );
};
```

## 数据库索引建议

为了提高查询性能，可以考虑添加索引：

```sql
-- 为separate_display字段添加索引
CREATE INDEX idx_goods_category_separate_display ON goods_category(separate_display);

-- 复合索引（如果需要按创建时间排序）
CREATE INDEX idx_goods_category_separate_display_create_time ON goods_category(separate_display, create_time);
```

## 注意事项

1. **默认值**: 新建分类默认为不单独显示（separateDisplay=0）
2. **数据一致性**: 现有分类在执行SQL后都会设置为不单独显示
3. **前端处理**: 前端根据separateDisplay字段自行过滤和展示分类
4. **权限控制**: 只有管理员可以修改分类的单独显示状态
5. **接口简化**: 减少了不必要的查询接口，前端根据字段值自行处理

## 优势

1. **简化API**: 只需要一个获取所有分类的接口
2. **前端灵活**: 前端可以根据业务需求灵活处理分类展示
3. **性能优化**: 减少了额外的数据库查询
4. **维护性好**: 减少了代码复杂度，更易维护

## 测试脚本

使用 `test-goods-category-separate-display.sh` 脚本可以测试相关功能：

```bash
chmod +x test-goods-category-separate-display.sh
./test-goods-category-separate-display.sh
``` 