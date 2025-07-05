# 付款方式前端API文档

## 概述

本文档专门为前端开发者提供付款方式相关的API接口说明。所有接口都使用简化的数据结构，无需复杂的JSON解析。

⚠️ **重要说明**：所有布尔字段使用Integer类型（0/1），请勿使用true/false。

## 接口列表

### 1. 获取所有启用的付款方式

**接口地址**: `GET /payment-methods`

**请求方式**: `GET`

**请求参数**: 无

**响应示例**:
```json
{
  "code": 200,
  "message": "获取成功",
  "data": [
    {
      "id": 1,
      "code": "aba_bank",
      "name": "ABA银行转账",
      "description": "ABA银行转账付款",
      "iconUrl": "/images/payment/aba_icon.png",
      "imageUrl": "/images/payment/aba_payment.jpg",
      "enabled": 1,
      "sortOrder": 1,
      "requireAccountName": 1,
      "accountNameLabel": "收款人姓名",
      "accountNamePlaceholder": "请输入收款人姓名",
      "requireAccountNumber": 1,
      "accountNumberLabel": "银行账号",
      "accountNumberPlaceholder": "请输入ABA银行账号",
      "requireWalletAddress": 0,
      "walletAddressLabel": null,
      "walletAddressPlaceholder": null,
      "requireCashAmount": 0,
      "cashAmountLabel": null,
      "cashAmountPlaceholder": null,
      "requireChangeAmount": 0,
      "changeAmountLabel": null,
      "changeAmountPlaceholder": null,
      "paymentInstructions": "请转账至ABA银行账户：{accountName}（账号：{accountNumber}）",
      "createTime": "2025-01-01T10:00:00",
      "updateTime": "2025-01-01T10:00:00"
    },
    {
      "id": 2,
      "code": "wing_bank",
      "name": "汇旺银行转账",
      "description": "汇旺银行转账付款",
      "iconUrl": "/images/payment/wing_icon.png",
      "imageUrl": "/images/payment/wing_payment.jpg",
      "enabled": 1,
      "sortOrder": 2,
      "requireAccountName": 1,
      "accountNameLabel": "收款人姓名",
      "accountNamePlaceholder": "请输入收款人姓名",
      "requireAccountNumber": 1,
      "accountNumberLabel": "银行账号",
      "accountNumberPlaceholder": "请输入汇旺银行账号",
      "requireWalletAddress": 0,
      "walletAddressLabel": null,
      "walletAddressPlaceholder": null,
      "requireCashAmount": 0,
      "cashAmountLabel": null,
      "cashAmountPlaceholder": null,
      "requireChangeAmount": 0,
      "changeAmountLabel": null,
      "changeAmountPlaceholder": null,
      "paymentInstructions": "请转账至汇旺银行账户：{accountName}（账号：{accountNumber}）",
      "createTime": "2025-01-01T10:00:00",
      "updateTime": "2025-01-01T10:00:00"
    },
    {
      "id": 3,
      "code": "usdt",
      "name": "USDT加密货币",
      "description": "USDT加密货币付款",
      "iconUrl": "/images/payment/usdt_icon.png",
      "imageUrl": "/images/payment/usdt_payment.jpg",
      "enabled": 1,
      "sortOrder": 3,
      "requireAccountName": 0,
      "accountNameLabel": null,
      "accountNamePlaceholder": null,
      "requireAccountNumber": 0,
      "accountNumberLabel": null,
      "accountNumberPlaceholder": null,
      "requireWalletAddress": 1,
      "walletAddressLabel": "USDT钱包地址",
      "walletAddressPlaceholder": "请输入USDT钱包地址",
      "requireCashAmount": 0,
      "cashAmountLabel": null,
      "cashAmountPlaceholder": null,
      "requireChangeAmount": 0,
      "changeAmountLabel": null,
      "changeAmountPlaceholder": null,
      "paymentInstructions": "请转账USDT至钱包地址：{walletAddress}",
      "createTime": "2025-01-01T10:00:00",
      "updateTime": "2025-01-01T10:00:00"
    },
    {
      "id": 4,
      "code": "cash_on_delivery",
      "name": "货到付款",
      "description": "送货时现金付款",
      "iconUrl": "/images/payment/cod_icon.png",
      "imageUrl": "/images/payment/cod_payment.jpg",
      "enabled": 1,
      "sortOrder": 4,
      "requireAccountName": 0,
      "accountNameLabel": null,
      "accountNamePlaceholder": null,
      "requireAccountNumber": 0,
      "accountNumberLabel": null,
      "accountNumberPlaceholder": null,
      "requireWalletAddress": 0,
      "walletAddressLabel": null,
      "walletAddressPlaceholder": null,
      "requireCashAmount": 1,
      "cashAmountLabel": "应付现金",
      "cashAmountPlaceholder": "请输入应付现金金额",
      "requireChangeAmount": 1,
      "changeAmountLabel": "找零金额",
      "changeAmountPlaceholder": "请输入需要找零的金额",
      "paymentInstructions": "货到付款：应付现金 {cashAmount}，找零 {changeAmount}",
      "createTime": "2025-01-01T10:00:00",
      "updateTime": "2025-01-01T10:00:00"
    }
  ]
}
```

