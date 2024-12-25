# 常见问题

本文收集了使用 Knockout.js 时的常见问题和解决方案。

## 基础问题

### 1. 为什么我的绑定不生效？

**问题：**
```html
<span data-bind="text: name"></span>
<script>
var viewModel = {
    name: "张三"
};
ko.applyBindings(viewModel);
</script>
```

**解决方案：**
```javascript
// 使用 observable
var viewModel = {
    name: ko.observable("张三")
};
ko.applyBindings(viewModel);
```

**说明：**
- Knockout 需要使用 observable 来追踪变化
- 普通属性不会触发 UI 更新

### 2. 如何获取和设置可观察对象的值？

**问题：**
```javascript
// 错误方式
console.log(viewModel.name);
viewModel.name = "李四";
```

**解决方案：**
```javascript
// 正确方式
console.log(viewModel.name()); // 获取值
viewModel.name("李四");      // 设置值
```

**说明：**
- 可观察对象是函数
- 调用函数获取值
- 传参设置新值

### 3. 为什么数组更新不显示？

**问题：**
```javascript
var viewModel = {
    items: ko.observableArray(["项目1", "项目2"])
};
viewModel.items[0] = "新项目"; // 不会更新 UI
```

**解决方案：**
```javascript
// 使用数组方法
viewModel.items.push("新项目");
viewModel.items.splice(0, 1, "新项目");

// 或者替换整个数组
viewModel.items(["新项目", "项目2"]);
```

## 进阶问题

### 4. 如何处理表单提交？

**问题：**
```html
<form data-bind="submit: handleSubmit">
    <input data-bind="value: username" />
    <button type="submit">提交</button>
</form>
```

**解决方案：**
```javascript
function ViewModel() {
    this.username = ko.observable("");
    
    this.handleSubmit = function(formElement) {
        // 阻止默认提交
        event.preventDefault();
        
        // 获取表单数据
        var data = {
            username: this.username()
        };
        
        // 发送请求
        $.ajax({
            url: '/api/submit',
            method: 'POST',
            data: data
        });
    };
}
```

### 5. 如何实现双向绑定？

**问题：**
```html
<input data-bind="value: name" />
```

**解决方案：**
```html
<!-- 即时更新 -->
<input data-bind="textInput: name" />

<!-- 失去焦点时更新 -->
<input data-bind="value: name" />

<script>
var viewModel = {
    name: ko.observable("")
};

// 监听变化
viewModel.name.subscribe(function(newValue) {
    console.log("名字改变为：" + newValue);
});
</script>
```

### 6. 如何处理异步数据？

**问题：**
```javascript
function ViewModel() {
    this.data = ko.observableArray([]);
    
    $.get('/api/data', function(response) {
        this.data(response); // this 指向错误
    });
}
```

**解决方案：**
```javascript
function ViewModel() {
    var self = this;
    self.data = ko.observableArray([]);
    self.isLoading = ko.observable(true);
    
    $.get('/api/data')
        .done(function(response) {
            self.data(response);
        })
        .fail(function(error) {
            console.error('加载失败：', error);
        })
        .always(function() {
            self.isLoading(false);
        });
}
```

## 性能问题

### 7. 大列表性能优化

**问题：**
```html
<ul data-bind="foreach: items">
    <li>
        <span data-bind="text: name"></span>
        <span data-bind="text: description"></span>
    </li>
</ul>
```

**解决方案：**
```javascript
// 使用虚拟滚动
ko.components.register('virtual-list', {
    viewModel: function(params) {
        this.items = params.items;
        this.visibleItems = ko.computed(function() {
            // 只返回可见区域的项目
            return this.getVisibleItems();
        }, this);
    },
    template: '<div data-bind="foreach: visibleItems">...</div>'
});
```

### 8. 计算属性性能优化

**问题：**
```javascript
// 每次依赖变化都会重新计算
this.total = ko.computed(function() {
    return this.items().reduce(function(sum, item) {
        return sum + item.price();
    }, 0);
});
```

**解决方案：**
```javascript
// 使用 pure computed
this.total = ko.pureComputed(function() {
    return this.items().reduce(function(sum, item) {
        return sum + item.price();
    }, 0);
});

// 使用 rate limit
this.searchResults = ko.computed(function() {
    // 复杂搜索逻辑
}).extend({ rateLimit: 500 });
```

## 组件问题

### 9. 组件通信

**问题：**
如何在组件之间传递数据？

