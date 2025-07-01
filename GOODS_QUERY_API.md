# 商品查询API优化记录

## 概述

本文档记录了商品查询功能的优化过程，主要实现了基于lastId的翻页功能和商品名称模糊搜索，参考了OrderController的query方法设计模式。

## 优化背景

原有的商品列表接口`GET /goods/list`功能较为简单，只能获取所有商品，不支持分页和条件查询。为了提升用户体验和系统性能，需要实现：

1. **高性能分页**: 使用cursor分页（lastId）替代传统的offset分页
2. **条件查询**: 支持商品名称模糊搜索、分类筛选、价格区间等
3. **前端友好**: 适配无限滚动加载场景

## 主要优化内容

### 1. 新增DTO类

#### GoodsQueryRequest.java - 商品查询请求DTO
```java
@Data
public class GoodsQueryRequest {
    private String name;                    // 商品名称（模糊查询）
    private Integer categoryId;             // 分类ID
    private Integer status;                 // 商品状态（扩展字段）
    private BigDecimal minPrice;            // 最小价格
    private BigDecimal maxPrice;            // 最大价格
    private Integer lastId;                 // 最后一条记录的ID（用于分页）
    private Integer pageSize = 10;          // 每页大小，默认10条
}
```

#### GoodsQueryResponse.java - 商品查询响应DTO
```java
@Data
public class GoodsQueryResponse {
    private List<Goods> goods;              // 商品列表
    private Boolean hasMore;                // 是否有更多数据
    private Integer lastId;                 // 最后一条记录的ID（用于下次查询）
    private Integer count;                  // 本次查询的记录数
}
```

### 2. Service层扩展

#### GoodsService接口新增方法
```java
/**
 * 多条件查询商品（支持lastId翻页）
 */
GoodsQueryResponse queryGoods(GoodsQueryRequest request);
```

#### GoodsServiceImpl实现
- 使用MyBatis-Flex的QueryWrapper构建动态查询条件
- 实现lastId cursor分页逻辑
- 支持多条件组合查询

### 3. Controller层优化

#### 新增查询接口
```java
@PostMapping("/query")
public ResponseEntity<GoodsQueryResponse> queryGoods(@RequestBody GoodsQueryRequest request)
```

#### 保持向后兼容
- 保留原有的`GET /goods/list`接口
- 新接口为`POST /goods/query`，支持复杂查询参数

## 技术实现详情

### 1. Cursor分页原理

使用lastId实现cursor分页，相比传统offset分页的优势：

- **性能优势**: 避免大offset时的性能问题
- **数据一致性**: 在数据变化时不会出现重复或遗漏
- **适合实时场景**: 特别适合无限滚动加载

```sql
-- 传统分页（性能较差）
SELECT * FROM goods ORDER BY id DESC LIMIT 10 OFFSET 1000;

-- Cursor分页（性能更好）
SELECT * FROM goods WHERE id < 1234 ORDER BY id DESC LIMIT 10;
```

### 2. 查询条件构建

使用MyBatis-Flex的条件构造器实现动态查询：

```java
QueryWrapper query = QueryWrapper.create()
    .where(GOODS.NAME.like("%" + request.getName() + "%").when(StringUtils.hasText(request.getName())))
    .and(GOODS.CATEGORY_ID.eq(request.getCategoryId()).when(request.getCategoryId() != null))
    .and(GOODS.PRICE.ge(request.getMinPrice()).when(request.getMinPrice() != null))
    .and(GOODS.PRICE.le(request.getMaxPrice()).when(request.getMaxPrice() != null))
    .and(GOODS.ID.lt(request.getLastId()).when(request.getLastId() != null))
    .orderBy(GOODS.ID.desc())
    .limit(request.getPageSize() + 1);
```

### 3. hasMore判断逻辑

通过多查询一条记录来判断是否还有更多数据：

```java
// 查询 pageSize + 1 条记录
List<Goods> goodsList = list(query);

// 判断是否还有更多数据
boolean hasMore = goodsList.size() > request.getPageSize();
if (hasMore) {
    goodsList = goodsList.subList(0, request.getPageSize()); // 移除多查询的那一条
}
```

## API接口说明

### 商品查询接口

**接口地址**: `POST /goods/query`

**请求参数**:
```json
{
  "name": "苹果",              // 可选，商品名称模糊搜索
  "categoryId": 1,            // 可选，分类ID
  "minPrice": 10.0,           // 可选，最小价格
  "maxPrice": 100.0,          // 可选，最大价格
  "lastId": null,             // 可选，最后一条记录的ID（首次查询传null）
  "pageSize": 10              // 可选，每页大小，默认10
}
```

