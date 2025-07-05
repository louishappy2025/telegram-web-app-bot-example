# 用户信息缓存功能

## 概述
为了提升用户体验，避免用户每次下单都需要重新输入收货地址和手机号，系统引入了用户信息缓存功能。

## 功能特点

### 1. 自动缓存
- **实时保存**：用户输入收货地址和手机号时，系统自动保存到本地存储
- **防抖处理**：使用500ms防抖机制，避免频繁保存操作
- **失焦保存**：用户离开输入框时也会触发保存

### 2. 自动恢复
- **页面加载**：进入下单页面时自动恢复上次输入的信息
- **智能填充**：只有存在缓存数据时才会填充表单

### 3. 数据更新
- **订单成功**：订单提交成功后更新缓存，确保使用最新的用户信息
- **版本控制**：为将来可能的数据结构变更预留扩展空间

## 技术实现

### 存储键定义
```javascript
const USER_ADDRESS_KEY = 'supermarket_user_address';
const USER_PHONE_KEY = 'supermarket_user_phone';
```

### 核心函数

#### 1. 保存用户地址
```javascript
function saveUserAddress(address) {
    try {
        if (address && address.trim()) {
            localStorage.setItem(USER_ADDRESS_KEY, address.trim());
            console.log('💾 [saveUserAddress] 用户地址已保存到本地存储:', address.trim());
        }
    } catch (error) {
        console.error('❌ [saveUserAddress] 保存用户地址失败:', error);
    }
}
```

#### 2. 加载用户地址
```javascript
function loadUserAddress() {
    try {
        const savedAddress = localStorage.getItem(USER_ADDRESS_KEY);
        if (savedAddress) {
            console.log('📍 [loadUserAddress] 从本地存储加载用户地址:', savedAddress);
            return savedAddress;
        }
    } catch (error) {
        console.error('❌ [loadUserAddress] 加载用户地址失败:', error);
    }
    return '';
}
```

#### 3. 保存用户手机号
```javascript
function saveUserPhone(phone) {
    try {
        if (phone && phone.trim()) {
            localStorage.setItem(USER_PHONE_KEY, phone.trim());
            console.log('💾 [saveUserPhone] 用户手机号已保存到本地存储:', phone.trim());
        }
    } catch (error) {
        console.error('❌ [saveUserPhone] 保存用户手机号失败:', error);
    }
}
```

#### 4. 加载用户手机号
```javascript
function loadUserPhone() {
    try {
        const savedPhone = localStorage.getItem(USER_PHONE_KEY);
        if (savedPhone) {
            console.log('📱 [loadUserPhone] 从本地存储加载用户手机号:', savedPhone);
            return savedPhone;
        }
    } catch (error) {
        console.error('❌ [loadUserPhone] 加载用户手机号失败:', error);
    }
    return '';
}
```

#### 5. 加载用户信息到表单
```javascript
function loadUserInfoFromCache() {
    try {
        // 加载收货地址
        const savedAddress = loadUserAddress();
        if (savedAddress) {
            const addressInput = document.getElementById('user-address');
            if (addressInput) {
                addressInput.value = savedAddress;
                console.log('📍 [loadUserInfoFromCache] 已恢复收货地址:', savedAddress);
            }
        }

        // 加载手机号
        const savedPhone = loadUserPhone();
        if (savedPhone) {
            const phoneInput = document.getElementById('user-phone');
            if (phoneInput) {
                phoneInput.value = savedPhone;
                console.log('📱 [loadUserInfoFromCache] 已恢复手机号:', savedPhone);
            }
        }
    } catch (error) {
        console.error('❌ [loadUserInfoFromCache] 加载用户信息失败:', error);
    }
}
```

#### 6. 初始化缓存功能
```javascript
function initUserInfoCaching() {
    try {
        // 为收货地址输入框绑定事件
        const addressInput = document.getElementById('user-address');
        if (addressInput) {
            // 使用防抖处理，避免频繁保存
            let addressSaveTimer;
            addressInput.addEventListener('input', function() {
                clearTimeout(addressSaveTimer);
                addressSaveTimer = setTimeout(() => {
                    const address = this.value.trim();
                    if (address) {
                        saveUserAddress(address);
                    }
                }, 500); // 500ms 防抖
            });
            
            // 失去焦点时也保存
            addressInput.addEventListener('blur', function() {
                const address = this.value.trim();
                if (address) {
                    saveUserAddress(address);
                }
            });
        }

        // 为手机号输入框绑定事件
        const phoneInput = document.getElementById('user-phone');
        if (phoneInput) {
            // 使用防抖处理，避免频繁保存
            let phoneSaveTimer;
            phoneInput.addEventListener('input', function() {
                clearTimeout(phoneSaveTimer);
                phoneSaveTimer = setTimeout(() => {
                    const phone = this.value.trim();
                    if (phone) {
                        saveUserPhone(phone);
                    }
                }, 500); // 500ms 防抖
            });
            
            // 失去焦点时也保存
            phoneInput.addEventListener('blur', function() {
                const phone = this.value.trim();
                if (phone) {
                    saveUserPhone(phone);
                }
            });
        }
    } catch (error) {
        console.error('❌ [initUserInfoCaching] 初始化用户信息缓存失败:', error);
    }
}
```

## 触发时机

### 1. 页面初始化
- 在`initPage()`函数中调用`initUserInfoCaching()`
- 绑定输入框的事件监听器

### 2. 进入下单页面
- 在`goToCheckout()`函数中调用`loadUserInfoFromCache()`
- 自动恢复上次输入的用户信息

### 3. 用户输入时
- 监听`input`事件，使用防抖机制自动保存
- 监听`blur`事件，失去焦点时立即保存

### 4. 订单提交成功
- 在订单提交成功后更新缓存
- 确保使用最新的用户信息

## 用户体验改进

### 1. 便利性
- 用户无需每次下单都重新输入收货地址和手机号
- 自动填充上次使用的信息，提升下单效率

### 2. 智能化
- 只有在用户实际输入内容时才会保存
- 防抖机制避免频繁的存储操作

### 3. 可靠性
- 完善的错误处理机制
- 详细的日志记录便于调试

## 数据安全

### 1. 本地存储
- 数据仅存储在用户设备的本地存储中
- 不会上传到服务器或第三方

### 2. 数据清理
- 用户可以通过清除浏览器数据来删除缓存
- 系统不会永久保存敏感信息

### 3. 隐私保护
- 仅缓存必要的配送信息
- 不存储支付相关的敏感数据

## 兼容性

### 1. 浏览器支持
- 支持所有现代浏览器的localStorage API
- 包含完善的错误处理机制

### 2. 设备支持
- 同时支持PC端和移动端
- 适配Telegram Mini App环境

### 3. 向后兼容
- 不影响现有功能
- 可以独立启用或禁用

## 调试支持

### 1. 控制台日志
- 详细记录保存和加载操作
- 包含错误信息和调试信息

### 2. 手动测试
- 可以在控制台手动调用相关函数
- 方便开发者调试和验证功能

## 总结

用户信息缓存功能显著提升了用户的下单体验，减少了重复输入的工作量。通过智能的缓存机制和完善的错误处理，确保了功能的可靠性和用户数据的安全性。

该功能采用了现代Web开发的最佳实践，包括防抖处理、错误处理、日志记录等，为用户提供了流畅的购物体验。 