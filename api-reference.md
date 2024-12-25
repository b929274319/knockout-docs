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

## Knockout 特殊功能

### 特殊注释语法

Knockout 提供了特殊的注释语法，用于在不支持 data-bind 属性的环境中使用绑定。

**语法**
```html
<!-- ko bindingName: value -->
内容
<!-- /ko -->
```

**示例**
```html
<!-- ko if: isVisible -->
    <div>只在 isVisible 为 true 时显示</div>
<!-- /ko -->

<!-- ko foreach: items -->
    <div>当前项: <span data-bind="text: $data"></span></div>
<!-- /ko -->
```

### 特殊上下文变量

在绑定表达式中可以使用的特殊变量：

- `$data`: 当前上下文的数据项
- `$parent`: 父级上下文的数据项
- `$parents[n]`: 第 n 层父级上下文
- `$root`: 最顶层的视图模型
- `$index`: 在 foreach 循环中的索引（从0开始）
- `$context`: 当前绑定上下文对象
- `$element`: 当前 DOM 元素
- `$component`: 当前组件的视图模型

**示例**
```html
<div data-bind="foreach: items">
    <span data-bind="text: $data"></span>
    <span data-bind="text: $parent.title"></span>
    <span data-bind="text: $root.globalProperty"></span>
    <span data-bind="text: '索引: ' + $index()"></span>
</div>
```

### 语法糖

#### 1. 链式写法

```javascript
// 普通写法
observable().property().method()

// 链式写法
observable.chain().property().method()
```

#### 2. 自动展开

在某些绑定中，Knockout 会自动展开可观察对象，无需手动调用：

```html
<!-- 自动展开 -->
<div data-bind="text: name"></div>

<!-- 等同于 -->
<div data-bind="text: name()"></div>
```

#### 3. 简写绑定

```html
<!-- 标准写法 -->
<input data-bind="value: prop, valueUpdate: 'afterkeydown'">

<!-- 简写 -->
<input data-bind="textInput: prop">
```

### 特殊 API 功能

#### 1. 扩展器 (Extenders)

用于向可观察对象添加自定义功能：

```javascript
// 定义扩展器
ko.extenders.numeric = function(target, precision) {
    return ko.pureComputed({
        read: target,
        write: function(newValue) {
            var val = parseFloat(newValue);
            target(isNaN(val) ? 0 : val.toFixed(precision));
        }
    });
};

// 使用扩展器
var value = ko.observable(123.456).extend({ numeric: 2 });
```

#### 2. 订阅器 (Subscriptions)

高级订阅功能：

```javascript
// 即时订阅
var subscription = observable.subscribe(callback);

// 延迟订阅
var subscription = observable.subscribeChanged(function(newValue, oldValue) {
    console.log('从', oldValue, '变为', newValue);
});

// 取消订阅
subscription.dispose();
```

#### 3. 计算属性选项

```javascript
ko.computed({
    read: function() { /* 读取逻辑 */ },
    write: function(value) { /* 写入逻辑 */ },
    pure: true,                    // 纯计算属性
    deferEvaluation: true,         // 延迟计算
    disposeWhen: function() { /* 销毁条件 */ }
});
```

#### 4. 手动依赖跟踪

```javascript
ko.dependencyDetection.ignore(function() {
    // 此处代码不会建立依赖关系
    observable();
});
```

#### 5. 自定义绑定

```javascript
ko.bindingHandlers.customBinding = {
    init: function(element, valueAccessor, allBindings, viewModel, bindingContext) {
        // 初始化逻辑
    },
    update: function(element, valueAccessor, allBindings, viewModel, bindingContext) {
        // 更新逻辑
    }
};
```

### 调试技巧

#### 1. 依赖跟踪

```javascript
observable.subscribe(function() {
    console.log('值已更改:', arguments);
    console.trace(); // 显示调用栈
});
```

#### 2. 计算属性调试

```javascript
var debugComputed = ko.computed(function() {
    console.log('计算中...');
    return observable() + 1;
}).extend({ rateLimit: 500 });
```

#### 3. 绑定调试

```javascript
ko.bindingHandlers.debug = {
    init: function() {
        debugger;
    }
};
``` 

## Knockout 特殊功能使用指南

### 特殊注释语法使用建议

**推荐场景**
- 在动态生成的 HTML 内容中使用
- 在不允许使用自定义属性的严格 HTML 环境中
- 需要嵌套多层控制流时（如 if 和 foreach 的组合）

**不推荐场景**
- 普通的静态 HTML 页面（优先使用 data-bind）
- 简单的单一绑定场景
- 性能关键的渲染场景（注释语法解析较慢）

