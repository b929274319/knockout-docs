# 性能优化

本文介绍在使用 Knockout.js 时的性能优化技巧。通过这些优化，你可以提高应用程序的响应速度和运行效率。

## 基本原则

### 1. 减少依赖追踪

```javascript
// 不好的做法 - 每次访问都会触发依赖追踪
<div data-bind="text: firstName() + ' ' + lastName()"></div>

// 好的做法 - 使用计算属性
function ViewModel() {
    this.firstName = ko.observable("张");
    this.lastName = ko.observable("三");
    
    this.fullName = ko.computed(function() {
        return this.firstName() + ' ' + this.lastName();
    }, this);
}
<div data-bind="text: fullName"></div>
```

### 2. 使用纯计算属性

```javascript
// 标准计算属性
this.total = ko.computed(function() {
    return this.price() * this.quantity();
});

// 纯计算属性 - 更高效
this.total = ko.pureComputed(function() {
    return this.price() * this.quantity();
});
```

### 3. 延迟更新

```javascript
// 使用 rateLimit
this.searchResults = ko.computed(function() {
    // 复杂搜索逻辑
}).extend({ rateLimit: 500 });

// 使用 throttle
this.windowSize = ko.observable().extend({ throttle: 100 });

// 使用 deferred
this.heavyComputation = ko.computed(function() {
    // 耗时计算
}).extend({ deferred: true });
```

## 列表优化

### 1. 虚拟滚动

```javascript
// 虚拟滚动组件
ko.components.register('virtual-scroll', {
    viewModel: function(params) {
        var self = this;
        this.allItems = params.items;
        this.itemHeight = params.itemHeight || 30;
        this.windowHeight = ko.observable(window.innerHeight);
        
        this.visibleItems = ko.computed(function() {
            var scrollTop = window.pageYOffset;
            var windowHeight = self.windowHeight();
            var startIndex = Math.floor(scrollTop / self.itemHeight);
            var endIndex = Math.ceil((scrollTop + windowHeight) / self.itemHeight);
            
            return self.allItems().slice(startIndex, endIndex);
        });
        
        // 监听滚动
        window.addEventListener('scroll', function() {
            self.visibleItems.valueHasMutated();
        });
    },
    template:
        '<div class="virtual-scroll" data-bind="foreach: visibleItems">\
            <div class="item" data-bind="text: name"></div>\
        </div>'
});

// 使用虚拟滚动
<virtual-scroll params="
    items: largeDataset,
    itemHeight: 30
"></virtual-scroll>
```

### 2. 批量更新

```javascript
// 不好的做法 - 频繁更新
items().forEach(function(item) {
    item.selected(true);
});

// 好的做法 - 批量更新
ko.utils.arrayForEach(items(), function(item) {
    item.selected(true);
});
items.valueHasMutated();
```

### 3. 使用 as 绑定

```html
<!-- 标准绑定 -->
<ul data-bind="foreach: items">
    <li data-bind="text: $data.name"></li>
</ul>

<!-- 使用 as 绑定 - 更高效 -->
<ul data-bind="foreach: { data: items, as: 'item' }">
    <li data-bind="text: item.name"></li>
</ul>
```

## DOM 优化

### 1. 避免不必要的绑定

```html
<!-- 不好的做法 - 过度绑定 -->
<div data-bind="foreach: items">
    <div data-bind="text: name, attr: { title: name }, css: { selected: isSelected }"></div>
</div>

<!-- 好的做法 - 合并绑定 -->
<div data-bind="foreach: items">
    <div data-bind="
        text: name,
        attr: { title: name },
        css: { selected: isSelected }
    "></div>
</div>
```

### 2. 使用容器绑定

```html
<!-- 不好的做法 - 每个项都有绑定 -->
<div data-bind="foreach: items">
    <div data-bind="visible: isVisible">
        <span data-bind="text: name"></span>
    </div>
</div>

<!-- 好的做法 - 使用容器绑定 -->
<div data-bind="with: visibleItems">
    <div data-bind="foreach: $data">
        <span data-bind="text: name"></span>
    </div>
</div>
```

### 3. 使用模板缓存

```javascript
// 注册模板
ko.templates["item-template"] =
    '<div class="item">\
        <span data-bind="text: name"></span>\
        <span data-bind="text: description"></span>\
    </div>';

// 使用模板
<div data-bind="template: { name: 'item-template', foreach: items }"></div>
```

## 内存优化

### 1. 清理订阅

```javascript
function ViewModel() {
    var subscriptions = [];
    
    // 添加订阅
    subscriptions.push(
        someObservable.subscribe(function(newValue) {
            // 处理变化
        })
    );
    
    // 清理函数
    this.dispose = function() {
        subscriptions.forEach(function(subscription) {
            subscription.dispose();
        });
    };
}
```

### 2. 避免闭包陷阱

```javascript
// 不好的做法 - 创建多个闭包
<div data-bind="foreach: items">
    <button data-bind="click: function() { $parent.remove($data); }">删除</button>
</div>

// 好的做法 - 使用共享函数
function ViewModel() {
    this.removeItem = function(item) {
        this.items.remove(item);
    }.bind(this);
}
<div data-bind="foreach: items">
    <button data-bind="click: $parent.removeItem">删除</button>
</div>
```

### 3. 及时释放资源

```javascript
// 组件清理
ko.components.register('cleanup-demo', {
    viewModel: {
        createViewModel: function(params, componentInfo) {
            var vm = new ViewModel(params);
            
            ko.utils.domNodeDisposal.addDisposeCallback(
                componentInfo.element,
                function() {
                    vm.dispose();
                    // 清理其他资源
                    delete vm.heavyData;
                    vm = null;
                }
            );
            
            return vm;
        }
    }
});
```

## 调试和监控

### 1. 性能分析

```javascript
// 添加性能标记
console.time('更新操作');
// 执行操作
console.timeEnd('更新操作');

// 监控依赖数量
function countDependencies(observable) {
    return observable._subscriptions ? 
        observable._subscriptions.length : 0;
}
```

### 2. 内存泄漏检测

```javascript
// 使用 Chrome 开发工具
// 1. 记录内存快照
// 2. 执行操作
// 3. 记录新快照
// 4. 比较快照找出泄漏

// 添加调试信息
ko.computed(function() {
    var value = this.someObservable();
    console.log('计算属性更新:', value);
    return value;
}, this).extend({ deferred: true });
```

### 3. 错误追踪

```javascript
// 全局错误处理
ko.options.onError = function(error) {
    console.error('Knockout 错误:', error);
    // 发送错误报告
};

// 绑定错误处理
<div data-bind="text: someValue, onError: handleError"></div>
```

## 最佳实践总结

1. **使用适当的计算属性类型**
   - 简单计算用 `pureComputed`
   - 有副作用的计算用 `computed`

2. **优化大型列表**
   - 使用虚拟滚动
   - 实现分页
   - 延迟加载

3. **减少 DOM 操作**
   - 使用模板缓存
   - 批量更新
   - 避免不必要的绑定

4. **内存管理**
   - 及时清理订阅
   - 避免闭包泄漏
   - 释放大型数据结构

5. **监控和调试**
   - 使用性能分析工具
   - 监控内存使用
   - 添加适当的日志

## 下一步

- 了解更多 [高级特性](../core/computed.md)
- 学习 [组件开发](components.md)
- 探索 [自定义绑定](custom-bindings.md) 