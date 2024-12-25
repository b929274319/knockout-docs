# Knockout.js 安全指南

本文介绍在使用 Knockout.js 开发应用时需要注意的安全问题和最佳实践。

## XSS 防护

### 1. 文本绑定

```javascript
// 不安全的绑定
<div data-bind="html: userInput"></div>

// 安全的绑定
<div data-bind="text: userInput"></div>

// 如果必须使用 HTML，请进行过滤
viewModel.safeHtml = ko.computed(function() {
    return DOMPurify.sanitize(this.userInput());
}, viewModel);
```

### 2. 自定义绑定

```javascript
// 不安全的自定义绑定
ko.bindingHandlers.unsafeBinding = {
    update: function(element, valueAccessor) {
        element.innerHTML = ko.unwrap(valueAccessor());
    }
};

// 安全的自定义绑定
ko.bindingHandlers.safeBinding = {
    update: function(element, valueAccessor) {
        element.textContent = ko.unwrap(valueAccessor());
    }
};
```

### 3. 模板处理

```javascript
// 不安全的模板
<script type="text/html" id="userTemplate">
    <div data-bind="html: content"></div>
</script>

// 安全的模板
<script type="text/html" id="userTemplate">
    <div data-bind="text: content"></div>
</script>

// 使用模板引擎的安全选项
ko.templates.options = {
    escapeHtml: true
};
```

## CSRF 防护

### 1. AJAX 请求

```javascript
// 添加 CSRF 令牌
var csrfToken = document.querySelector('meta[name="csrf-token"]').content;

$.ajaxSetup({
    headers: {
        'X-CSRF-TOKEN': csrfToken
    }
});

// 使用示例
function ViewModel() {
    this.saveData = function() {
        $.post('/api/save', {
            data: this.data(),
            _token: csrfToken
        });
    };
}
```

### 2. 表单提交

```html
<!-- 添加 CSRF 字段 -->
<form data-bind="submit: submitForm">
    <input type="hidden" name="_token" data-bind="value: csrfToken">
    <!-- 其他表单字段 -->
</form>

<script>
function ViewModel() {
    this.csrfToken = ko.observable(document.querySelector('meta[name="csrf-token"]').content);
    
    this.submitForm = function() {
        // 表单提交逻辑
    };
}
</script>
```

## 数据验证

### 1. 输入验证

```javascript
// 使用 knockout-validation
var viewModel = {
    username: ko.observable().extend({
        required: true,
        pattern: {
            message: '用户名只能包含字母和数字',
            params: '^[a-zA-Z0-9]+$'
        }
    }),
    
    email: ko.observable().extend({
        required: true,
        email: true
    })
};

// 自定义验证规则
ko.validation.rules['safeString'] = {
    validator: function(val) {
        return /^[\w\s-]+$/.test(val);
    },
    message: '包含不允许的字符'
};
```

### 2. 数据清理

```javascript
// 清理用户输入
function sanitizeInput(input) {
    // 移除 HTML 标签
    input = input.replace(/<[^>]*>/g, '');
    
    // 转义特殊字符
    input = input.replace(/[&<>"']/g, function(m) {
        const map = {
            '&': '&amp;',
            '<': '&lt;',
            '>': '&gt;',
            '"': '&quot;',
            "'": '&#39;'
        };
        return map[m];
    });
    
    return input;
}

// 在视图模型中使用
function ViewModel() {
    this.userInput = ko.observable('');
    
    this.sanitizedInput = ko.computed(function() {
        return sanitizeInput(this.userInput());
    }, this);
}
```

## 安全存储

### 1. 本地存储

```javascript
// 敏感数据加密
var storage = {
    set: function(key, value) {
        const encrypted = CryptoJS.AES.encrypt(
            JSON.stringify(value),
            'secret_key'
        ).toString();
        localStorage.setItem(key, encrypted);
    },
    
    get: function(key) {
        const encrypted = localStorage.getItem(key);
        if (!encrypted) return null;
        
        const decrypted = CryptoJS.AES.decrypt(
            encrypted,
            'secret_key'
        ).toString(CryptoJS.enc.Utf8);
        
        return JSON.parse(decrypted);
    }
};

// 使用示例
viewModel.saveData = function() {
    storage.set('userData', {
        name: this.name(),
        preferences: this.preferences()
    });
};
```

