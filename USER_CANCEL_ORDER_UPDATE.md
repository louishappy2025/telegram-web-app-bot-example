# 用户端取消订单功能调整

## 变更概述

调整用户端取消订单的业务逻辑，确保只有在未接单状态下用户才能取消订单，接单后不允许用户取消。

## 变更日期

2025-01-XX

## 变更内容

### 1. 业务逻辑调整

#### 修改前
- 用户只能在"正在备货"状态下取消订单
- 这与实际业务需求不符

#### 修改后
- 用户只能在"下单成功"状态下取消订单
- 一旦商家接单，用户不能再取消订单
- 管理员仍可以在任何状态下取消订单

### 2. 状态流转规则

#### 允许用户取消的状态
- ✅ **下单成功** (orderStatus=0) - 用户可以取消
- ❌ **正在备货** (orderStatus=1) - 用户不能取消，管理员可以取消
- ❌ **已配送** (orderStatus=2) - 用户不能取消，管理员可以取消
- ❌ **已完成** (orderStatus=3) - 任何人都不能取消
- ❌ **已取消** (orderStatus=4) - 任何人都不能取消
- ❌ **异常状态** (orderStatus=5) - 用户不能取消，管理员可以取消

### 3. 代码变更

#### 3.1 OrderController.java
**文件位置**: `src/main/java/com/business/supermarket/controller/OrderController.java`

**变更内容**:
- 修改 `cancelOrder` 方法的状态检查逻辑
- 从检查 `OrderStatus.CONFIRMED` 改为检查 `OrderStatus.PENDING_CONFIRMATION`
- 更新错误消息为 "只有在未接单状态下才可以取消订单"

```java
// 修改前
if (!OrderStatus.CONFIRMED.getCode().equals(order.getOrderStatus())) {
    // 错误消息：只有在接单状态下才可以取消订单
}

// 修改后
if (!OrderStatus.PENDING_CONFIRMATION.getCode().equals(order.getOrderStatus())) {
    // 错误消息：只有在未接单状态下才可以取消订单
}
```

#### 3.2 test-frontend-cancel-order.sh
**文件位置**: `test-frontend-cancel-order.sh`

**变更内容**:
- 更新测试脚本以验证新的业务逻辑
- 增加更多测试场景，包括各种状态下的取消测试
- 验证未接单状态下可以取消，接单后不能取消

#### 3.3 FRONTEND_CANCEL_ORDER_API.md
**文件位置**: `docs/FRONTEND_CANCEL_ORDER_API.md`

**变更内容**:
- 更新API文档以反映新的业务规则
- 明确说明只有在"下单成功"状态下才能取消
- 增加状态流转说明表格
- 更新错误响应示例

### 4. 状态流转规则保持不变

#### 4.1 OrderStatusControlServiceImpl.java
**文件位置**: `src/main/java/com/business/supermarket/service/impl/OrderStatusControlServiceImpl.java`

**说明**:
- 状态流转规则保持不变，仍然允许从任何状态流转到取消状态
- 这确保了管理员端的取消功能不受影响
- 用户端的限制通过前端控制器层面的业务逻辑实现

### 5. 影响范围

#### 5.1 用户端影响
- **前端界面**: 需要调整取消按钮的显示逻辑，只在未接单状态显示
- **用户体验**: 用户只能在下单后、商家接单前的时间窗口内取消订单
- **错误提示**: 更新错误提示信息，告知用户正确的取消时机

#### 5.2 管理员端影响
- **无影响**: 管理员仍可以在任何状态下取消订单
- **Telegram Bot**: 管理员通过Telegram Bot的取消功能不受影响

#### 5.3 系统影响
- **数据库**: 无需修改数据库结构
- **API兼容性**: 保持API接口不变，只调整业务逻辑
- **日志记录**: 取消原因仍会记录在订单备注中

### 6. 测试验证

#### 6.1 测试场景
1. **未接单状态取消** - 应该成功
2. **接单状态取消** - 应该失败
3. **配送状态取消** - 应该失败
4. **已完成状态取消** - 应该失败
5. **已取消状态再次取消** - 应该失败

#### 6.2 测试脚本
使用 `test-frontend-cancel-order.sh` 脚本进行自动化测试验证

### 7. 部署注意事项

#### 7.1 前端更新
- 需要同步更新前端代码，调整取消按钮的显示逻辑
- 更新用户提示信息

#### 7.2 文档更新
- 更新用户手册，说明新的取消规则
- 通知客服团队新的业务规则

#### 7.3 监控
- 监控取消订单的API调用情况
- 关注用户对新规则的反馈

### 8. 回滚方案

如需回滚到原有逻辑，只需要：
1. 将 `OrderController.java` 中的状态检查改回 `OrderStatus.CONFIRMED`
2. 更新错误消息
3. 恢复原有的API文档

### 9. 相关文档

- [前端取消订单API文档](./FRONTEND_CANCEL_ORDER_API.md)
- [订单状态流转文档](./ORDER_STATUS_CHANGE_NOTIFICATION.md)
- [测试脚本](../test-frontend-cancel-order.sh)

## 总结

此次变更确保了用户端取消订单功能符合实际业务需求，用户只能在商家接单前取消订单，提高了业务流程的合理性和用户体验。同时保持了管理员端功能的完整性，不影响现有的管理流程。 