**响应格式**:
```json
{
  "goods": [
    {
      "id": 123,
      "name": "红富士苹果",
      "price": 25.80,
      "categoryId": 1,
      "description": "新鲜红富士苹果",
      "imageUrl": "https://example.com/apple.jpg",
      "createTime": "2025-01-28T10:30:00"
    }
  ],
  "hasMore": true,            // 是否有更多数据
  "lastId": 123,              // 最后一条记录的ID
  "count": 10                 // 本次返回的记录数
}
```

### 兼容性接口

**接口地址**: `GET /goods/list`

保留原有接口，返回所有商品列表，用于向后兼容。

## 前端集成指南

### 1. 首次查询
```javascript
// 首次查询
const request = {
  name: "苹果",
  pageSize: 10,
  lastId: null
};

const response = await fetch('/goods/query', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify(request)
});
```

### 2. 加载更多
```javascript
// 加载下一页
const nextRequest = {
  name: "苹果",
  pageSize: 10,
  lastId: response.lastId  // 使用上次查询返回的lastId
};
```

### 3. 无限滚动实现
```javascript
let goods = [];
let lastId = null;
let hasMore = true;

async function loadMore() {
  if (!hasMore) return;
  
  const response = await queryGoods({
    name: searchKeyword,
    lastId: lastId,
    pageSize: 10
  });
  
  goods = [...goods, ...response.goods];
  lastId = response.lastId;
  hasMore = response.hasMore;
}
```

## 性能优化建议

### 1. 数据库索引
确保以下字段有适当的索引：
```sql
-- 主要查询字段索引
CREATE INDEX idx_goods_name ON goods(name);
CREATE INDEX idx_goods_category_id ON goods(category_id);
CREATE INDEX idx_goods_price ON goods(price);

-- 复合索引（根据常用查询组合）
CREATE INDEX idx_goods_category_price ON goods(category_id, price);
```

### 2. 缓存策略
- 对于分类列表等相对稳定的数据可以考虑缓存
- 热门商品查询可以使用Redis缓存
- 搜索关键词可以考虑搜索引擎（如Elasticsearch）

### 3. 分页大小建议
- 移动端建议: 10-20条/页
- PC端建议: 20-50条/页
- 避免单次查询过多数据影响性能

## 测试场景

### 1. 功能测试
- [ ] 商品名称模糊搜索
- [ ] 分类筛选
- [ ] 价格区间筛选
- [ ] 组合条件查询
- [ ] 翻页功能
- [ ] 边界条件（空结果、单条结果等）

### 2. 性能测试
- [ ] 大数据量下的查询性能
- [ ] 并发查询测试
- [ ] 内存使用监控

### 3. 兼容性测试
- [ ] 原有list接口功能正常
- [ ] 新旧接口数据一致性

## 文件变更清单

### 新增文件
- `src/main/java/com/business/supermarket/dto/GoodsQueryRequest.java`
- `src/main/java/com/business/supermarket/dto/GoodsQueryResponse.java`
- `docs/GOODS_QUERY_API.md` (本文档)

### 修改文件
- `src/main/java/com/business/supermarket/service/GoodsService.java` - 添加queryGoods方法
- `src/main/java/com/business/supermarket/service/impl/GoodsServiceImpl.java` - 实现查询逻辑
- `src/main/java/com/business/supermarket/controller/GoodsController.java` - 添加查询接口

## 后续规划

### 1. 功能扩展
- 支持多关键词搜索
- 支持商品排序（价格、销量、评分等）
- 支持商品状态筛选（上架/下架）
- 支持库存筛选

### 2. 性能优化
- 引入搜索引擎（Elasticsearch）
- 实现查询结果缓存
- 优化数据库查询语句

### 3. 用户体验
- 搜索关键词高亮
- 搜索历史记录
- 热门搜索推荐

## 注意事项

1. **数据一致性**: cursor分页在数据变化时可能导致部分数据被跳过或重复，需要前端适当处理
2. **参数验证**: 前端需要对价格区间等参数进行合理性验证
3. **性能监控**: 建议对查询性能进行监控，及时发现慢查询
4. **安全考虑**: 对查询参数进行适当的安全校验，防止SQL注入等安全问题

---

**文档版本**: 1.0  
**创建日期**: 2025-01-28  
**最后更新**: 2025-01-28 