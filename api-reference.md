# API 参考

本文提供 Knockout.js 的 API 参考文档，包含所有核心功能和常用方法。

## 核心 API

### ko.observable

创建可观察对象。

```javascript
// 创建
var name = ko.observable("张三");

// 读取值
console.log(name());

// 设置值
name("李四");

// 订阅变化
name.subscribe(function(newValue) {
    console.log("值变化为：" + newValue);
});
```

### ko.observableArray

创建可观察数组。

```javascript
// 创建
var items = ko.observableArray(["项目1", "项目2"]);

// 数组操作
items.push("项目3");           // 添加项目
items.pop();                   // 移除最后一项
items.unshift("项目0");        // 添加到开头
items.shift();                // 移除第一项
items.reverse();              // 反转数组
items.sort();                 // 排序
items.splice(1, 1, "新项目");  // 替换项目

// Knockout 特有方法
items.remove("项目2");         // 移除特定项
items.removeAll();            // 移除所有项
items.destroy("项目1");        // 销毁项目
```

### ko.computed

创建计算属性。

```javascript
// 基本计算属性
this.fullName = ko.computed(function() {
    return this.firstName() + " " + this.lastName();
}, this);

// 可写计算属性
this.fullName = ko.computed({
    read: function() {
        return this.firstName() + " " + this.lastName();
    },
    write: function(value) {
        var parts = value.split(" ");
        this.firstName(parts[0]);
        this.lastName(parts[1]);
    }
}, this);

// 纯计算属性
this.total = ko.pureComputed(function() {
    return this.price() * this.quantity();
}, this);
```

### ko.applyBindings

应用绑定到 DOM 元素。

```javascript
// 应用到整个文档
ko.applyBindings(viewModel);

// 应用到特定元素
ko.applyBindings(viewModel, document.getElementById('someElement'));

// 应用到特定元素并提供额外上下文
var context = {
    $data: viewModel,
    $root: rootViewModel
};
ko.applyBindings(context, element);
```

## 绑定 API

### 文本和外观绑定

```html
<!-- 文本绑定 -->
<span data-bind="text: message"></span>

<!-- HTML 绑定 -->
<div data-bind="html: htmlContent"></div>

<!-- 可见性绑定 -->
<div data-bind="visible: isVisible"></div>

<!-- 样式绑定 -->
<div data-bind="css: { active: isActive, error: hasError }"></div>

<!-- 样式绑定 -->
<div data-bind="style: { color: textColor, display: isVisible() ? 'block' : 'none' }"></div>

<!-- 属性绑定 -->
<img data-bind="attr: { src: imageUrl, alt: imageAlt }">
```

### 表单绑定

```html
<!-- 值绑定 -->
<input data-bind="value: userName">

<!-- 即时值绑定 -->
<input data-bind="textInput: searchTerm">

<!-- 选中状态绑定 -->
<input type="checkbox" data-bind="checked: isAgree">

<!-- 启用/禁用绑定 -->
<button data-bind="enable: isValid">提交</button>

<!-- 选项绑定 -->
<select data-bind="options: availableCountries,
                  optionsText: 'name',
                  optionsValue: 'id',
                  value: selectedCountry"></select>
```

### 流程控制绑定

```html
<!-- if 绑定 -->
<div data-bind="if: isLoggedIn">
    <h2>欢迎回来</h2>
</div>

<!-- ifnot 绑定 -->
<div data-bind="ifnot: isLoggedIn">
    <p>请登录</p>
</div>

<!-- foreach 绑定 -->
<ul data-bind="foreach: items">
    <li data-bind="text: name"></li>
</ul>

<!-- with 绑定 -->
<div data-bind="with: userProfile">
    <p data-bind="text: name"></p>
    <p data-bind="text: email"></p>
</div>
```

## 组件 API

### 注册组件