**注意事项**
```html
<!-- 推荐：使用注释语法处理复杂嵌套 -->
<!-- ko if: isLoggedIn -->
    <!-- ko foreach: userPermissions -->
        <div>权限: <span data-bind="text: $data"></span></div>
    <!-- /ko -->
<!-- /ko -->

<!-- 不推荐：简单场景使用注释语法 -->
<!-- ko text: userName --><!-- /ko -->
<!-- 应该使用：-->
<span data-bind="text: userName"></span>
```

### 特殊上下文变量使用建议

**推荐使用场景**
- `$data`: 在 foreach 循环中访问当前项
- `$parent`: 在嵌套组件中访问父级数据
- `$root`: 访问全局共享数据
- `$component`: 在组件内部访问组件的视图模型

**不推荐使用场景**
- 滥用 `$parent` 跨越多层访问数据（应该通过参数传递）
- 过度依赖 `$root` 存储全局状态（考虑使用依赖注入）

**最佳实践**
```html
<!-- 推荐：清晰的数据访问 -->
<ul data-bind="foreach: items">
    <li>
        <!-- 使用 $data 访问当前项 -->
        <span data-bind="text: $data.name"></span>
        <!-- 使用 $parent 访问外层数据 -->
        <button data-bind="click: $parent.removeItem">删除</button>
    </li>
</ul>

<!-- 不推荐：过度使用 $parents -->
<span data-bind="text: $parents[2].someData"></span>
<!-- 应该重构数据结构或通过参数传递 -->
```

### 语法糖使用建议

**推荐使用场景**
1. 自动展开：
   - 在绑定表达式中使用可观察对象
   - 在计算属性中组合多个可观察对象

2. 链式写法：
   - 处理复杂的数据转换
   - 需要多步操作的场景

3. 简写绑定：
   - 表单输入处理
   - 常见的 UI 交互场景

**注意事项**
```javascript
// 推荐：合理使用自动展开
<div data-bind="text: userName"></div>

// 不推荐：在性能关键场景频繁使用链式操作
items.chain()
     .filter()
     .map()
     .sort()
     .value();

// 应该使用计算属性缓存结果
this.processedItems = ko.computed(function() {
    return this.items().filter().map().sort();
});
```

### 特殊 API 功能使用建议

**1. 扩展器 (Extenders)**
推荐场景：
- 添加数据验证
- 实现数据转换
- 添加通用功能（如防抖、节流）

```javascript
// 推荐：创建可复用的扩展器
ko.extenders.validate = function(target, rules) {
    target.hasError = ko.observable(false);
    target.errorMessage = ko.observable();
    
    return target;
};

// 使用扩展器
this.userName = ko.observable().extend({
    validate: {
        required: true,
        minLength: 3
    }
});
```

**2. 订阅器 (Subscriptions)**
推荐场景：
- 监控关键数据变化
- 实现数据同步
- 触发副作用

注意事项：
- 及时清理订阅，避免内存泄漏
- 使用 disposeWhen 自动管理订阅生命周期

```javascript
// 推荐：使用 computed 自动管理订阅
ko.computed(function() {
    // 订阅会随计算属性自动清理
    this.data.subscribe(this.handleChange, this);
}, viewModel);

// 不推荐：手动订阅without清理
var subscription = this.data.subscribe(this.handleChange);
// 容易忘记调用 subscription.dispose()
```

**3. 计算属性选项**
推荐场景：
- `pure`: 用于纯数据计算，提高性能
- `deferEvaluation`: 延迟计算大量数据
- `disposeWhen`: 自动清理资源

```javascript
// 推荐：使用纯计算属性优化性能
this.total = ko.pureComputed(function() {
    return this.items().reduce((sum, item) => sum + item.price(), 0);
});

// 推荐：延迟计算大量数据
this.filteredItems = ko.computed({
    read: function() {
        return heavyComputation();
    },
    deferEvaluation: true
});
```

### 调试技巧使用建议

**推荐场景**
1. 开发环境调试：
   - 使用 subscribe 跟踪数据变化
   - 添加计算属性日志
   - 使用 debugger 断点

2. 生产环境监控：
   - 使用 error 回调捕获异常
   - 添加性能监控点

```javascript
// 开发环境：详细日志
if (DEV_MODE) {
    this.data.subscribe(function(newValue) {
        console.log('数据变化:', newValue);
        console.trace('变化调用栈');
    });
}

// 生产环境：错误监控
ko.options.onError = function(error) {
    // 上报错误到监控系统
    errorTracker.capture(error);
};
```

**最佳实践总结**
1. 选择合适的特性：
   - 简单场景用标准绑定
   - 复杂场景再考虑特殊语法
   - 性能优先场景使用纯计算属性

2. 数据访问原则：
   - 保持数据流向清晰
   - 避免过度嵌套
   - 合理使用计算属性缓存

3. 性能优化：
   - 使用 pure 优化计算
   - 延迟计算大数据
   - 及时清理订阅

4. 调试友好：
   - 添加适量日志
   - 使用有意义的变量名
   - 保持代码结构清晰

// ... existing code ... 