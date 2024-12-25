# 自定义绑定

本文介绍如何在 Knockout.js 中创建和使用自定义绑定。通过自定义绑定，你可以扩展 Knockout.js 的功能，实现特定的交互和展示效果。

## 基础知识

### 创建自定义绑定

使用 `ko.bindingHandlers` 注册新的绑定。

```javascript
ko.bindingHandlers.customBinding = {
    init: function(element, valueAccessor, allBindings, viewModel, bindingContext) {
        // 初始化代码
    },
    update: function(element, valueAccessor, allBindings, viewModel, bindingContext) {
        // 更新代码
    }
};
```

### 基本示例

```html
<!-- 使用自定义绑定 -->
<div data-bind="tooltip: tooltipText"></div>

<script>
// 定义工具提示绑定
ko.bindingHandlers.tooltip = {
    init: function(element, valueAccessor) {
        var tooltipText = ko.unwrap(valueAccessor());
        $(element).tooltip({
            title: tooltipText
        });
    },
    update: function(element, valueAccessor) {
        var tooltipText = ko.unwrap(valueAccessor());
        $(element).tooltip('dispose').tooltip({
            title: tooltipText
        });
    }
};

// 使用绑定
var viewModel = {
    tooltipText: ko.observable("这是一个提示")
};
</script>
```

## 高级特性

### 1. 控制更新

```javascript
// 自定义更新逻辑
ko.bindingHandlers.customUpdate = {
    update: function(element, valueAccessor) {
        var value = ko.unwrap(valueAccessor());
        
        // 只在值变化时更新
        if (element.lastValue !== value) {
            element.lastValue = value;
            // 执行更新操作
        }
    }
};
```

### 2. 访问其他绑定

```javascript
ko.bindingHandlers.compound = {
    init: function(element, valueAccessor, allBindings) {
        // 获取其他绑定的值
        var text = allBindings.get('text');
        var visible = allBindings.get('visible');
        
        // 使用这些值
    }
};
```

### 3. 使用绑定上下文

```javascript
ko.bindingHandlers.withContext = {
    init: function(element, valueAccessor, allBindings, viewModel, bindingContext) {
        // 访问父级上下文
        var parentData = bindingContext.$parent;
        
        // 创建子上下文
        var childContext = bindingContext.createChildContext(
            valueAccessor(),
            null,
            function(context) {
                context.$custom = "自定义数据";
            }
        );
        
        // 应用子上下文
        ko.applyBindingsToDescendants(childContext, element);
        
        return { controlsDescendantBindings: true };
    }
};
```

## 实用示例

### 1. 拖拽绑定

```javascript
ko.bindingHandlers.draggable = {
    init: function(element, valueAccessor) {
        var options = ko.unwrap(valueAccessor()) || {};
        
        $(element).draggable({
            handle: options.handle,
            axis: options.axis,
            containment: options.containment,
            start: function(event, ui) {
                if (options.onStart) {
                    options.onStart(event, ui);
                }
            },
            drag: function(event, ui) {
                if (options.onDrag) {
                    options.onDrag(event, ui);
                }
            },
            stop: function(event, ui) {
                if (options.onStop) {
                    options.onStop(event, ui);
                }
            }
        });
    }
};

// 使用示例
<div data-bind="draggable: {
    handle: '.handle',
    axis: 'x',
    onDrag: handleDrag
}">
    <div class="handle">拖动这里</div>
    <div class="content">内容</div>
</div>
```

### 2. 延迟加载图片

```javascript
ko.bindingHandlers.lazyLoad = {
    init: function(element, valueAccessor) {
        var options = ko.unwrap(valueAccessor());
        
        // 创建 IntersectionObserver
        var observer = new IntersectionObserver(function(entries) {
            entries.forEach(function(entry) {
                if (entry.isIntersecting) {
                    // 加载图片
                    element.src = options.src;
                    // 停止观察
                    observer.unobserve(element);
                }
            });
        });
        
        // 开始观察
        observer.observe(element);
    }
};

// 使用示例
<img data-bind="lazyLoad: { src: imageUrl }" />
```

