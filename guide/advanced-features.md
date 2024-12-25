# Knockout.js 高级特性

本文档补充了一些 Knockout.js 的高级特性和不常见但非常有用的功能。

## 高级绑定特性

### 1. 自定义绑定值解析器

```javascript
// 自定义绑定值解析器
ko.bindingProvider.instance.preprocessBindingValue = function(bindingString) {
    return bindingString.replace(/\$data\./g, 'ko.dataFor(element).');
};
```

### 2. 绑定上下文扩展

```javascript
// 扩展绑定上下文
ko.bindingContext.prototype.createChildContext = function(dataItem, bindingContextMethods) {
    var context = ko.bindingContext.prototype.createChildContext.call(this, dataItem);
    for (var key in bindingContextMethods) {
        context[key] = bindingContextMethods[key];
    }
    return context;
};
```

### 3. 动态绑定生成

```javascript
function generateBindings(data) {
    return ko.computed(function() {
        var bindings = {};
        Object.keys(data).forEach(function(key) {
            bindings[key] = ko.unwrap(data[key]);
        });
        return bindings;
    });
}
```

## 高级组件特性

### 1. 组件生命周期钩子

```javascript
ko.components.register('advanced-component', {
    viewModel: {
        createViewModel: function(params, componentInfo) {
            var vm = {
                // 初始化钩子
                onInit: function() {
                    console.log('组件初始化');
                },
                
                // 更新钩子
                onUpdate: function() {
                    console.log('组件更新');
                },
                
                // 销毁钩子
                onDispose: function() {
                    console.log('组件销毁');
                }
            };
            
            // 注册生命周期事件
            ko.utils.domNodeDisposal.addDisposeCallback(
                componentInfo.element,
                vm.onDispose
            );
            
            return vm;
        }
    }
});
```

### 2. 组件通信增强

```javascript
// 组件事件总线
var ComponentEventBus = {
    handlers: {},
    
    on: function(event, handler) {
        if (!this.handlers[event]) {
            this.handlers[event] = [];
        }
        this.handlers[event].push(handler);
    },
    
    off: function(event, handler) {
        if (this.handlers[event]) {
            this.handlers[event] = this.handlers[event].filter(
                function(h) { return h !== handler; }
            );
        }
    },
    
    emit: function(event, data) {
        if (this.handlers[event]) {
            this.handlers[event].forEach(function(handler) {
                handler(data);
            });
        }
    }
};
```

### 3. 动态组件加载器

```javascript
function ComponentLoader() {
    this.loadComponent = function(name, callback) {
        require(['components/' + name], function(component) {
            ko.components.register(name, component);
            callback && callback();
        });
    };
    
    this.unloadComponent = function(name) {
        ko.components.unregister(name);
    };
}
```

## 高级数据处理

### 1. 深度观察者

```javascript
function createDeepObservable(initialValue) {
    var observable = ko.observable(initialValue);
    
    // 递归使对象的所有属性可观察
    function makeObservable(obj) {
        if (typeof obj !== 'object' || obj === null) return obj;
        
        Object.keys(obj).forEach(function(key) {
            if (typeof obj[key] === 'object' && obj[key] !== null) {
                obj[key] = makeObservable(obj[key]);
            } else {
                obj[key] = ko.observable(obj[key]);
            }
        });
        
        return obj;
    }
    
    return ko.computed({
        read: function() {
            return makeObservable(ko.unwrap(observable));
        },
        write: function(newValue) {
            observable(makeObservable(newValue));
        }
    });
}
```

### 2. 数据转换管道

```javascript
function createPipeline(transforms) {
    return function(data) {
        return transforms.reduce(function(result, transform) {
            return transform(result);
        }, data);
    };
}

// 使用示例
var dataPipeline = createPipeline([
    function filterData(data) {
        return data.filter(function(item) { return item.active; });
    },
    function sortData(data) {
        return data.sort(function(a, b) { return a.order - b.order; });
    },
    function transformData(data) {
        return data.map(function(item) {
            return { id: item.id, name: item.name.toUpperCase() };
        });
    }
]);
```

### 3. 批量数据处理

```javascript
function BatchProcessor(options) {
    var self = this;
    var queue = ko.observableArray([]);
    var processing = ko.observable(false);
    
    this.add = function(items) {
        queue.push.apply(queue, items);
        if (!processing()) {
            this.process();
        }
    };
    
    this.process = function() {
        if (queue().length === 0) {
            processing(false);
            return;
        }
        
        processing(true);
        var batch = queue.splice(0, options.batchSize || 100);
        
        setTimeout(function() {
            options.processBatch(batch);
            self.process();
        }, options.delay || 0);
    };
}
```

## 性能优化技巧

### 1. 虚拟滚动实现