**解决方案：**
```javascript
// 1. 使用参数
<parent-component>
    <child-component params="
        data: parentData,
        onUpdate: handleUpdate
    "></child-component>
</parent-component>

// 2. 使用事件聚合器
var EventBus = {
    subscribers: {},
    subscribe: function(event, callback) {
        // 订阅事件
    },
    publish: function(event, data) {
        // 发布事件
    }
};

// 3. 使用共享服务
function SharedService() {
    this.data = ko.observable();
}
```

### 10. 组件生命周期

**问题：**
如何在组件创建和销毁时执行代码？

**解决方案：**
```javascript
ko.components.register('lifecycle-demo', {
    viewModel: {
        createViewModel: function(params, componentInfo) {
            // 创建视图模型
            var vm = new ViewModel(params);
            
            // 组件创建后
            vm.onCreated = function() {
                console.log('组件创建完成');
            };
            
            // 组件销毁前
            ko.utils.domNodeDisposal.addDisposeCallback(
                componentInfo.element,
                function() {
                    vm.dispose();
                }
            );
            
            return vm;
        }
    }
});
```

## 调试问题

### 11. 如何调试绑定？

**解决方案：**
```javascript
// 1. 使用 ko.dataFor 和 ko.contextFor
var element = document.getElementById('myElement');
console.log(ko.dataFor(element));    // 获取数据上下文
console.log(ko.contextFor(element)); // 获取绑定上下文

// 2. 添加调试信息
ko.computed(function() {
    var value = this.someObservable();
    console.log('计算属性更新:', value);
    return value;
}, this);

// 3. 使用 Chrome 开发工具的 Knockout 上下文调试器
```

### 12. 如何处理绑定错误？

**解决方案：**
```javascript
// 1. 全局错误处理
ko.options.onError = function(error) {
    console.error('Knockout 错误:', error);
};

// 2. 使用 try-catch
ko.computed(function() {
    try {
        // 可能出错的代码
        return this.someComputation();
    } catch (error) {
        console.error('计算出错:', error);
        return defaultValue;
    }
}, this);

// 3. 数据验证
this.isValid = ko.computed(function() {
    var value = this.someValue();
    return value != null && value !== '';
}, this);
```

## 最佳实践

### 13. 如何组织大型应用？

**解决方案：**
```javascript
// 1. 使用模块化
// app/services/userService.js
define(['knockout'], function(ko) {
    function UserService() {
        // 服务实现
    }
    return new UserService();
});

// 2. 使用组件
// app/components/user-list/index.js
define(['knockout', './template.html'], function(ko, template) {
    function ViewModel(params) {
        // 组件逻辑
    }
    return { viewModel: ViewModel, template: template };
});

// 3. 使用路由
ko.components.register('app', {
    viewModel: function() {
        this.route = ko.observable('home');
    },
    template: 
        '<div class="app">\
            <!-- ko component: { name: route } --><!-- /ko -->\
        </div>'
});
```

### 14. 如何处理表单验证？

**解决方案：**
```javascript
// 1. 使用计算属性
function FormViewModel() {
    this.username = ko.observable("");
    this.password = ko.observable("");
    
    this.usernameError = ko.computed(function() {
        var value = this.username();
        if (!value) return "用户名不能为空";
        if (value.length < 3) return "用户名太短";
        return null;
    }, this);
    
    this.isValid = ko.computed(function() {
        return !this.usernameError() && 
               !this.passwordError();
    }, this);
}

// 2. 使用验证插件
ko.validation.init({
    insertMessages: false,
    decorateElement: true,
    errorElementClass: 'has-error'
});

this.username = ko.observable()
    .extend({ required: true, minLength: 3 });
```

### 15. 如何优化性能？

**解决方案：**
```javascript
// 1. 使用纯计算属性
this.total = ko.pureComputed(function() {
    return this.price() * this.quantity();
});

// 2. 批量更新
ko.computed(function() {
    var items = this.items();
    items.forEach(function(item) {
        item.selected(true);
    });
}).extend({ rateLimit: 50 });

// 3. 使用虚拟滚动
// 参见问题 7 的解决方案

// 4. 避免不必要的计算
this.filteredItems = ko.computed(function() {
    var search = this.searchTerm().toLowerCase();
    if (!search) return this.items();
    return this.items().filter(function(item) {
        return item.name().toLowerCase().includes(search);
    });
}).extend({ rateLimit: 200 });
```

## 下一步

- 探索 [高级特性](core/computed.md)
- 学习 [组件开发](advanced/components.md)
- 了解 [性能优化](advanced/performance.md) 