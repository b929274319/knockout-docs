# API 参考

> 本文提供 Knockout.js 的完整 API 参考文档。如果你是初学者，建议先阅读 [快速开始](guide/quickstart.md) 和 [基本概念](guide/concepts.md)。

## 目录

- [核心 API](#核心-api)
  - [ko.observable](#koobservable) - 创建可观察对象
  - [ko.observableArray](#koobservablearray) - 创建可观察数组
  - [ko.computed](#kocomputed) - 创建计算属性
  - [ko.applyBindings](#koapplybindings) - 应用绑定
- [绑定 API](#绑定-api)
  - [文本和外观](#文本和外观绑定)
  - [表单控件](#表单绑定)
  - [流程控制](#流程控制绑定)
- [组件 API](#组件-api)
  - [注册组件](#注册组件)
  - [使用组件](#使用组件)
- [工具函数](#工具函数)
  - [ko.utils](#koutils)
  - [ko.extenders](#koextenders)
- [调试 API](#调试-api)
- [配置选项](#配置选项)
- [扩展 API](#扩展-api)

## 核心 API

### ko.observable

创建一个可观察对象，用于跟踪数据变化。

**语法**
```javascript
ko.observable([initialValue])
```

**参数**
- `initialValue` (可选): 初始值，可以是任意类型

**返回值**
- 返回一个可观察对象函数，可以通过 `()` 读取值，通过 `(newValue)` 设置值

**方法**
- `subscribe(callback[, callbackTarget, event])`: 订阅值变化
  - `callback`: 值变化时的回调函数
  - `callbackTarget` (可选): 回调函数的 this 上下文
  - `event` (可选): 事件类型，可选值：'change'（默认）、'beforeChange'
- `peek()`: 获取当前值，不建立依赖关系
- `valueHasMutated()`: 手动触发值变化通知
- `extend(extenders)`: 扩展可观察对象

**示例**
```javascript
// 基本用法
var name = ko.observable("张三");
console.log(name());         // 输出: "张三"
name("李四");               // 设置新值
console.log(name());         // 输出: "李四"

// 订阅变化
name.subscribe(function(newValue) {
    console.log("值变化为：" + newValue);
}, null, "change");

// 使用 peek
console.log(name.peek());    // 读取值但不建立依赖

// 使用扩展
var limitedName = ko.observable().extend({ maxLength: 10 });
```

### ko.observableArray

创建一个可观察数组，继承了普通可观察对象的所有功能，并添加了数组特有的操作方法。

**语法**
```javascript
ko.observableArray([initialArray])
```

**参数**
- `initialArray` (可选): 初始数组

**返回值**
- 返回一个可观察数组函数，可以通过 `()` 读取数组，通过 `(newArray)` 设置整个数组

**数组方法**
所有原生数组方法都可用，并且会自动触发通知：
- `push(item)`: 添加项目到末尾
- `pop()`: 移除并返回最后一项
- `unshift(item)`: 添加项目到开头
- `shift()`: 移除并返回第一项
- `reverse()`: 反转数组
- `sort([compareFunction])`: 排序数组
- `splice(index, howMany[, ...items])`: 删除/插入项目

**Knockout 特有方法**
- `remove(item/predicate)`: 移除匹配的项目
- `removeAll([items])`: 移除所有或指定项目
- `destroy(item/predicate)`: 标记项目为已销毁
- `destroyAll([items])`: 标记所有或指定项目为已销毁
- `indexOf(item)`: 查找项目索引
- `slice(start[, end])`: 返回数组的一部分

**示例**
```javascript
// 基本用法
var items = ko.observableArray(["项目1", "项目2"]);

// 数组操作
items.push("项目3");                    // 添加到末尾
console.log(items().length);            // 输出: 3
console.log(items()[0]);                // 输出: "项目1"

// 使用 Knockout 特有方法
items.remove("项目2");                  // 移除特定项
items.removeAll(["项目1", "项目3"]);    // 移除多个项
items.destroyAll();                     // 标记所有项为已销毁

// 链式操作
items().filter(function(item) {
    return item.startsWith("项目");
}).forEach(function(item) {
    console.log(item);
});

// 订阅变化
items.subscribe(function(changes) {
    console.log("数组变化：", changes);
}, null, "arrayChange");
```

### ko.computed

创建一个计算属性，其值依赖于其他可观察对象。

**语法**
```javascript
ko.computed(evaluator[, targetObject, options])
// 或
ko.computed(options)
```

**参数**
- `evaluator`: 计算函数，返回计算后的值
- `targetObject` (可选): 计算函数的 this 上下文
- `options` (可选): 配置对象，包含以下属性：
  - `read`: 读取函数
  - `write` (可选): 写入函数
  - `owner`: 函数的 this 上下文
  - `pure`: 是否是纯计算属性
  - `deferEvaluation`: 是否延迟计算
  - `disposeWhenNodeIsRemoved`: 当指定 DOM 节点被移除时销毁
  - `disposeWhen`: 返回 true 时销毁的函数

**返回值**
- 返回一个计算属性函数，可以通过 `()` 读取值，如果定义了 write 函数，也可以通过 `(newValue)` 设置值

**示例**
```javascript
// 基本计算属性
var firstName = ko.observable("张");
var lastName = ko.observable("三");
var fullName = ko.computed(function() {
    return this.firstName() + " " + this.lastName();
}, this);

console.log(fullName());  // 输出: "张 三"
firstName("李");         // fullName 自动更新
console.log(fullName());  // 输出: "李 三"

// 可写计算属性
var fullName = ko.computed({
    read: function() {
        return this.firstName() + " " + this.lastName();
    },
    write: function(value) {
        var parts = value.split(" ");
        this.firstName(parts[0]);
        this.lastName(parts[1]);
    }
}, this);

fullName("王 五");      // 自动更新 firstName 和 lastName

// 纯计算属性（性能优化）
var total = ko.pureComputed(function() {
    return this.price() * this.quantity();
}, this);

// 延迟计算
var expensive = ko.computed({
    read: function() {
        // 复杂计算
        return heavyCalculation();
    },
    deferEvaluation: true  // 直到首次访问才计算
});
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

用于控制元素的文本内容和视觉外观。

#### text 绑定

将文本内容绑定到元素。

**语法**
```html
<element data-bind="text: value"></element>
```

**参数**
- `value`: 字符串、数字或可观察对象

**示例**
```html
<span data-bind="text: message"></span>
<span data-bind="text: name() + ' (' + age() + '岁)'"></span>
```

#### html 绑定

将 HTML 内容绑定到元素。

**语法**
```html
<element data-bind="html: htmlValue"></element>
```

**参数**
- `htmlValue`: 包含 HTML 的字符串或可观察对象

**注意事项**
- 请确保 HTML 内容是安全的，避免 XSS 攻击
- 如果只需显示文本，优先使用 text 绑定

**示例**
```html
<div data-bind="html: description"></div>
```

#### visible 绑定

控制元素的可见性。

**语法**
```html
<element data-bind="visible: shouldShow"></element>
```

**参数**
- `shouldShow`: 布尔值或返回布尔值的表达式

**工作原理**
- true: 显示元素（display: ''）
- false: 隐藏元素（display: none）

**示例**
```html
<div data-bind="visible: isVisible">
    仅在 isVisible 为 true 时显示
</div>

<div data-bind="visible: items().length > 0">
    有项目时显示
</div>
```

#### css 绑定

动态添加或移除 CSS 类。

**语法**
```html
<element data-bind="css: { className: condition }"></element>
```

**参数**
- 对象，其中：
  - key: CSS 类名
  - value: 布尔值或返回布尔值的表达式

**示例**
```html
<!-- 单个类 -->
<div data-bind="css: { active: isActive }"></div>

<!-- 多个类 -->
<div data-bind="css: { 
    active: isActive,
    error: hasError,
    highlight: isSelected
}"></div>

<!-- 使用计算属性 -->
<div data-bind="css: cssClasses"></div>
```

```javascript
this.cssClasses = ko.computed(function() {
    return {
        active: this.isActive(),
        error: this.hasError(),
        highlight: this.isSelected()
    };
}, this);
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

### 定义绑定

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

## 代码片段

### 基本模式

```javascript
// 基本视图模型
function ViewModel() {
    // 可观察属性
    this.firstName = ko.observable("张");
    this.lastName = ko.observable("三");
    
    // 计算属性
    this.fullName = ko.computed(function() {
        return this.firstName() + " " + this.lastName();
    }, this);
    
    // 事件处理
    this.save = function() {
        // 处理保存
        var data = {
            firstName: this.firstName(),
            lastName: this.lastName()
        };
        // 发送到服务器...
    }.bind(this);
}

// 应用绑定
ko.applyBindings(new ViewModel());
```

### 常见模式

```javascript
// 延迟加载
function ViewModel() {
    var self = this;
    self.items = ko.observableArray();
    self.isLoading = ko.observable(false);
    
    self.loadItems = function() {
        self.isLoading(true);
        fetch('/api/items')
            .then(response => response.json())
            .then(data => {
                self.items(data);
                self.isLoading(false);
            });
    };
}

// 表单处理
function FormViewModel() {
    var self = this;
    
    // 表单字段
    self.username = ko.observable().extend({
        required: true,
        minLength: 3
    });
    self.email = ko.observable().extend({
        required: true,
        email: true
    });
    
    // 验证状态
    self.errors = ko.validation.group(self);
    
    // 提交处理
    self.submit = function() {
        if (self.errors().length === 0) {
            // 提交表单...
        }
    };
}

// 分页列表
function PaginatedList() {
    var self = this;
    
    self.items = ko.observableArray();
    self.currentPage = ko.observable(1);
    self.itemsPerPage = ko.observable(10);
    self.totalItems = ko.observable(0);
    
    self.totalPages = ko.computed(function() {
        return Math.ceil(self.totalItems() / self.itemsPerPage());
    });
    
    self.loadPage = function(page) {
        // 加载指定页的数据...
    };
}
```

## 最佳实践

### 1. 性能优化

```javascript
// 使用纯计算属性
this.total = ko.pureComputed(function() {
    return this.price() * this.quantity();
}, this);

// 避免不必要的计算
this.items = ko.observableArray();
this.filteredItems = ko.computed(function() {
    var search = this.searchTerm().toLowerCase();
    return this.items().filter(function(item) {
        return item.name().toLowerCase().includes(search);
    });
}, this).extend({ rateLimit: { timeout: 500, method: "notifyWhenChangesStop" } });
```

### 2. 内存管理

```javascript
// 正确销毁订阅
var subscription = myObservable.subscribe(function() {
    // 处理变化
});

// 在不需要时取消订阅
subscription.dispose();

// 使用 computedContext
ko.computed(function() {
    // 在计算属性内部订阅
    myObservable.subscribe(function() {
        // 处理变化
    }, null, "change", this);
}, this);
```

### 3. 组件化

```javascript
// 定义可重用组件
ko.components.register('user-card', {
    viewModel: function(params) {
        this.user = params.user;
        this.onEdit = params.onEdit;
    },
    template: 
        '<div class="user-card">\
            <h3 data-bind="text: user().name"></h3>\
            <button data-bind="click: onEdit">编辑</button>\
        </div>'
});

// 使用组件
<user-card params="
    user: currentUser,
    onEdit: editUser
"></user-card>
```

### 4. 错误处理

```javascript
// 全局错误处理
ko.options.onError = function(error) {
    console.error('Knockout 错误:', error);
    // 上报错误...
};

// 优雅降级
<div data-bind="if: data">
    <!-- 有数据时显示 -->
    <div data-bind="with: data">
        <span data-bind="text: name"></span>
    </div>
</div>
<div data-bind="ifnot: data">
    <!-- 无数据时显示 -->
    <p>暂无数据</p>
</div>
```

## 调试技巧

### 1. 检查绑定上下文

```javascript
// 在控制台中使用
var element = document.querySelector('#myElement');
var context = ko.contextFor(element);
var data = ko.dataFor(element);

console.log('上下文:', context);
console.log('数据:', data);
```

### 2. 追踪依赖

```javascript
// 在计算属性中添加日志
this.total = ko.computed(function() {
    console.log('计算 total...');
    console.log('price:', this.price());
    console.log('quantity:', this.quantity());
    return this.price() * this.quantity();
}, this);
```

### 3. 监控变化

```javascript
// 订阅所有变化
myObservable.subscribe(function(newValue) {
    console.log('值变化为:', newValue);
}, null, 'change');

// 订阅数组变化
myArray.subscribe(function(changes) {
    console.log('数组变化:', changes);
}, null, 'arrayChange');
```

## 相关资源

- [Knockout.js 官方文档](http://knockoutjs.com/documentation/introduction.html)
- [Knockout.js GitHub 仓库](https://github.com/knockout/knockout)
- [Knockout.js 中文社区](https://github.com/b929274319/knockout-docs) 