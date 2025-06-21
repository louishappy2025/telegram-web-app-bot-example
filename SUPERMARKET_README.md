# Telegram WebApp 超市下单系统

基于Telegram WebApp API开发的完整超市下单系统，支持商品浏览、购物车管理、订单创建和查看等功能。

## 🚀 功能特性

### 核心功能
- **商品浏览**: 展示商品列表，包含商品图片、名称、价格和描述
- **购物车管理**: 添加/删除商品，实时更新数量和总价
- **订单系统**: 完整的下单流程，包含收货信息填写
- **订单查看**: 查看历史订单和订单状态
- **数据持久化**: 使用Telegram CloudStorage保存购物车和订单数据

### 用户体验
- **主题适配**: 自动适配Telegram的深色/浅色主题
- **触觉反馈**: 操作时提供触觉反馈
- **响应式设计**: 适配不同屏幕尺寸
- **原生体验**: 集成Telegram原生UI组件

### 技术特性
- **Telegram WebApp API**: 完整集成Telegram WebApp功能
- **CloudStorage**: 数据持久化存储
- **主题系统**: 动态主题切换
- **事件处理**: 完整的生命周期管理

## 📱 界面截图

### 商品页面
- 网格布局展示商品
- 每个商品卡片包含图片、名称、价格
- 一键添加到购物车

### 购物车页面
- 显示已选商品列表
- 实时计算总价
- 数量调整功能
- 结算按钮

### 订单页面
- 订单历史列表
- 订单详细信息
- 订单状态显示

## 🛠️ 技术实现

### 文件结构
```
telegram-web-app-bot-example/
├── supermarket-app.html          # 基础版本
├── supermarket-app-enhanced.html # 增强版本
├── index.html                    # 原始示例文件
└── README.md                     # 项目说明
```

### 核心组件

#### 1. SupermarketApp 主对象
```javascript
const SupermarketApp = {
    products: [],    // 商品数据
    cart: {},        // 购物车数据
    orders: [],      // 订单数据
    
    init(),          // 初始化应用
    setupTelegramWebApp(), // 设置Telegram WebApp
    loadData(),      // 加载数据
    saveData(),      // 保存数据
    // ... 其他方法
};
```

#### 2. 数据管理
- **商品数据**: 静态商品列表，包含ID、名称、价格、图片
- **购物车数据**: 存储在Telegram CloudStorage中
- **订单数据**: 包含完整的订单信息，存储在CloudStorage中

#### 3. UI组件
- **标签页导航**: 商品、购物车、订单三个主要页面
- **商品网格**: 响应式网格布局
- **购物车列表**: 可编辑的商品列表
- **订单卡片**: 详细的订单信息展示

## 🚀 使用方法

### 1. 部署到服务器
将HTML文件部署到支持HTTPS的Web服务器上。

### 2. 配置Telegram Bot
在BotFather中设置WebApp URL：
```
/setmenubutton
选择你的机器人
输入WebApp URL: https://your-domain.com/supermarket-app.html
```

### 3. 用户使用流程
1. 用户在Telegram中打开机器人
2. 点击菜单按钮进入WebApp
3. 浏览商品并添加到购物车
4. 在购物车中调整数量
5. 点击结算填写收货信息
6. 确认下单完成

## 📋 功能详细说明

### 商品管理
- **商品列表**: 8种常见商品，包含水果、乳制品、肉类等
- **商品信息**: 名称、价格、图片、描述
- **添加商品**: 点击"+"按钮添加到购物车
- **移除商品**: 点击"-"按钮从购物车移除

### 购物车功能
- **实时更新**: 添加/删除商品时实时更新界面
- **数量控制**: 可以调整每个商品的数量
- **总价计算**: 自动计算购物车总价
- **数据持久化**: 购物车数据保存在CloudStorage中

### 订单系统
- **下单流程**: 确认购物车 → 填写信息 → 创建订单
- **收货信息**: 收货人姓名、手机号码、地址
- **订单状态**: 待处理、已完成等状态
- **订单历史**: 查看所有历史订单

### 数据存储
- **CloudStorage**: 使用Telegram提供的云存储
- **数据同步**: 自动同步购物车和订单数据
- **数据安全**: 数据存储在Telegram服务器上

## 🎨 主题系统

### 自动适配
- 根据Telegram主题自动切换深色/浅色模式
- 使用CSS变量实现主题切换
- 支持所有Telegram主题颜色

### 颜色变量
```css
--bg-color: var(--tg-theme-bg-color, #fff)
--text-color: var(--tg-theme-text-color, #222)
--button-color: var(--tg-theme-button-color, #50a8eb)
--hint-color: var(--tg-theme-hint-color, #ccc)
```

## 🔧 自定义配置

### 添加新商品
在`SupermarketApp.products`数组中添加新商品：
```javascript
{
    id: 9,
    name: '新商品名称',
    price: 19.99,
    image: '🆕',
    description: '商品描述'
}
```

### 修改主题
调整CSS变量来自定义主题：
```css
.product-card {
    background: var(--custom-bg-color, #fff);
    border-color: var(--custom-border-color, #ccc);
}
```

### 添加支付方式
在结算流程中添加更多支付选项：
```javascript
paymentMethods: [
    { id: 'wechat', name: '微信支付', icon: '💳' },
    { id: 'alipay', name: '支付宝', icon: '💰' },
    { id: 'cash', name: '货到付款', icon: '💵' }
]
```

## 📊 性能优化

### 代码优化
- 使用事件委托减少事件监听器
- 避免频繁的DOM操作
- 合理使用缓存机制

### 用户体验
- 添加加载状态提示
- 操作反馈（触觉反馈、视觉反馈）
- 错误处理和用户提示

## 🔒 安全考虑

### 数据验证
- 手机号格式验证
- 必填字段检查
- 数据长度限制

### 权限管理
- 使用Telegram WebApp权限系统
- 最小权限原则
- 用户数据保护

## 🚀 扩展功能

### 可能的扩展
1. **商品分类**: 按类别组织商品
2. **搜索功能**: 商品搜索和筛选
3. **用户评价**: 商品评价系统
4. **优惠券**: 折扣和优惠券功能
5. **配送跟踪**: 订单配送状态跟踪
6. **支付集成**: 真实的支付系统集成

### 后端集成
- 连接真实的商品数据库
- 集成支付网关
- 订单管理系统
- 库存管理

## 📞 技术支持

### 常见问题
1. **WebApp无法打开**: 检查HTTPS配置和Bot设置
2. **数据不保存**: 检查CloudStorage权限
3. **主题不切换**: 确认Telegram版本支持

### 调试方法
- 使用浏览器开发者工具
- 查看Telegram WebApp控制台
- 检查网络请求

## 📄 许可证

本项目基于MIT许可证开源。

## 🤝 贡献

欢迎提交Issue和Pull Request来改进这个项目。

---

**注意**: 这是一个演示项目，实际使用时需要根据具体需求进行调整和完善。 