### 2. 根据代码获取付款方式详情

**接口地址**: `GET /payment-methods/{code}`

**请求方式**: `GET`

**路径参数**:
- `code` (String, 必填): 付款方式代码

**请求示例**:
```bash
GET /payment-methods/aba_bank
```

**响应示例**:
```json
{
  "code": 200,
  "message": "获取成功",
  "data": {
    "id": 1,
    "code": "aba_bank",
    "name": "ABA银行转账",
    "description": "ABA银行转账付款",
    "iconUrl": "/images/payment/aba_icon.png",
    "imageUrl": "/images/payment/aba_payment.jpg",
    "enabled": 1,
    "sortOrder": 1,
    "requireAccountName": 1,
    "accountNameLabel": "收款人姓名",
    "accountNamePlaceholder": "请输入收款人姓名",
    "requireAccountNumber": 1,
    "accountNumberLabel": "银行账号",
    "accountNumberPlaceholder": "请输入ABA银行账号",
    "requireWalletAddress": 0,
    "walletAddressLabel": null,
    "walletAddressPlaceholder": null,
    "requireCashAmount": 0,
    "cashAmountLabel": null,
    "cashAmountPlaceholder": null,
    "requireChangeAmount": 0,
    "changeAmountLabel": null,
    "changeAmountPlaceholder": null,
    "paymentInstructions": "请转账至ABA银行账户：{accountName}（账号：{accountNumber}）",
    "createTime": "2025-01-01T10:00:00",
    "updateTime": "2025-01-01T10:00:00"
  }
}
```

## 响应字段说明

| 字段名 | 类型 | 说明 |
|--------|------|------|
| id | Long | 付款方式ID |
| code | String | 付款方式代码（唯一标识） |
| name | String | 付款方式名称 |
| description | String | 付款方式描述 |
| iconUrl | String | 付款方式图标URL（用于列表展示，32x32） |
| imageUrl | String | 付款方式图片URL（用于详情页面，400x300） |
| enabled | Integer | 是否启用（0-禁用，1-启用） |
| sortOrder | Integer | 排序顺序 |
| requireAccountName | Integer | 是否需要收款人姓名（0-不需要，1-需要） |
| accountNameLabel | String | 收款人姓名字段标签 |
| accountNamePlaceholder | String | 收款人姓名占位符提示 |
| requireAccountNumber | Integer | 是否需要收款账号（0-不需要，1-需要） |
| accountNumberLabel | String | 收款账号字段标签 |
| accountNumberPlaceholder | String | 收款账号占位符提示 |
| requireWalletAddress | Integer | 是否需要钱包地址（0-不需要，1-需要） |
| walletAddressLabel | String | 钱包地址字段标签 |
| walletAddressPlaceholder | String | 钱包地址占位符提示 |
| requireCashAmount | Integer | 是否需要现金金额（0-不需要，1-需要） |
| cashAmountLabel | String | 现金金额字段标签 |
| cashAmountPlaceholder | String | 现金金额占位符提示 |
| requireChangeAmount | Integer | 是否需要找零金额（0-不需要，1-需要） |
| changeAmountLabel | String | 找零金额字段标签 |
| changeAmountPlaceholder | String | 找零金额占位符提示 |
| paymentInstructions | String | 付款说明模板（支持占位符） |
| createTime | DateTime | 创建时间 |
| updateTime | DateTime | 更新时间 |