```javascript
// 基本组件
ko.components.register('user-profile', {
    viewModel: function(params) {
        this.name = params.name;
    },
    template: '<div class="user-profile">...</div>'
});

// 异步组件
ko.components.register('async-component', {
    viewModel: { require: 'path/to/viewModel' },
    template: { require: 'text!path/to/template.html' }
});

// 使用工厂函数
ko.components.register('factory-component', {
    viewModel: {
        createViewModel: function(params, componentInfo) {
            return new ViewModel(params);
        }
    },
    template: '...'
});
```

### 使用组件

```html
<!-- 基本用法 -->
<user-profile params="name: userName"></user-profile>

<!-- 使用 component 绑定 -->
<div data-bind="component: {
    name: 'user-profile',
    params: { name: userName }
}"></div>

<!-- 动态组件 -->
<div data-bind="component: {
    name: currentComponent,
    params: componentParams
}"></div>
```

## 工具函数

### ko.utils

```javascript
// 数组操作
ko.utils.arrayForEach(array, callback);
ko.utils.arrayFirst(array, predicate);
ko.utils.arrayFilter(array, predicate);
ko.utils.arrayMap(array, mapping);

// 对象操作
ko.utils.extend(target, source);
ko.utils.objectForEach(object, callback);

// DOM 操作
ko.utils.parseHtmlFragment(html);
ko.utils.setHtml(element, html);
ko.utils.registerEventHandler(element, eventType, handler);
```

### ko.extenders

```javascript
// 创建扩展
ko.extenders.numeric = function(target, precision) {
    var result = ko.computed({
        read: target,
        write: function(newValue) {
            var current = target(),
                rounded = Math.round(newValue * Math.pow(10, precision)) / Math.pow(10, precision);
            target(rounded);
        }
    });
    return result;
};

// 使用扩展
var myNumber = ko.observable(0).extend({ numeric: 2 });
```

## 调试 API

### ko.dataFor 和 ko.contextFor

```javascript
// 获取元素的数据上下文
var element = document.getElementById('myElement');
var data = ko.dataFor(element);
var context = ko.contextFor(element);

// 访问特殊属性
var $parent = context.$parent;
var $root = context.$root;
var $data = context.$data;
```

### ko.isObservable 和相关函数

```javascript
// 检查可观察对象
ko.isObservable(obj);         // 是否是可观察对象
ko.isWriteableObservable(obj); // 是否是可写的可观察对象
ko.isComputed(obj);           // 是否是计算属性

// 获取原始值
ko.unwrap(obj);               // 获取可观察对象的值
```

## 配置选项

### ko.options

```javascript
// 全局配置
ko.options.deferUpdates = true;           // 延迟更新
ko.options.useOnlyNativeEvents = true;    // 只使用原生事件
ko.options.foreachHidesDestroyed = true;  // foreach 隐藏已销毁项

// 错误处理
ko.options.onError = function(error) {
    console.error('Knockout 错误:', error);
};
```

## 扩展 API

### 自定义绑定

```javascript
// 注册自定义绑定
ko.bindingHandlers.customBinding = {
    init: function(element, valueAccessor, allBindings, viewModel, bindingContext) {
        // 初始化代码
    },
    update: function(element, valueAccessor, allBindings, viewModel, bindingContext) {
        // 更新代码
    }
};

// 使用自定义绑定
<div data-bind="customBinding: value"></div>
```

### 自定义绑定提供者

```javascript
// 创建自定义绑定提供者
function CustomProvider() {
    this.nodeHasBindings = function(node) {
        // 检查节点是否有绑定
        return true/false;
    };
    
    this.getBindings = function(node, bindingContext) {
        // 返回绑定对象
        return {
            text: someValue,
            click: someHandler
        };
    };
}

// 注册提供者
ko.bindingProvider.instance = new CustomProvider();
```

## 下一步

- 了解 [最佳实践](advanced/performance.md)
- 学习 [组件开发](advanced/components.md)
- 探索 [自定义绑定](advanced/custom-bindings.md) 