### 2. 会话管理

```javascript
// 安全的会话处理
var session = {
    start: function() {
        this.token = generateSecureToken();
        storage.set('sessionToken', this.token);
    },
    
    validate: function() {
        const storedToken = storage.get('sessionToken');
        return storedToken === this.token;
    },
    
    end: function() {
        this.token = null;
        storage.set('sessionToken', null);
    }
};
```

## API 安全

### 1. 请求验证

```javascript
// API 请求包装器
var api = {
    request: function(url, options = {}) {
        // 添加认证头
        options.headers = {
            ...options.headers,
            'Authorization': `Bearer ${this.getToken()}`,
            'X-CSRF-TOKEN': this.getCsrfToken()
        };
        
        // 验证请求参数
        if (options.data) {
            options.data = this.validateRequestData(options.data);
        }
        
        return fetch(url, options)
            .then(this.handleResponse)
            .catch(this.handleError);
    },
    
    validateRequestData: function(data) {
        // 实现数据验证逻辑
        return data;
    }
};
```

### 2. 响应处理

```javascript
// 安全的响应处理
var responseHandler = {
    handle: function(response) {
        if (!response.ok) {
            throw new Error('Network response was not ok');
        }
        
        return response.json().then(data => {
            // 验证响应数据
            if (!this.validateResponseData(data)) {
                throw new Error('Invalid response data');
            }
            
            return data;
        });
    },
    
    validateResponseData: function(data) {
        // 实现响应数据验证
        return true;
    }
};
```

## 权限控制

### 1. 视图权限

```javascript
// 权限检查绑定
ko.bindingHandlers.requirePermission = {
    init: function(element, valueAccessor) {
        const permission = ko.unwrap(valueAccessor());
        if (!hasPermission(permission)) {
            element.style.display = 'none';
        }
    }
};

// 使用示例
<div data-bind="requirePermission: 'admin'">
    管理员内容
</div>
```

### 2. 功能权限

```javascript
// 权限检查装饰器
function checkPermission(permission) {
    return function(target, key, descriptor) {
        const original = descriptor.value;
        
        descriptor.value = function(...args) {
            if (!hasPermission(permission)) {
                throw new Error('Permission denied');
            }
            return original.apply(this, args);
        };
        
        return descriptor;
    };
}

// 使用示例
class AdminViewModel {
    @checkPermission('admin')
    deleteUser(userId) {
        // 删除用户逻辑
    }
}
```

## 安全配置

### 1. CSP 配置

```html
<!-- 在 HTML 中添加 CSP -->
<meta http-equiv="Content-Security-Policy" content="
    default-src 'self';
    script-src 'self' 'unsafe-inline' 'unsafe-eval';
    style-src 'self' 'unsafe-inline';
    img-src 'self' data:;
">
```

### 2. 框架配置

```javascript
// Knockout.js 安全配置
ko.options = {
    deferUpdates: true,
    useOnlyNativeEvents: true
};

// 禁用不安全的特性
ko.options.allowPrototypeForTrackArrayChanges = false;
```

## 最佳实践

### 1. 代码安全

- 使用严格模式
- 避免 eval 和 new Function
- 验证所有用户输入

### 2. 数据安全

- 加密敏感数据
- 安全存储凭证
- 定期清理数据

### 3. 通信安全

- 使用 HTTPS
- 实施 CSRF 保护
- 验证 API 调用

## 安全检查清单

1. **输入验证**
   - [ ] 验证所有用户输入
   - [ ] 实施 XSS 防护
   - [ ] 使用安全的绑定

2. **认证授权**
   - [ ] 实施会话管理
   - [ ] 控制访问权限
   - [ ] 保护敏感操作

3. **数据保护**
   - [ ] 加密敏感数据
   - [ ] 安全存储凭证
   - [ ] 实施 CSRF 保护

## 总结

安全要点：

1. 防止 XSS 攻击
2. 实施 CSRF 保护
3. 验证用户输入
4. 保护敏感数据
5. 控制访问权限 