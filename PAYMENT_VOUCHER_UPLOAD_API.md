# 支付凭证上传API文档

## 概述

支付凭证上传功能允许用户在完成支付后，上传支付凭证图片并更新订单的支付状态。该功能采用两步式设计：
1. 用户端上传图片文件获取URL
2. 提交支付凭证URL到订单系统

## 设计优势

- **分离关注点**：文件上传和业务逻辑分离，便于维护
- **灵活性**：支持任意来源的图片URL，不仅限于当前系统上传
- **性能优化**：避免重复上传，支持URL复用
- **错误处理**：清晰的错误提示和状态验证

## API接口详情

### 1. 用户端文件上传接口

#### 1.1 获取上传配置

```http
GET /api/v1/file/config
```

**响应示例：**
```json
{
  "code": 200,
  "message": "获取配置成功",
  "data": {
    "maxFileSize": 10485760,
    "allowedTypes": ["jpg", "jpeg", "png", "gif"],
    "uploadPath": "/uploads/"
  }
}
```

#### 1.2 图片上传接口

```http
POST /api/v1/file/upload/image
```

**请求参数：**
- `file` (file, required): 图片文件
- `folder` (string, optional): 文件夹名称，建议使用 "payment"

**请求示例：**
```bash
curl -X POST http://localhost:9595/api/v1/file/upload/image \
  -F "file=@payment_voucher.jpg" \
  -F "folder=payment"
```

**响应示例：**
```json
{
  "code": 200,
  "message": "上传成功",
  "data": [
    "https://your-s3-bucket.s3.amazonaws.com/payment/20250101/payment_voucher_123456.jpg"
  ]
}
```

#### 1.3 通用文件上传接口

```http
POST /api/v1/file/upload
```

**请求参数：**
- `file` (file, required): 文件
- `type` (string, required): 文件类型，使用 "image"
- `folder` (string, optional): 文件夹名称

**请求示例：**
```bash
curl -X POST http://localhost:9595/api/v1/file/upload \
  -F "file=@payment_voucher.jpg" \
  -F "type=image" \
  -F "folder=payment"
```

### 2. 支付凭证提交接口

#### 2.1 提交支付凭证

```http
POST /order/{id}/payment-voucher
```

**路径参数：**
- `id` (long, required): 订单ID

**请求体参数：**
```json
{
  "paymentUrl": "string, required - 支付凭证图片URL",
  "remark": "string, optional - 支付备注"
}
```

**请求示例：**
```bash
curl -X POST http://localhost:9595/order/123/payment-voucher \
  -H "Content-Type: application/json" \
  -d '{
    "paymentUrl": "https://your-s3-bucket.s3.amazonaws.com/payment/20250101/payment_voucher_123456.jpg",
    "remark": "ABA银行转账，交易号：ABA123456789，已完成支付"
  }'
```

**成功响应：**
```json
{
  "code": 200,
  "message": "支付凭证提交成功，订单已标记为已支付",
  "data": {
    "id": 123,
    "orderNo": "ORD20250101123456",
    "payStatus": 1,
    "paymentUrl": "https://your-s3-bucket.s3.amazonaws.com/payment/20250101/payment_voucher_123456.jpg",
    "remark": "ABA银行转账，交易号：ABA123456789，已完成支付",
    "updateTime": "2025-01-01T12:00:00"
  }
}
```

**错误响应：**

订单不存在：
```json
{
  "code": 404,
  "message": "订单不存在"
}
```

订单已支付：
```json
{
  "code": 400,
  "message": "订单已支付，无需重复提交支付凭证"
}
```

支付凭证URL为空：
```json
{
  "code": 400,
  "message": "支付凭证URL不能为空"
}
```

## 完整使用流程

### 前端JavaScript示例

```javascript
// 第一步：上传图片获取URL
async function uploadPaymentVoucher(file) {
    const formData = new FormData();
    formData.append('file', file);
    formData.append('folder', 'payment');
    
    try {
        const response = await fetch('/api/v1/file/upload/image', {
            method: 'POST',
            body: formData
        });
        
        const result = await response.json();
        
        if (result.code === 200 && result.data && result.data.length > 0) {
            return result.data[0]; // 返回图片URL
        } else {
            throw new Error(result.message || '上传失败');
        }
    } catch (error) {
        console.error('上传图片失败:', error);
        throw error;
    }
}

// 第二步：提交支付凭证到订单
async function submitPaymentVoucher(orderId, paymentUrl, remark = '') {
    try {
        const response = await fetch(`/order/${orderId}/payment-voucher`, {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json'
            },
            body: JSON.stringify({
                paymentUrl: paymentUrl,
                remark: remark
            })
        });
        
        const result = await response.json();
        
        if (result.code === 200) {
            console.log('支付凭证提交成功:', result.data);
            return result.data;
        } else {
            throw new Error(result.message || '提交失败');
        }
    } catch (error) {
        console.error('提交支付凭证失败:', error);
        throw error;
    }
}

// 完整流程示例
async function handlePaymentVoucherSubmission(orderId, file, remark) {
    try {
        // 第一步：上传图片
        console.log('正在上传图片...');
        const imageUrl = await uploadPaymentVoucher(file);
        console.log('图片上传成功，URL:', imageUrl);
        
        // 第二步：提交支付凭证
        console.log('正在提交支付凭证...');
        const updatedOrder = await submitPaymentVoucher(orderId, imageUrl, remark);
        console.log('支付凭证提交成功，订单已更新:', updatedOrder);
        
        return updatedOrder;
    } catch (error) {
        console.error('支付凭证提交流程失败:', error);
        throw error;
    }
}

// 使用示例
const fileInput = document.getElementById('payment-voucher-file');
const orderId = 123;
const remark = 'ABA银行转账，已完成支付';

fileInput.addEventListener('change', async (event) => {
    const file = event.target.files[0];
    if (file) {
        try {
            await handlePaymentVoucherSubmission(orderId, file, remark);
            alert('支付凭证提交成功！');
        } catch (error) {
            alert('支付凭证提交失败：' + error.message);
        }
    }
});
```

