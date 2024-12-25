# 组件开发

本文介绍如何在 Knockout.js 中开发和使用组件。组件是一种可重用的 UI 单元，它包含自己的视图模型和模板。

## 基础知识

### 注册组件

```javascript
ko.components.register('user-profile', {
    viewModel: function(params) {
        this.name = params.name;
        this.email = params.email;
    },
    template: 
        '<div class="user-profile">\
            <h3 data-bind="text: name"></h3>\
            <p data-bind="text: email"></p>\
        </div>'
});
```

### 使用组件

```html
<div data-bind="component: {
    name: 'user-profile',
    params: {
        name: userName,
        email: userEmail
    }
}"></div>

<!-- 或者使用简写形式 -->
<user-profile params="name: userName, email: userEmail"></user-profile>
```

## 高级特性

### 1. 异步加载

```javascript
// 注册异步组件
ko.components.register('lazy-component', {
    viewModel: { require: 'path/to/viewModel' },
    template: { require: 'text!path/to/template.html' }
});

// AMD 模块形式的视图模型
define(['knockout'], function(ko) {
    function ViewModel(params) {
        // 组件逻辑
    }
    return { viewModel: ViewModel };
});
```

### 2. 生命周期

```javascript
function ComponentViewModel(params) {
    // 构造函数 - 组件初始化
    this.data = params.data;
    
    // 清理函数
    this.dispose = function() {
        // 清理资源
    };
}

ko.components.register('lifecycle-demo', {
    viewModel: {
        createViewModel: function(params, componentInfo) {
            // componentInfo.element - 组件的 DOM 元素
            var vm = new ComponentViewModel(params);
            
            // 组件移除时调用清理函数
            ko.utils.domNodeDisposal.addDisposeCallback(
                componentInfo.element,
                function() { vm.dispose(); }
            );
            
            return vm;
        }
    },
    template: '...'
});
```

## 实用示例

### 1. 表单组件

```javascript
// 注册表单组件
ko.components.register('form-field', {
    viewModel: function(params) {
        this.label = params.label || '';
        this.value = params.value;
        this.type = params.type || 'text';
        this.error = params.error;
        
        this.hasError = ko.computed(function() {
            return this.error && this.error();
        }, this);
    },
    template:
        '<div class="form-group" data-bind="css: { \'has-error\': hasError }">\
            <label data-bind="text: label"></label>\
            <input data-bind="value: value, attr: { type: type }" class="form-control">\
            <span class="error-message" data-bind="text: error, visible: hasError"></span>\
        </div>'
});

// 使用表单组件
<form-field params="
    label: '用户名',
    value: username,
    error: usernameError
"></form-field>
```

### 2. 列表组件

```javascript
// 注册列表组件
ko.components.register('item-list', {
    viewModel: function(params) {
        this.items = params.items;
        this.onSelect = params.onSelect;
        
        this.selectedItem = ko.observable();
        
        this.select = function(item) {
            this.selectedItem(item);
            if (this.onSelect) {
                this.onSelect(item);
            }
        }.bind(this);
    },
    template:
        '<ul class="item-list" data-bind="foreach: items">\
            <li data-bind="css: { selected: $parent.selectedItem() === $data }">\
                <span data-bind="text: name"></span>\
                <button data-bind="click: $parent.select">选择</button>\
            </li>\
        </ul>'
});

// 使用列表组件
<item-list params="
    items: availableItems,
    onSelect: handleSelection
"></item-list>
```

### 3. 模态框组件

