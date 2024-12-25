# 计算属性（Computed Observables）

计算属性是 Knockout.js 中的一种特殊的可观察对象，它的值是由其他可观察对象计算得出的。本文将详细介绍计算属性的使用方法和高级特性。

## 基础概念

### 什么是计算属性？

计算属性是一个依赖于其他可观察对象的属性，当依赖的可观察对象发生变化时，计算属性会自动重新计算。

```javascript
function ViewModel() {
    var self = this;
    // 基础可观察对象
    self.price = ko.observable(100);
    self.quantity = ko.observable(2);
    
    // 计算属性
    self.total = ko.computed(function() {
        return self.price() * self.quantity();
    });
}
```

### 为什么需要计算属性？

1. **自动计算**
   - 依赖变化时自动更新
   - 避免手动同步数据

2. **性能优化**
   - 只在依赖变化时才重新计算
   - 缓存计算结果

3. **代码组织**
   - 将相关逻辑集中管理
   - 提高代码可维护性

## 基本用法

### 1. 创建计算属性

```javascript
// 基本形式
function ViewModel() {
    var self = this;
    self.firstName = ko.observable("张");
    self.lastName = ko.observable("三");
    
    self.fullName = ko.computed(function() {
        return self.firstName() + " " + self.lastName();
    });
}

// 链式写法
var fullName = ko.computed(function() {
    return firstName() + " " + lastName();
});
```

### 2. 读取计算属性

```javascript
var vm = new ViewModel();
console.log(vm.fullName()); // 输出: "张 三"

// 在绑定中使用
// <span data-bind="text: fullName"></span>
```

### 3. 可写计算属性

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

## 高级特性

### 1. 纯计算属性

```javascript
// 纯计算属性 - 不会有副作用
var self = this;
self.total = ko.pureComputed(function() {
    return self.price() * self.quantity();
});
```

### 2. 延迟计算

```javascript
var self = this;
self.expensiveOperation = ko.computed(function() {
    // 复杂计算
    return heavyCalculation();
}).extend({ deferred: true });
```

### 3. 条件计算

```javascript
var self = this;
self.conditionalValue = ko.computed(function() {
    if (self.shouldCompute()) {
        return expensiveComputation();
    }
    return defaultValue;
});
```

## 最佳实践

### 1. 性能优化

```javascript
// 使用 rate limiting
var self = this;
self.searchResults = ko.computed(function() {
    // 复杂搜索逻辑
    return search(self.searchTerm());
}).extend({ rateLimit: 500 });

// 使用 deferred updates
ko.options.deferUpdates = true;
```

### 2. 依赖管理

```javascript
// 显式声明依赖
var self = this;
self.total = ko.computed(function() {
    ko.dependencyDetection.ignore(function() {
        // 这里的代码不会创建依赖
    });
    return self.price() * self.quantity();
});
```

### 3. 错误处理

```javascript
var self = this;
self.safeComputed = ko.computed(function() {
    try {
        return riskyOperation();
    } catch (error) {
        console.error('计算出错：', error);
        return defaultValue;
    }
});
```

## 常见用例

### 1. 数据转换

```javascript
function ViewModel() {
    var self = this;
    self.price = ko.observable(100);
    
    // 格式化价格
    self.formattedPrice = ko.computed(function() {
        return "￥" + self.price().toFixed(2);
    });
}
```

### 2. 数据过滤

```javascript
function ViewModel() {
    var self = this;
    self.items = ko.observableArray([/* ... */]);
    self.searchTerm = ko.observable("");
    
    self.filteredItems = ko.computed(function() {
        var search = self.searchTerm().toLowerCase();
        return self.items().filter(function(item) {
            return item.name.toLowerCase().indexOf(search) !== -1;
        });
    });
}
```

### 3. 状态管理

```javascript
function FormViewModel() {
    var self = this;
    self.username = ko.observable("");
    self.password = ko.observable("");
    
    self.isValid = ko.computed(function() {
        return self.username().length >= 3 && 
               self.password().length >= 6;
    });
    
    self.canSubmit = ko.computed(function() {
        return self.isValid() && !self.isSubmitting();
    });
}
```

## 常见问题

### 1. 循环依赖

```javascript
// 错误示例
var a = ko.computed(function() { return b(); });
var b = ko.computed(function() { return a(); });

// 解决方案
var a = ko.computed(function() {
    return b();
}).extend({ deferred: true });
```

### 2. 性能问题

```javascript
// 不好的做法 - 每次都重新创建数组
var self = this;
self.processedItems = ko.computed(function() {
    return self.items().map(function(item) {
        return new ItemViewModel(item);
    });
});

// 好的做法 - 缓存处理结果
var self = this;
self.processedItems = ko.computed(function() {
    if (!self._cache) {
        self._cache = {};
    }
    return self.items().map(function(item) {
        if (!self._cache[item.id]) {
            self._cache[item.id] = new ItemViewModel(item);
        }
        return self._cache[item.id];
    });
});
```

### 3. 内存泄漏

```javascript
// 不好的做法
function ViewModel() {
    this.computed = ko.computed(function() {
        // 这里引用了外部资源
    });
}

// 好的做法
function ViewModel() {
    var self = this;
    this.computed = ko.computed(function() {
        // 使用
    }).extend({ dispose: function() {
        // 清理资源
    }});
    
    this.dispose = function() {
        self.computed.dispose();
    };
}
```

## 下一步

- 学习 [可观察数组](arrays.md) 的使用
- 了解 [自定义绑定](../advanced/custom-bindings.md) 的开发
- 探索 [组件](../advanced/components.md) 的创建 