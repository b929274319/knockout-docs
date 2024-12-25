# 可观察对象（Observables）

可观察对象是 Knockout.js 最核心的特性，它能够自动追踪数据变化并更新 UI。本文将详细介绍可观察对象的使用方法和高级特性。

## 基础概念

### 什么是可观察对象？

可观察对象是一个可以通知订阅者其值已更改的对象。在 Knockout 中，可观察对象是通过 `ko.observable()` 函数创建的。

```javascript
// 创建可观察对象
var name = ko.observable("张三");

// 读取值
console.log(name()); // 输出: "张三"

// 设置值
name("李四"); // UI 会自动更新
```

### 为什么需要可观察对象？

1. **自动 UI 更新**
   - 当数据改变时，UI 自动更新
   - 无需手动操作 DOM

2. **依赖追踪**
   - 自动检测数据之间的依赖关系
   - 高效地更新相关联的数据

3. **双向数据绑定**
   - 用户输入自动同步到数据模型
   - 数据模型更改自动反映到 UI

## 基本用法

### 1. 创建可观察对象

```javascript
// 基本类型
var name = ko.observable("张三");
var age = ko.observable(25);
var isActive = ko.observable(true);

// 对象
var user = ko.observable({
    name: "张三",
    age: 25
});

// 空值
var nullableValue = ko.observable(null);
var undefinedValue = ko.observable();
```

### 2. 读取和设置值

```javascript
// 读取值
var currentName = name();

// 设置值
name("李四");

// 链式调用
name("王五");
age(30);

// 使用现有值设置新值
name(name() + "先生");
```

### 3. 订阅变化

```javascript
// 基本订阅
name.subscribe(function(newValue) {
    console.log("名字改变为：" + newValue);
});

// 包含旧值的订阅
name.subscribe(function(newValue, oldValue) {
    console.log("名字从 " + oldValue + " 改变为 " + newValue);
}, null, "beforeChange");

// 立即执行和取消订阅
var subscription = name.subscribe(function(value) {
    console.log(value);
});
subscription.dispose(); // 取消订阅
```

## 高级特性

### 1. 可写计算属性

```javascript
function ViewModel() {
    var self = this;
    self.firstName = ko.observable("张");
    self.lastName = ko.observable("三");
    
    self.fullName = ko.computed({
        read: function() {
            return self.firstName() + " " + self.lastName();
        },
        write: function(value) {
            var parts = value.split(" ");
            self.firstName(parts[0]);
            self.lastName(parts[1]);
        }
    });
}
```

### 2. 扩展可观察对象

```javascript
ko.extenders.numeric = function(target, precision) {
    var result = ko.computed({
        read: target,
        write: function(newValue) {
            var current = target();
            var rounded = Math.round(newValue * Math.pow(10, precision)) / Math.pow(10, precision);
            target(rounded);
        }
    });
    return result;
};

// 使用扩展
var price = ko.observable(0).extend({ numeric: 2 });
```

### 3. 延迟更新

```javascript
ko.options.deferUpdates = true; // 全局设置

// 或者针对特定可观察对象
var delayedValue = ko.observable().extend({ deferred: true });
```

## 最佳实践

### 1. 性能优化

```javascript
// 批量更新
ko.computed(function() {
    // 在一个计算属性中进行多个更新
    name("李四");
    age(30);
    isActive(true);
}).extend({ rateLimit: 50 });

// 使用 rateLimit 限制更新频率
var throttledValue = ko.observable().extend({ rateLimit: 400 });
```

### 2. 内存管理

```javascript
// 及时清理订阅
function SomeViewModel() {
    var self = this;
    self.someValue = ko.observable();
    
    // 保存订阅引用
    self.subscription = self.someValue.subscribe(function() {
        // 处理变化
    });
    
    // 清理方法
    self.dispose = function() {
        self.subscription.dispose();
    };
}
```

### 3. 调试技巧

```javascript
// 添加调试信息
var debuggableValue = ko.observable("初始值").extend({ 
    trackChange: true 
});

// 导出纯 JavaScript 对象
function exportData() {
    var plainData = ko.toJS(viewModel);
    console.log(plainData);
}
```

## 常见问题

### 1. 值不更新

```javascript
// 错误方式
var user = ko.observable({ name: "张三" });
user().name = "李四"; // UI 不会更新

// 正确方式
var user = ko.observable({ name: "张三" });
var newUser = user();
newUser.name = "李四";
user(newUser); // UI 会更新
```

### 2. 忘记加括号

```javascript
// 错误
<span data-bind="text: name"></span>

// 正确
<span data-bind="text: name()"></span>

// 注意：在 data-bind 属性中不需要加括号
```

### 3. 循环依赖

```javascript
// 可能导致死循环
var a = ko.observable(1);
var b = ko.computed(function() { 
    return a(); 
});
var c = ko.computed(function() { 
    a(b()); // 危险！
});

// 解决方案：使用 deferEvaluation
var c = ko.computed(function() {
    return a(b());
}, null, { deferEvaluation: true });
```

## 下一步

- 学习 [计算属性](computed.md) 的使用
- 了解 [数组操作](arrays.md) 的方法
- 探索 [自定义绑定](../advanced/custom-bindings.md) 的开发 