```javascript
// 注册模态框组件
ko.components.register('modal-dialog', {
    viewModel: function(params) {
        this.isVisible = params.isVisible;
        this.title = params.title;
        this.content = params.content;
        this.onClose = params.onClose;
        
        this.close = function() {
            this.isVisible(false);
            if (this.onClose) {
                this.onClose();
            }
        }.bind(this);
    },
    template:
        '<div class="modal" data-bind="visible: isVisible">\
            <div class="modal-dialog">\
                <div class="modal-content">\
                    <div class="modal-header">\
                        <h4 data-bind="text: title"></h4>\
                        <button type="button" data-bind="click: close">&times;</button>\
                    </div>\
                    <div class="modal-body" data-bind="template: { nodes: content }"></div>\
                </div>\
            </div>\
        </div>'
});

// 使用模态框组件
<modal-dialog params="
    isVisible: showModal,
    title: modalTitle,
    content: modalContent,
    onClose: handleClose
">
    <!-- 模态框内容 -->
    <div>这是模态框的内容</div>
</modal-dialog>
```

## 最佳实践

### 1. 组件通信

```javascript
// 使用事件总线
var EventBus = {
    subscribers: {},
    
    subscribe: function(event, callback) {
        if (!this.subscribers[event]) {
            this.subscribers[event] = [];
        }
        this.subscribers[event].push(callback);
    },
    
    publish: function(event, data) {
        if (this.subscribers[event]) {
            this.subscribers[event].forEach(function(callback) {
                callback(data);
            });
        }
    }
};

// 在组件中使用
function ComponentA() {
    EventBus.publish('eventA', { data: 'some data' });
}

function ComponentB() {
    EventBus.subscribe('eventA', function(data) {
        // 处理数据
    });
}
```

### 2. 组件复用

```javascript
// 创建基础组件
ko.components.register('base-list', {
    viewModel: function(params) {
        this.items = params.items;
        this.itemTemplate = params.itemTemplate;
    },
    template:
        '<ul data-bind="foreach: items">\
            <!-- ko template: { nodes: $parent.itemTemplate, data: $data } --><!-- /ko -->\
        </ul>'
});

// 扩展基础组件
<base-list params="items: users">
    <template>
        <li>
            <span data-bind="text: name"></span>
            <span data-bind="text: email"></span>
        </li>
    </template>
</base-list>
```

### 3. 性能优化

```javascript
// 使用 pure computed
function OptimizedViewModel(params) {
    this.items = params.items;
    
    this.processedItems = ko.pureComputed(function() {
        return this.items().map(function(item) {
            return new ItemViewModel(item);
        });
    }, this);
}

// 使用虚拟滚动
ko.components.register('virtual-list', {
    viewModel: function(params) {
        this.allItems = params.items;
        this.visibleItems = ko.computed(function() {
            var scrollTop = window.pageYOffset;
            var viewportHeight = window.innerHeight;
            // 计算可见项
            return this.getVisibleItems(scrollTop, viewportHeight);
        }, this);
    }
});
```

## 常见问题

### 1. 组件加载失败

```javascript
// 注册加载器
ko.components.loaders.unshift({
    loadComponent: function(name, componentConfig, callback) {
        // 自定义加载逻辑
        try {
            // 加载组件
            callback(null, componentConfig);
        } catch (error) {
            // 处理错误
            console.error('组件加载失败：', error);
            callback(error);
        }
    }
});
```

### 2. 组件通信问题

```javascript
// 使用回调函数
<parent-component>
    <child-component params="
        onEvent: handleChildEvent,
        data: parentData
    "></child-component>
</parent-component>

// 使用观察者模式
function ParentViewModel() {
    this.childEvents = new ko.subscribable();
    
    this.childEvents.subscribe(function(data) {
        // 处理子组件事件
    });
}
```

### 3. 内存泄漏

```javascript
function ComponentViewModel(params) {
    var subscriptions = [];
    
    // 添加订阅
    subscriptions.push(
        params.data.subscribe(function(newValue) {
            // 处理数据变化
        })
    );
    
    // 清理函数
    this.dispose = function() {
        // 清理所有订阅
        subscriptions.forEach(function(subscription) {
            subscription.dispose();
        });
    };
}
```

## 下一步

- 学习 [性能优化](performance.md)
- 了解 [自定义绑定](custom-bindings.md)
- 探索 [高级特性](../core/computed.md) 