## 前端开发指南

### 1. 获取付款方式列表

```javascript
async function getPaymentMethods() {
  try {
    const response = await fetch('/payment-methods');
    const result = await response.json();
    
    if (result.code === 200) {
      return result.data;
    } else {
      console.error('获取付款方式失败:', result.message);
      return [];
    }
  } catch (error) {
    console.error('请求失败:', error);
    return [];
  }
}
```

### 2. 动态生成表单

```javascript
function generatePaymentForm(paymentMethod) {
  const formHTML = [];
  
  // 收款人姓名字段
  if (paymentMethod.requireAccountName === 1) {
    formHTML.push(`
      <div class="form-group">
        <label>${paymentMethod.accountNameLabel}</label>
        <input 
          type="text" 
          name="accountName" 
          placeholder="${paymentMethod.accountNamePlaceholder}"
          required 
        />
      </div>
    `);
  }
  
  // 收款账号字段
  if (paymentMethod.requireAccountNumber === 1) {
    formHTML.push(`
      <div class="form-group">
        <label>${paymentMethod.accountNumberLabel}</label>
        <input 
          type="text" 
          name="accountNumber" 
          placeholder="${paymentMethod.accountNumberPlaceholder}"
          required 
        />
      </div>
    `);
  }
  
  // 钱包地址字段
  if (paymentMethod.requireWalletAddress === 1) {
    formHTML.push(`
      <div class="form-group">
        <label>${paymentMethod.walletAddressLabel}</label>
        <input 
          type="text" 
          name="walletAddress" 
          placeholder="${paymentMethod.walletAddressPlaceholder}"
          required 
        />
      </div>
    `);
  }
  
  // 现金金额字段
  if (paymentMethod.requireCashAmount === 1) {
    formHTML.push(`
      <div class="form-group">
        <label>${paymentMethod.cashAmountLabel}</label>
        <input 
          type="number" 
          name="cashAmount" 
          placeholder="${paymentMethod.cashAmountPlaceholder}"
          required 
        />
      </div>
    `);
  }
  
  // 找零金额字段
  if (paymentMethod.requireChangeAmount === 1) {
    formHTML.push(`
      <div class="form-group">
        <label>${paymentMethod.changeAmountLabel}</label>
        <input 
          type="number" 
          name="changeAmount" 
          placeholder="${paymentMethod.changeAmountPlaceholder}"
          required 
        />
      </div>
    `);
  }
  
  return formHTML.join('');
}
```

### 3. 表单验证

```javascript
function validatePaymentForm(paymentMethod, formData) {
  const errors = [];
  
  if (paymentMethod.requireAccountName === 1 && !formData.accountName) {
    errors.push(`请填写${paymentMethod.accountNameLabel}`);
  }
  
  if (paymentMethod.requireAccountNumber === 1 && !formData.accountNumber) {
    errors.push(`请填写${paymentMethod.accountNumberLabel}`);
  }
  
  if (paymentMethod.requireWalletAddress === 1 && !formData.walletAddress) {
    errors.push(`请填写${paymentMethod.walletAddressLabel}`);
  }
  
  if (paymentMethod.requireCashAmount === 1 && (!formData.cashAmount || formData.cashAmount <= 0)) {
    errors.push(`请填写有效的${paymentMethod.cashAmountLabel}`);
  }
  
  if (paymentMethod.requireChangeAmount === 1 && (!formData.changeAmount || formData.changeAmount < 0)) {
    errors.push(`请填写有效的${paymentMethod.changeAmountLabel}`);
  }
  
  return errors;
}
```

### 4. Vue.js 组件示例