### 3. 富文本编辑器

```javascript
ko.bindingHandlers.richEditor = {
    init: function(element, valueAccessor, allBindings) {
        var options = ko.unwrap(valueAccessor());
        var value = allBindings.get('value');
        
        // 初始化编辑器
        var editor = new RichEditor(element, {
            initialValue: value(),
            onChange: function(newValue) {
                value(newValue);
            }
        });
        
        // 清理函数
        ko.utils.domNodeDisposal.addDisposeCallback(element, function() {
            editor.destroy();
        });
    },
    update: function(element, valueAccessor, allBindings) {
        var value = allBindings.get('value')();
        // 更新编辑器内容
        element.editor.setContent(value);
    }
};

// 使用示例
<div data-bind="richEditor: editorOptions, value: content"></div>
```

## 最佳实践

### 1. 性能优化

```javascript
ko.bindingHandlers.optimized = {
    init: function(element, valueAccessor) {
        // 使用 rateLimit 限制更新频率
        var value = valueAccessor();
        if (ko.isObservable(value)) {
            value.extend({ rateLimit: 500 });
        }
    },
    update: function(element, valueAccessor) {
        // 批量更新 DOM
        ko.utils.setTimeoutOnce(function() {
            var value = ko.unwrap(valueAccessor());
            // 执行更新
        });
    }
};
```

### 2. 错误处理

```javascript
ko.bindingHandlers.safe = {
    init: function(element, valueAccessor) {
        try {
            // 初始化代码
        } catch (error) {
            console.error('绑定初始化错误：', error);
            // 优雅降级处理
        }
    },
    update: function(element, valueAccessor) {
        try {
            // 更新代码
        } catch (error) {
            console.error('绑定更新错误：', error);
            // 恢复到安全状态
        }
    }
};
```

### 3. 可复用性

```javascript
// 创建可配置的绑定工厂
function createBinding(options) {
    return {
        init: function(element, valueAccessor) {
            var value = ko.unwrap(valueAccessor());
            var config = ko.utils.extend({}, options, value);
            // 使用配置初始化
        }
    };
}

// 注册多个相关绑定
ko.bindingHandlers.fadeIn = createBinding({ effect: 'fadeIn' });
ko.bindingHandlers.slideDown = createBinding({ effect: 'slideDown' });
```

## 常见问题

### 1. 内存泄漏

```javascript
ko.bindingHandlers.leakFree = {
    init: function(element, valueAccessor) {
        // 保存事件处理器引用
        var handler = function() {
            // 处理事件
        };
        element.addEventListener('click', handler);
        
        // 清理函数
        ko.utils.domNodeDisposal.addDisposeCallback(element, function() {
            element.removeEventListener('click', handler);
        });
    }
};
```

### 2. 异步操作

```javascript
ko.bindingHandlers.async = {
    init: function(element, valueAccessor) {
        var value = valueAccessor();
        
        // 处理异步操作
        ko.computed(function() {
            var data = ko.unwrap(value);
            
            // 取消之前的请求
            if (element.currentRequest) {
                element.currentRequest.abort();
            }
            
            // 发起新请求
            element.currentRequest = $.ajax({
                // 配置...
            });
        });
    }
};
```

### 3. 与第三方库集成

```javascript
ko.bindingHandlers.thirdParty = {
    init: function(element, valueAccessor) {
        // 确保第三方库已加载
        if (typeof ThirdParty === 'undefined') {
            throw new Error('需要加载第三方库');
        }
        
        // 初始化第三方组件
        var instance = new ThirdParty(element, {
            // 配置...
        });
        
        // 保存实例引用
        ko.utils.domData.set(element, 'thirdPartyInstance', instance);
        
        // 清理
        ko.utils.domNodeDisposal.addDisposeCallback(element, function() {
            var instance = ko.utils.domData.get(element, 'thirdPartyInstance');
            if (instance) {
                instance.destroy();
            }
        });
    }
};
```

## 下一步

- 学习 [组件开发](components.md)
- 了解 [性能优化](performance.md)
- 探索更多 [高级特性](../core/computed.md) 