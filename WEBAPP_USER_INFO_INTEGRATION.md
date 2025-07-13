# WebApp 用户信息获取解决方案

## 问题描述

用户通过 `/start` 命令创建的底部 WebApp 按钮（ReplyKeyboardMarkup）无法获取到用户信息，因为 `window.Telegram.WebApp.initDataUnsafe.user` 为空。

## 核心问题

**ReplyKeyboardMarkup 与 InlineKeyboardMarkup 的根本区别**：
- **ReplyKeyboardMarkup**（底部按钮）：无法传递用户信息到 `initDataUnsafe`
- **InlineKeyboardMarkup**（内联按钮）：可以传递完整用户信息到 `initDataUnsafe`

## 解决方案对比

### 方案1：URL参数传递（推荐）✅

**优点**：
- 立即可用，无需等待SDK更新
- 用户体验好，WebApp保持打开状态
- 实现简单，维护容易

**缺点**：
- URL可见，安全性相对较低
- 参数可被篡改（需要后端验证）

### 方案2：切换到内联按钮 ✅

**优点**：
- 原生支持用户信息传递
- 安全性高，数据由Telegram保证
- 无需额外处理

**缺点**：
- 改变了用户界面布局
- 内联按钮不在底部固定位置

### 方案3：混合方案 ✅

**优点**：
- 保持底部按钮的用户体验
- 关键操作使用内联按钮确保安全
- 灵活性高

## 推荐实现：URL参数方案

### 1. 后端实现

```java
/**
 * 创建包含用户信息的WebApp按钮
 */
private ReplyKeyboardMarkup createMiniAppReplyKeyboard(org.telegram.telegrambots.meta.api.objects.User user) {
    String baseUrl = "https://dhcs.cc/supermarket-tg/index.html";
    
    try {
        // 构建用户信息参数
        String userParams = String.format("?user_id=%d&username=%s&first_name=%s&last_name=%s",
                user.getId(),
                urlEncode(user.getUserName()),
                urlEncode(user.getFirstName()),
                urlEncode(user.getLastName()));
        
        // 创建不同页面的URL
        String orderUrl = baseUrl + userParams + "&page=order";
        String myOrdersUrl = baseUrl + userParams + "&page=my_orders";
        
        // 创建键盘
        List<KeyboardRow> keyboard = new ArrayList<>();
        
        // 订单按钮
        KeyboardRow firstRow = new KeyboardRow();
        KeyboardButton orderButton = KeyboardButton.builder()
                .text("订单 🛒")
                .webApp(WebAppInfo.builder().url(orderUrl).build())
                .build();
        firstRow.add(orderButton);
        keyboard.add(firstRow);
        
        // 我的订单按钮
        KeyboardRow secondRow = new KeyboardRow();
        KeyboardButton myOrdersButton = KeyboardButton.builder()
                .text("我的订单 📋")
                .webApp(WebAppInfo.builder().url(myOrdersUrl).build())
                .build();
        secondRow.add(myOrdersButton);
        keyboard.add(secondRow);
        
        // 刷新按钮
        KeyboardRow thirdRow = new KeyboardRow();
        KeyboardButton startButton = KeyboardButton.builder()
                .text("/start")
                .build();
        thirdRow.add(startButton);
        keyboard.add(thirdRow);
        
        return ReplyKeyboardMarkup.builder()
                .keyboard(keyboard)
                .resizeKeyboard(true)
                .oneTimeKeyboard(false)
                .build();
                
    } catch (Exception e) {
        log.error("创建WebApp按钮时URL编码失败", e);
        return createFallbackKeyboard();
    }
}

/**
 * URL编码辅助方法
 */
private String urlEncode(String value) {
    if (value == null || value.trim().isEmpty()) {
        return "";
    }
    try {
        return java.net.URLEncoder.encode(value, "UTF-8");
    } catch (Exception e) {
        log.warn("URL编码失败: {}", value, e);
        return "";
    }
}

/**
 * 降级方案：创建普通文本按钮
 */
private ReplyKeyboardMarkup createFallbackKeyboard() {
    List<KeyboardRow> keyboard = new ArrayList<>();
    
    KeyboardRow row = new KeyboardRow();
    row.add(KeyboardButton.builder().text("📱 请联系客服获取小程序链接").build());
    keyboard.add(row);
    
    return ReplyKeyboardMarkup.builder()
            .keyboard(keyboard)
            .resizeKeyboard(true)
            .oneTimeKeyboard(false)
            .build();
}
```