```vue
<template>
  <div class="payment-selector">
    <!-- 付款方式列表 -->
    <div class="payment-methods">
      <div 
        v-for="method in paymentMethods" 
        :key="method.id"
        class="payment-method"
        :class="{ active: selectedMethod?.id === method.id }"
        @click="selectMethod(method)"
      >
        <img :src="method.iconUrl" :alt="method.name" />
        <span>{{ method.name }}</span>
      </div>
    </div>
    
    <!-- 付款表单 -->
    <div v-if="selectedMethod" class="payment-form">
      <h4>{{ selectedMethod.name }}</h4>
      <p>{{ selectedMethod.description }}</p>
      
      <form @submit.prevent="submitPayment">
        <!-- 收款人姓名 -->
        <div v-if="selectedMethod.requireAccountName === 1" class="form-group">
          <label>{{ selectedMethod.accountNameLabel }}</label>
          <input 
            v-model="formData.accountName"
            type="text"
            :placeholder="selectedMethod.accountNamePlaceholder"
            required
          />
        </div>
        
        <!-- 收款账号 -->
        <div v-if="selectedMethod.requireAccountNumber === 1" class="form-group">
          <label>{{ selectedMethod.accountNumberLabel }}</label>
          <input 
            v-model="formData.accountNumber"
            type="text"
            :placeholder="selectedMethod.accountNumberPlaceholder"
            required
          />
        </div>
        
        <!-- 钱包地址 -->
        <div v-if="selectedMethod.requireWalletAddress === 1" class="form-group">
          <label>{{ selectedMethod.walletAddressLabel }}</label>
          <input 
            v-model="formData.walletAddress"
            type="text"
            :placeholder="selectedMethod.walletAddressPlaceholder"
            required
          />
        </div>
        
        <!-- 现金金额 -->
        <div v-if="selectedMethod.requireCashAmount === 1" class="form-group">
          <label>{{ selectedMethod.cashAmountLabel }}</label>
          <input 
            v-model="formData.cashAmount"
            type="number"
            :placeholder="selectedMethod.cashAmountPlaceholder"
            required
          />
        </div>
        
        <!-- 找零金额 -->
        <div v-if="selectedMethod.requireChangeAmount === 1" class="form-group">
          <label>{{ selectedMethod.changeAmountLabel }}</label>
          <input 
            v-model="formData.changeAmount"
            type="number"
            :placeholder="selectedMethod.changeAmountPlaceholder"
            required
          />
        </div>
        
        <button type="submit">确认付款</button>
      </form>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      paymentMethods: [],
      selectedMethod: null,
      formData: {}
    };
  },
  async mounted() {
    this.paymentMethods = await this.getPaymentMethods();
  },
  methods: {
    async getPaymentMethods() {
      const response = await fetch('/payment-methods');
      const result = await response.json();
      return result.code === 200 ? result.data : [];
    },
    
    selectMethod(method) {
      this.selectedMethod = method;
      this.formData = {};
    },
    
    submitPayment() {
      const paymentData = {
        paymentMethodCode: this.selectedMethod.code,
        paymentDetails: this.formData
      };
      console.log('提交付款数据:', paymentData);
    }
  }
};
</script>
```

## 错误处理

### 常见错误码

| 错误码 | 说明 | 处理方式 |
|--------|------|----------|
| 200 | 请求成功 | 正常处理数据 |
| 404 | 付款方式不存在或已禁用 | 提示用户选择其他付款方式 |
| 500 | 服务器内部错误 | 提示用户稍后重试 |

### 错误处理示例

```javascript
async function handlePaymentMethodRequest() {
  try {
    const response = await fetch('/payment-methods');
    const result = await response.json();
    
    if (result.code === 200) {
      return result.data;
    } else if (result.code === 404) {
      alert('付款方式不可用，请稍后重试');
      return [];
    } else {
      throw new Error(result.message);
    }
  } catch (error) {
    console.error('获取付款方式失败:', error);
    alert('网络错误，请检查网络连接');
    return [];
  }
}
```

## 最佳实践

1. **缓存付款方式列表**：避免频繁请求
2. **图片懒加载**：优化页面加载速度
3. **表单验证**：在提交前验证必填字段
4. **错误处理**：提供友好的错误提示
5. **响应式设计**：适配不同屏幕尺寸

## 注意事项

- 所有布尔字段使用Integer类型（0/1），不要使用true/false
- 图片URL可能为null，需要做空值判断
- 付款说明支持占位符，可以动态替换用户输入的值
- 排序按sortOrder字段升序排列 