### React组件示例

```jsx
import React, { useState } from 'react';

const PaymentVoucherUpload = ({ orderId, onSuccess, onError }) => {
    const [uploading, setUploading] = useState(false);
    const [remark, setRemark] = useState('');

    const handleFileUpload = async (event) => {
        const file = event.target.files[0];
        if (!file) return;

        setUploading(true);
        
        try {
            // 第一步：上传图片
            const formData = new FormData();
            formData.append('file', file);
            formData.append('folder', 'payment');
            
            const uploadResponse = await fetch('/api/v1/file/upload/image', {
                method: 'POST',
                body: formData
            });
            
            const uploadResult = await uploadResponse.json();
            
            if (uploadResult.code !== 200 || !uploadResult.data || uploadResult.data.length === 0) {
                throw new Error(uploadResult.message || '图片上传失败');
            }
            
            const imageUrl = uploadResult.data[0];
            
            // 第二步：提交支付凭证
            const voucherResponse = await fetch(`/order/${orderId}/payment-voucher`, {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json'
                },
                body: JSON.stringify({
                    paymentUrl: imageUrl,
                    remark: remark
                })
            });
            
            const voucherResult = await voucherResponse.json();
            
            if (voucherResult.code === 200) {
                onSuccess && onSuccess(voucherResult.data);
            } else {
                throw new Error(voucherResult.message || '支付凭证提交失败');
            }
        } catch (error) {
            onError && onError(error);
        } finally {
            setUploading(false);
        }
    };

    return (
        <div className="payment-voucher-upload">
            <div className="form-group">
                <label htmlFor="payment-voucher">上传支付凭证：</label>
                <input
                    type="file"
                    id="payment-voucher"
                    accept="image/*"
                    onChange={handleFileUpload}
                    disabled={uploading}
                />
            </div>
            
            <div className="form-group">
                <label htmlFor="payment-remark">支付备注：</label>
                <textarea
                    id="payment-remark"
                    value={remark}
                    onChange={(e) => setRemark(e.target.value)}
                    placeholder="请输入支付相关信息，如转账单号、支付方式等"
                    disabled={uploading}
                />
            </div>
            
            {uploading && <div className="loading">正在处理...</div>}
        </div>
    );
};

export default PaymentVoucherUpload;
```

## 业务逻辑说明

### 支付状态更新

提交支付凭证后，系统会自动执行以下操作：

1. **验证订单状态**：只有未支付的订单才能提交支付凭证
2. **保存凭证URL**：将图片URL保存到订单的`paymentUrl`字段
3. **更新支付状态**：将`payStatus`字段设置为1（已支付）
4. **设置支付时间**：自动设置`payTime`为当前时间
5. **更新备注**：如果提供了备注，会更新订单的`remark`字段

### 字段映射

| 字段名 | 类型 | 说明 |
|--------|------|------|
| paymentUrl | String | 支付凭证图片URL |
| payStatus | Integer | 支付状态：0-未支付，1-已支付 |
| payTime | LocalDateTime | 支付时间（自动设置） |
| remark | String | 订单备注（可选更新） |

### 错误处理

系统会验证以下情况并返回相应错误：

- 订单不存在 → 404错误
- 订单已支付 → 400错误，防止重复提交
- 支付凭证URL为空 → 400错误
- 系统异常 → 500错误

## 测试说明

可以使用提供的测试脚本进行功能验证：

```bash
chmod +x test-payment-voucher.sh
./test-payment-voucher.sh
```

测试脚本会验证：
- 用户端文件上传接口
- 支付凭证提交接口
- 各种错误情况处理
- 订单状态更新正确性

## 注意事项

1. **图片格式**：建议支持 jpg、jpeg、png、gif 格式
2. **文件大小**：默认限制10MB，可在配置中调整
3. **URL有效性**：系统不会验证URL的有效性，请确保上传的图片URL可访问
4. **重复提交**：已支付的订单不允许重复提交支付凭证
5. **备注长度**：建议限制备注字段长度，避免过长内容
6. **权限控制**：根据业务需要添加用户权限验证

## 扩展功能建议

1. **图片预览**：前端可以在提交前预览上传的图片
2. **多图片支持**：支持一次上传多张支付凭证图片
3. **图片压缩**：前端可以在上传前压缩图片以节省带宽
4. **进度显示**：显示上传进度和处理状态
5. **历史记录**：记录支付凭证的提交历史 