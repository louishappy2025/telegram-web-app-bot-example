# 订单备注字段重构说明

## 修改概述

将下单页面的订单备注字段分离为两个独立的字段：
1. **付款参数** (`paymentRemark`) - 存储付款相关的自动生成参数
2. **订单备注** (`remark`) - 用户手动输入的订单备注信息

## 具体修改内容

### 1. HTML 结构调整

**新增付款参数字段：**
```html
<div class="form-group" style="align-items: flex-start;">
    <label style="line-height: 36px;">付款参数：</label>
    <textarea id="payment-remark" placeholder="付款相关参数（自动生成）" rows="2" 
              style="..." readonly></textarea>
</div>
```

**保留订单备注字段：**
```html
<div class="form-group" style="align-items: flex-start;">
    <label style="line-height: 36px;">订单备注：</label>
    <textarea id="user-remark" placeholder="请输入订单备注（可选）" rows="2" 
              style="..."></textarea>
</div>
```

### 2. JavaScript 逻辑更新

#### 2.1 订单提交逻辑
```javascript
// 获取两个字段的值
const remark = document.getElementById('user-remark')?.value?.trim() || '';
const paymentRemark = document.getElementById('payment-remark')?.value?.trim() || '';

// 请求数据结构
const requestData = {
    order: {
        // ... 其他字段
        remark: remark,              // 用户输入的订单备注
        paymentRemark: paymentRemark // 系统生成的付款参数
    },
    orderItems: orderItems
};
```

#### 2.2 货到付款逻辑更新
- `updateCodRemark()` 函数现在将生成的付款信息写入 `payment-remark` 字段
- 用户可以在 `user-remark` 字段中输入自己的订单备注
- 两个字段相互独立，互不影响

#### 2.3 重置逻辑
新增 `resetPaymentRemark()` 函数：
```javascript
function resetPaymentRemark() {
    const paymentRemarkInput = document.getElementById('payment-remark');
    if (paymentRemarkInput) {
        paymentRemarkInput.value = '';
    }
}
```

在以下场景中调用重置：
- 付款方式变更时
- 切换到下单页面时
- 重置货到付款表单时

### 3. 字段特性

#### 付款参数字段 (`payment-remark`)
- **只读属性**：用户无法直接编辑
- **自动生成**：系统根据付款方式自动填充
- **货到付款专用**：主要用于存储货到付款的现金详情
- **样式区别**：背景色为灰色 (`#f5f5f5`)，表示只读状态

#### 订单备注字段 (`user-remark`)
- **用户输入**：用户可以自由输入订单相关备注
- **可选字段**：不是必填项
- **通用字段**：适用于所有付款方式
- **正常样式**：白色背景，可编辑状态

### 4. 后端接口变更

#### 新增字段
- `paymentRemark`: 付款参数（字符串类型，可选）

#### 保留字段
- `remark`: 订单备注（字符串类型，可选）

### 5. 用户体验改进

#### 5.1 明确的字段分离
- 用户清楚地知道哪个是系统生成的付款参数
- 用户可以专门在订单备注中输入自己的需求

#### 5.2 货到付款流程优化
- 付款参数自动生成，如：`货到付款：支付美金 $10.00 + 支付柬币 5000៛，找零 1000៛`
- 用户可以在订单备注中添加额外信息，如：`请在下午2点后送达`

#### 5.3 视觉区分
- 付款参数字段为只读状态，背景色区分
- 订单备注字段为正常输入状态

### 6. 兼容性说明

#### 6.1 向后兼容
- 保留原有的 `remark` 字段，确保现有订单数据不受影响
- 新增的 `paymentRemark` 字段为可选，不会影响现有API

#### 6.2 数据迁移
- 现有订单的备注数据保持不变
- 新订单开始使用分离的字段结构

### 7. 技术实现细节

#### 7.1 字段验证
- 两个字段都是可选的，不进行必填验证
- 付款参数字段由系统控制，不需要用户验证

#### 7.2 数据传输
```javascript
// 订单提交时的数据结构
{
    "order": {
        "remark": "用户输入的订单备注",
        "paymentRemark": "货到付款：支付美金 $10.00，找零 1000៛",
        // ... 其他字段
    }
}
```

#### 7.3 错误处理
- 如果付款参数生成失败，不影响订单提交
- 用户备注输入错误不影响付款参数

## 总结

这次重构实现了：
1. **职责分离**：付款参数和用户备注各司其职
2. **用户友好**：界面更清晰，用户体验更好
3. **系统健壮**：自动化程度更高，减少用户错误
4. **扩展性强**：为未来的功能扩展预留空间

通过这种设计，系统可以更好地管理付款相关信息，同时保留用户自定义备注的灵活性。 