# 文本和外观绑定

本文介绍 Knockout.js 中用于处理文本显示和元素外观的绑定。这些绑定可以帮助你控制文本内容的显示方式和元素的视觉效果。

## 文本绑定

### text 绑定

用于显示纯文本内容。

```html
<span data-bind="text: message"></span>

<script>
var viewModel = {
    message: ko.observable("你好，世界！")
};
</script>
```

### html 绑定

用于显示 HTML 内容。

```html
<div data-bind="html: htmlContent"></div>

<script>
var viewModel = {
    htmlContent: ko.observable("<strong>重要提示：</strong>这是一条消息")
};
</script>
```

### value 绑定

用于表单元素的值绑定。

```html
<input data-bind="value: userName" />
<textarea data-bind="value: description"></textarea>

<script>
var viewModel = {
    userName: ko.observable("张三"),
    description: ko.observable("这是一段描述")
};
</script>
```

## 外观绑定

### visible 绑定

控制元素的可见性。

```html
<div data-bind="visible: isVisible">
    只有 isVisible 为 true 时才显示
</div>

<script>
var viewModel = {
    isVisible: ko.observable(true)
};
</script>
```

### hidden 绑定

控制元素的隐藏状态（与 visible 相反）。

```html
<div data-bind="hidden: isHidden">
    当 isHidden 为 true 时隐藏
</div>

<script>
var viewModel = {
    isHidden: ko.observable(false)
};
</script>
```

### css 绑定

动态添加或删除 CSS 类。

```html
<div data-bind="css: { active: isActive, error: hasError }">
    根据条件应用不同的类
</div>

<script>
var viewModel = {
    isActive: ko.observable(true),
    hasError: ko.observable(false)
};
</script>
```

### style 绑定

直接设置 CSS 样式。

```html
<div data-bind="style: { 
    color: textColor,
    backgroundColor: bgColor,
    fontSize: fontSize + 'px'
}">样式测试</div>

<script>
var viewModel = {
    textColor: ko.observable('red'),
    bgColor: ko.observable('#f0f0f0'),
    fontSize: ko.observable(16)
};
</script>
```

## 属性绑定

### attr 绑定

设置 HTML 属性。

```html
<img data-bind="attr: { 
    src: imageUrl,
    alt: imageAlt,
    title: imageTitle
}" />

<script>
var viewModel = {
    imageUrl: ko.observable('images/photo.jpg'),
    imageAlt: ko.observable('照片描述'),
    imageTitle: ko.observable('鼠标悬停提示')
};
</script>
```

## 组合使用

### 多重绑定

在同一个元素上使用多个绑定。

```html
<div data-bind="
    text: content,
    visible: isVisible,
    css: { highlight: isHighlighted },
    style: { color: textColor }
">
</div>

<script>
var viewModel = {
    content: ko.observable("内容"),
    isVisible: ko.observable(true),
    isHighlighted: ko.observable(false),
    textColor: ko.observable('blue')
};
</script>
```

## 高级用法

### 1. 条件样式

```html
<div data-bind="css: {
    'success': status() === 'success',
    'warning': status() === 'warning',
    'error': status() === 'error'
}">状态指示器</div>

<script>
var viewModel = {
    status: ko.observable('success')
};
</script>
```

### 2. 动态样式计算

```html
<div data-bind="style: {
    backgroundColor: getBackgroundColor(),
    opacity: isActive() ? 1 : 0.5
}"></div>

<script>
var viewModel = {
    isActive: ko.observable(true),
    getBackgroundColor: function() {
        return this.isActive() ? '#4CAF50' : '#ccc';
    }
};
</script>
```

### 3. HTML 安全处理

```html
<div data-bind="html: sanitizedContent"></div>

<script>
var viewModel = {
    rawContent: ko.observable('<script>alert("危险代码")</script>'),
    sanitizedContent: ko.computed(function() {
        // 使用 DOMPurify 或其他库清理 HTML
        return DOMPurify.sanitize(this.rawContent());
    })
};
</script>
```

## 最佳实践

### 1. 性能优化

```javascript
// 使用计算属性缓存复杂的样式计算
function ViewModel() {
    this.items = ko.observableArray([/*...*/]);
    
    this.containerStyle = ko.computed(function() {
        return {
            height: this.items().length * 50 + 'px',
            backgroundColor: this.items().length > 5 ? '#f0f0f0' : '#fff'
        };
    }, this);
}
```

### 2. 可维护性

```javascript
// 将样式逻辑封装在视图模型中
function ViewModel() {
    this.status = ko.observable('active');
    
    this.statusClass = ko.computed(function() {
        var status = this.status();
        return {
            'active': status === 'active',
            'inactive': status === 'inactive',
            'pending': status === 'pending'
        };
    }, this);
}
```

### 3. 响应式设计

```html
<div data-bind="css: responsiveClasses">
    响应式内容
</div>

<script>
function ViewModel() {
    this.windowWidth = ko.observable(window.innerWidth);
    
    // 监听窗口大小变化
    window.addEventListener('resize', function() {
        this.windowWidth(window.innerWidth);
    }.bind(this));
    
    this.responsiveClasses = ko.computed(function() {
        var width = this.windowWidth();
        return {
            'mobile': width < 768,
            'tablet': width >= 768 && width < 1024,
            'desktop': width >= 1024
        };
    }, this);
}
</script>
```

## 常见问题

### 1. 绑定不生效

```html
<!-- 错误 -->
<div data-bind="css: { active: isActive }">
    <!-- isActive 未定义在视图模型中 -->
</div>

<!-- 正确 -->
<div data-bind="css: { active: isActive }">
    <!-- 确保在视图模型中定义了 isActive -->
</div>
<script>
var viewModel = {
    isActive: ko.observable(true)
};
ko.applyBindings(viewModel);
</script>
```

### 2. 样式闪烁

```javascript
// 使用 with 绑定避免样式闪烁
<div data-bind="with: data">
    <div data-bind="css: { loaded: isLoaded }">
        内容
    </div>
</div>

var viewModel = {
    data: ko.observable(null)
};

// 加载数据后再设置
loadData().then(function(result) {
    viewModel.data(result);
});
```

### 3. 性能问题

```javascript
// 避免在绑定中进行复杂计算
<!-- 不推荐 -->
<div data-bind="style: { 
    transform: 'rotate(' + (Math.sin(Date.now() / 1000) * 360) + 'deg)'
}"></div>

<!-- 推荐 -->
<div data-bind="style: { transform: rotation }"></div>
<script>
function ViewModel() {
    this.rotation = ko.observable('rotate(0deg)');
    
    // 使用 requestAnimationFrame 更新
    function update() {
        this.rotation('rotate(' + (Math.sin(Date.now() / 1000) * 360) + 'deg)');
        requestAnimationFrame(update.bind(this));
    }
    update.bind(this)();
}
</script>
```

## 下一步

- 学习 [表单控件绑定](form.md)
- 了解 [流程控制绑定](flow-control.md)
- 探索 [自定义绑定](../advanced/custom-bindings.md) 