### 2. 前端实现

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>超市订购系统</title>
    <script src="https://telegram.org/js/telegram-web-app.js"></script>
    <style>
        body {
            margin: 0;
            padding: 0;
            font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
            background-color: var(--tg-theme-bg-color, #ffffff);
            color: var(--tg-theme-text-color, #000000);
        }
        .container {
            max-width: 400px;
            margin: 0 auto;
            padding: 20px;
        }
        .user-info {
            background: var(--tg-theme-secondary-bg-color, #f0f0f0);
            padding: 15px;
            border-radius: 10px;
            margin-bottom: 20px;
        }
        .button {
            background: var(--tg-theme-button-color, #007bff);
            color: var(--tg-theme-button-text-color, #ffffff);
            border: none;
            padding: 15px 30px;
            border-radius: 10px;
            font-size: 16px;
            cursor: pointer;
            width: 100%;
            margin: 10px 0;
        }
        .error {
            color: #e74c3c;
            text-align: center;
            padding: 20px;
        }
    </style>
</head>
<body>
    <div class="container">
        <div id="app">正在加载...</div>
    </div>
    
    <script>
        /**
         * 从URL参数获取用户信息
         */
        function getUserInfoFromUrl() {
            const urlParams = new URLSearchParams(window.location.search);
            
            const userInfo = {
                id: urlParams.get('user_id'),
                username: urlParams.get('username'),
                first_name: decodeURIComponent(urlParams.get('first_name') || ''),
                last_name: decodeURIComponent(urlParams.get('last_name') || ''),
                page: urlParams.get('page') || 'order'
            };
            
            // 验证必要信息
            if (!userInfo.id) {
                return null;
            }
            
            return userInfo;
        }
        
        /**
         * 从 Telegram WebApp API 获取用户信息
         */
        function getUserInfoFromTelegram() {
            try {
                if (window.Telegram && window.Telegram.WebApp && 
                    window.Telegram.WebApp.initDataUnsafe && 
                    window.Telegram.WebApp.initDataUnsafe.user) {
                    return window.Telegram.WebApp.initDataUnsafe.user;
                }
            } catch (e) {
                console.log('无法从 Telegram API 获取用户信息:', e);
            }
            return null;
        }
        
        /**
         * 初始化应用
         */
        function initializeApp() {
            console.log('初始化应用...');
            
            // 尝试多种方式获取用户信息
            let userInfo = getUserInfoFromTelegram(); // 首先尝试原生API
            
            if (!userInfo) {
                userInfo = getUserInfoFromUrl(); // 回退到URL参数
            }
            
            if (!userInfo) {
                showError('无法获取用户信息，请重新打开小程序');
                return;
            }
            
            // 显示用户信息
            displayUserInfo(userInfo);
            
            // 初始化页面功能
            initializePageFeatures(userInfo);
        }
        
        /**
         * 显示用户信息
         */
        function displayUserInfo(userInfo) {
            const page = userInfo.page || 'order';
            const pageTitle = page === 'order' ? '📱 订单页面' : '📋 我的订单';
            
            const html = `
                <h2>${pageTitle}</h2>
                <div class="user-info">
                    <h3>👤 用户信息</h3>
                    <p><strong>姓名:</strong> ${userInfo.first_name} ${userInfo.last_name || ''}</p>
                    <p><strong>用户ID:</strong> ${userInfo.id}</p>
                    ${userInfo.username ? `<p><strong>用户名:</strong> @${userInfo.username}</p>` : ''}
                </div>
                <button class="button" onclick="startShopping()">
                    ${page === 'order' ? '开始购物 🛒' : '查看我的订单 📋'}
                </button>
                <button class="button" onclick="testUserInfo()">测试用户信息</button>
            `;
            
            document.getElementById('app').innerHTML = html;
        }
        
        /**
         * 显示错误信息
         */
        function showError(message) {
            document.getElementById('app').innerHTML = `
                <div class="error">
                    <h3>❌ 错误</h3>
                    <p>${message}</p>
                    <button class="button" onclick="location.reload()">重新加载</button>
                </div>
            `;
        }
        
        /**
         * 初始化页面功能
         */
        function initializePageFeatures(userInfo) {
            // 根据页面类型初始化不同功能
            const page = userInfo.page || 'order';
            
            if (page === 'order') {
                console.log('初始化订单页面功能');
                // 初始化订单相关功能
            } else if (page === 'my_orders') {
                console.log('初始化我的订单页面功能');
                // 初始化我的订单相关功能
            }
        }
        
        /**
         * 开始购物
         */
        function startShopping() {
            alert('购物功能开发中...');
        }
        
        /**
         * 测试用户信息
         */
        function testUserInfo() {
            const urlInfo = getUserInfoFromUrl();
            const telegramInfo = getUserInfoFromTelegram();
            
            const message = `
URL参数信息: ${JSON.stringify(urlInfo, null, 2)}

Telegram API信息: ${JSON.stringify(telegramInfo, null, 2)}
            `;
            
            alert(message);
        }
        
        // 页面加载时初始化
        window.addEventListener('load', function() {
            // 确保 Telegram WebApp 完全加载
            if (window.Telegram && window.Telegram.WebApp) {
                window.Telegram.WebApp.ready();
            }
            
            // 延迟初始化
            setTimeout(initializeApp, 300);
        });
    </script>
</body>
</html>
```

### 3. 安全考虑

```javascript
/**
 * 验证用户信息的完整性（可选）
 */
function validateUserInfo(userInfo) {
    // 基本验证
    if (!userInfo.id || !userInfo.first_name) {
        return false;
    }
    
    // 可以添加更多验证逻辑
    // 例如：检查用户ID格式、名字长度等
    
    return true;
}

/**
 * 向后端发送用户信息进行验证
 */
async function verifyUserWithBackend(userInfo) {
    try {
        const response = await fetch('/api/verify-user', {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json',
            },
            body: JSON.stringify(userInfo)
        });
        
        return response.ok;
    } catch (error) {
        console.error('用户验证失败:', error);
        return false;
    }
}
```

## URL参数格式

### 完整URL示例

```
https://dhcs.cc/supermarket-tg/index.html?user_id=1939818627&username=lancefengye&first_name=Lance&last_name=Feng&page=order
```

### 参数说明

| 参数 | 类型 | 必需 | 描述 |
|------|------|------|------|
| user_id | Integer | ✅ | Telegram用户ID |
| username | String | ❌ | 用户名（可能为空） |
| first_name | String | ✅ | 名字 |
| last_name | String | ❌ | 姓氏（可能为空） |
| page | String | ❌ | 页面类型：order/my_orders |

## 实施步骤

1. **修改机器人代码**：更新 `createMiniAppReplyKeyboard` 方法
2. **更新前端代码**：添加URL参数解析功能
3. **测试验证**：确保用户信息正确传递
4. **安全加固**：添加用户信息验证机制

## 测试方法

```bash
# 测试URL生成
curl -X POST "http://localhost:8080/test-webapp-url" \
  -H "Content-Type: application/json" \
  -d '{"user_id": 1939818627, "username": "test", "first_name": "Test", "last_name": "User"}'
```

这个方案是目前最实用和可靠的解决方案，可以立即实施并解决用户信息获取问题。 