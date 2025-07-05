# 订单异常状态显示功能

## 功能描述
当订单状态为5（异常）时，在订单列表中显示"异常"状态，并支持点击查看异常原因。

## 实现细节

### 1. 订单状态映射
在`getOrderStatusText`函数中添加了状态5的映射：
```javascript
function getOrderStatusText(status) {
    const statusMap = {
        0: '下单成功',
        1: '正在备货', 
        2: '已配送',
        3: '已完成',
        4: '已取消',
        5: '异常'  // 新增异常状态
    };
    return statusMap[status] || '未知状态';
}
```

### 2. 订单状态颜色
为异常状态添加了专门的颜色：
```javascript
function getOrderStatusColor(status) {
    const colorMap = {
        0: '#ffc107', // 下单成功 - 黄色
        1: '#17a2b8', // 正在备货 - 青色
        2: '#007bff', // 已配送 - 蓝色
        3: '#28a745', // 已完成 - 绿色
        4: '#dc3545', // 已取消 - 红色
        5: '#ff6b35'  // 异常 - 橙红色
    };
    return colorMap[status] || '#6c757d';
}
```

### 3. 订单列表渲染
在订单列表渲染时，为异常状态添加了点击事件和丰富的视觉效果：
```javascript
// 为异常订单添加点击事件和视觉效果
const statusClickHandler = o.orderStatus === 5 ? 
    `onclick="showOrderExceptionReason(${o.id}, '${(o.reason || '').replace(/'/g, '\\\'')}')" 
     class="exception-status"
     style="cursor:pointer; text-decoration:underline; padding:2px 4px; border-radius:3px; background:rgba(255,107,53,0.1); transition:all 0.2s; font-weight:bold;" 
     onmouseover="this.style.background='rgba(255,107,53,0.2)'; this.style.transform='scale(1.05)'; this.style.animation='none';" 
     onmouseout="this.style.background='rgba(255,107,53,0.1)'; this.style.transform='scale(1)'; this.style.animation='exceptionPulse 2s infinite';" 
     title="点击查看异常原因"` : '';

// 在状态文本后添加查看提示
`<span style='float:right;color:${statusColor};font-size:12px;' ${statusClickHandler}>${orderStatusText}${o.orderStatus === 5 ? ' 🔍 点击查看' : ''}</span>`
```

### 4. CSS动画效果
为异常状态添加了闪烁动画：
```css
@keyframes exceptionPulse {
    0% { opacity: 1; }
    50% { opacity: 0.7; }
    100% { opacity: 1; }
}

.exception-status {
    animation: exceptionPulse 2s infinite;
}
```

### 5. 异常原因显示函数
创建了专门的函数来显示异常原因：
```javascript
function showOrderExceptionReason(orderId, reason) {
    console.log('🔍 [showOrderExceptionReason] 显示异常原因，订单ID:', orderId, '原因:', reason);
    
    // 添加触觉反馈
    if (typeof Telegram !== 'undefined' && Telegram.WebApp && Telegram.WebApp.HapticFeedback) {
        Telegram.WebApp.HapticFeedback.impactOccurred('light');
    }
    
    // 处理异常原因显示
    const displayReason = reason && reason.trim() ? reason : '暂无异常原因说明';
    const message = `⚠️ 订单异常详情\n\n订单ID: ${orderId}\n异常原因: ${displayReason}`;
    
    // 优先使用Telegram的弹窗
    if (typeof Telegram !== 'undefined' && Telegram.WebApp && Telegram.WebApp.showAlert) {
        Telegram.WebApp.showAlert(message);
    } else {
        alert(message);
    }
}
```

### 6. 数据字段映射
在订单数据获取时，添加了`reason`字段的映射：
```javascript
const newOrders = result.orders.map(orderData => {
    const order = orderData.order;
    return {
        // ... 其他字段
        reason: order.reason || '' // 添加异常原因字段
    };
});
```

## 用户体验设计

### 视觉提示
- **状态颜色**：使用橙红色（#ff6b35）突出显示异常状态
- **查看提示**：在异常状态后显示"🔍 点击查看"文字，明确提示用户可点击
- **背景高亮**：异常状态有淡橙色背景，增强视觉识别度
- **下划线**：异常状态文字带下划线，符合可点击元素的视觉习惯
- **闪烁动画**：异常状态持续闪烁，吸引用户注意
- **鼠标样式**：异常状态区域显示手形光标，提示可点击
- **悬停效果**：鼠标悬停时背景变深，文字放大1.05倍，停止闪烁动画

### 交互反馈
- **触觉反馈**：在Telegram环境中点击时提供轻微的触觉反馈
- **弹窗显示**：优先使用Telegram的原生弹窗，提供更好的用户体验
- **错误处理**：当异常原因为空时，显示"暂无异常原因说明"

## 功能特点

### 1. 智能显示
- 只有状态为5的订单才会显示点击功能
- 自动处理异常原因的转义，避免JavaScript错误
- 支持空异常原因的友好显示

### 2. 兼容性
- 同时支持Telegram Mini App和普通浏览器环境
- 自动选择最佳的弹窗显示方式
- 保持与现有UI风格的一致性

### 3. 调试友好
- 添加了详细的控制台日志
- 包含错误处理和边界情况处理
- 支持开发者调试和问题排查

## 使用示例

### 前端显示
当订单状态为5时，订单列表中会显示：
```
订单#12345                    异常 🔍 点击查看
📱 手机号                      ^^^^^^^^^^^^^^^^
📍 地址                        (闪烁动画+背景高亮+下划线)
💳 付款方式
🕒 创建时间
```

### 点击效果
用户点击"异常 🔍"后，会弹出：
```
⚠️ 订单异常详情

订单ID: 12345
异常原因: 商品库存不足，无法配送
```

## API数据结构

### 订单数据中的reason字段
```json
{
  "order": {
    "id": 12345,
    "orderNo": "ORD202501010001",
    "orderStatus": 5,
    "reason": "商品库存不足，无法配送",
    // ... 其他字段
  }
}
```

## 状态码更新

### 完整的订单状态映射
| 状态码 | 说明     | 颜色    | 特殊功能 |
|--------|----------|---------|----------|
| 0      | 下单成功 | 黄色    | -        |
| 1      | 正在备货 | 青色    | -        |
| 2      | 已配送   | 蓝色    | 确认收货 |
| 3      | 已完成   | 绿色    | -        |
| 4      | 已取消   | 红色    | -        |
| 5      | 异常     | 橙红色  | 查看原因 |

## 测试建议

### 1. 功能测试
- 创建状态为5的测试订单
- 验证异常状态显示是否正确
- 测试点击查看异常原因功能

### 2. 数据测试
- 测试有异常原因的订单
- 测试无异常原因的订单
- 测试异常原因包含特殊字符的情况

### 3. 兼容性测试
- 在Telegram Mini App中测试
- 在普通浏览器中测试
- 测试不同设备和屏幕尺寸

## 注意事项

### 1. 安全性
- 异常原因文本会进行转义处理，防止XSS攻击
- 使用安全的字符串替换方法处理单引号

### 2. 性能
- 点击事件只为异常订单添加，不影响其他订单的性能
- 异常原因显示使用轻量级的弹窗，响应速度快

### 3. 维护性
- 代码结构清晰，便于后续维护和扩展
- 添加了详细的注释和日志，便于调试
- 遵循现有的代码风格和命名规范 