```javascript
ko.bindingHandlers.virtualScroll = {
    init: function(element, valueAccessor) {
        var options = valueAccessor();
        var viewportHeight = options.height || '400px';
        var itemHeight = options.itemHeight || 40;
        var items = options.items;
        
        element.style.height = viewportHeight;
        element.style.overflowY = 'scroll';
        
        var viewModel = {
            visibleItems: ko.computed(function() {
                var scrollTop = element.scrollTop;
                var startIndex = Math.floor(scrollTop / itemHeight);
                var endIndex = startIndex + Math.ceil(element.clientHeight / itemHeight);
                
                return items().slice(startIndex, endIndex).map(function(item, index) {
                    return {
                        data: item,
                        top: (startIndex + index) * itemHeight + 'px'
                    };
                });
            })
        };
        
        element.addEventListener('scroll', function() {
            viewModel.visibleItems.notifySubscribers();
        });
        
        ko.applyBindingsToDescendants(viewModel, element);
        return { controlsDescendantBindings: true };
    }
};
```

### 2. 内存泄漏检测

```javascript
function MemoryLeakDetector() {
    var subscriptions = new WeakMap();
    
    this.track = function(observable, owner) {
        if (!subscriptions.has(owner)) {
            subscriptions.set(owner, []);
        }
        
        var subscription = observable.subscribe(function() {});
        subscriptions.get(owner).push(subscription);
    };
    
    this.check = function(owner) {
        var ownerSubs = subscriptions.get(owner) || [];
        return ownerSubs.some(function(sub) {
            return !sub.isDisposed;
        });
    };
    
    this.cleanup = function(owner) {
        var ownerSubs = subscriptions.get(owner) || [];
        ownerSubs.forEach(function(sub) {
            sub.dispose();
        });
        subscriptions.delete(owner);
    };
}
```

### 3. 计算属性优化

```javascript
function optimizedComputed(evaluator, options) {
    options = options || {};
    var timeoutHandle;
    var lastValue;
    
    return ko.computed(function() {
        var currentValue = evaluator();
        
        if (options.throttle) {
            clearTimeout(timeoutHandle);
            timeoutHandle = setTimeout(function() {
                lastValue = currentValue;
            }, options.throttle);
            return lastValue || currentValue;
        }
        
        return currentValue;
    }).extend({ deferred: true });
}
```

## 调试工具

### 1. 依赖图生成器

```javascript
function DependencyGraphGenerator() {
    var graph = {};
    
    this.track = function(observable, name) {
        if (!graph[name]) {
            graph[name] = {
                dependencies: [],
                subscribers: []
            };
        }
        
        ko.computed(function() {
            observable();
            ko.computedContext.getDependenciesCount();
        }).subscribe(function() {
            // 记录依赖关系
        });
    };
    
    this.generateGraph = function() {
        return {
            nodes: Object.keys(graph).map(function(name) {
                return { id: name, type: 'observable' };
            }),
            edges: Object.keys(graph).reduce(function(edges, name) {
                return edges.concat(
                    graph[name].dependencies.map(function(dep) {
                        return { from: name, to: dep, type: 'depends-on' };
                    })
                );
            }, [])
        };
    };
}
```

### 2. 性能分析器

```javascript
function PerformanceProfiler() {
    var measurements = {};
    
    this.start = function(name) {
        measurements[name] = {
            startTime: performance.now(),
            updates: 0
        };
    };
    
    this.end = function(name) {
        if (measurements[name]) {
            measurements[name].endTime = performance.now();
            measurements[name].duration = 
                measurements[name].endTime - measurements[name].startTime;
        }
    };
    
    this.trackObservable = function(observable, name) {
        var original = observable.notifySubscribers;
        measurements[name] = { updates: 0, totalTime: 0 };
        
        observable.notifySubscribers = function() {
            var start = performance.now();
            original.apply(this, arguments);
            var end = performance.now();
            
            measurements[name].updates++;
            measurements[name].totalTime += (end - start);
        };
    };
    
    this.getReport = function() {
        return measurements;
    };
}
```

## 最佳实践补充

### 1. 错误处理

```javascript
ko.onError = function(error) {
    console.error('Knockout 错误:', error);
    // 发送错误报告
    reportError(error);
};

function safeComputed(evaluator) {
    return ko.computed(function() {
        try {
            return evaluator();
        } catch (error) {
            console.error('计算属性错误:', error);
            return null;
        }
    });
}
```

### 2. 安全绑定

```javascript
// 防止 XSS 的安全绑定
ko.bindingHandlers.safeHtml = {
    init: function(element, valueAccessor) {
        var sanitizedValue = ko.computed(function() {
            var html = ko.unwrap(valueAccessor());
            return DOMPurify.sanitize(html);
        });
        
        ko.applyBindingsToNode(element, { html: sanitizedValue });
    }
};
```

### 3. 可访问性增强

```javascript
// ARIA 支持
ko.bindingHandlers.ariaBinding = {
    update: function(element, valueAccessor) {
        var value = ko.unwrap(valueAccessor());
        Object.keys(value).forEach(function(key) {
            element.setAttribute('aria-' + key, ko.unwrap(value[key]));
        });
    }
};
```

## 注意事项

1. **性能考虑**
   - 使用 `throttle` 和 `debounce` 控制更新频率
   - 避免不必要的计算属性依赖
   - 及时清理不再需要的订阅

2. **内存管理**
   - 使用 WeakMap 存储临时数据
   - 避免循环引用
   - 定期检查内存泄漏

3. **代码组织**
   - 遵循单一职责原则
   - 使用模块化组织代码
   - 保持组件粒度合适

4. **测试建议**
   - 编写单元测试
   - 使用集成测试验证组件交互
   - 进行性能测试 