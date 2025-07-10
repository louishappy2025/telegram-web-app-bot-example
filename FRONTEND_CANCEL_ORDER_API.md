# 前端取消订单接口

## 接口信息

- **接口路径**: `PUT /order/{id}/cancel`
- **请求方式**: PUT
- **功能**: 取消指定订单
- **限制**: 只有在接单状态下才可以取消

## 请求参数

| 参数名 | 类型 | 位置 | 必填 | 描述 |
|--------|------|------|------|------|
| id | Long | 路径 | 是 | 订单ID |
| cancelReason | String | 查询参数 | 否 | 取消原因 |

## 业务规则

**只有订单状态为"正在备货"(orderStatus=1)时才可以取消**

## 响应格式

### 成功响应 (200)
```json
{
  "success": true,
  "message": "订单取消成功",
  "order": {
    "id": 123,
    "orderNo": "ORD202501141530450001",
    "orderStatus": 4
  }
}
```

### 错误响应 (400)
```json
{
  "success": false,
  "message": "只有在接单状态下才可以取消订单，当前状态：下单成功"
}
```

## 请求示例

```bash
# 基本取消
curl -X PUT "http://localhost:8080/order/123/cancel"

# 带取消原因
curl -X PUT "http://localhost:8080/order/123/cancel?cancelReason=用户主动取消"
```

```javascript
// JavaScript示例
fetch(`/order/${orderId}/cancel?cancelReason=${encodeURIComponent(reason)}`, {
  method: 'PUT'
})
.then(response => response.json())
.then(data => {
  if (data.success) {
    alert('订单取消成功');
  } else {
    alert('取消失败：' + data.message);
  }
});
``` 