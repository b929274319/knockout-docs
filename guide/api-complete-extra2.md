# Knockout.js API 补充文档 2

本文档补充了更多常用的功能和 API。

## 1. 高级事件处理

```javascript
// 事件管理器
ko.events = {
    handlers: new Map(),
    
    // 注册事件
    on: function(event, handler) {
        if (!this.handlers.has(event)) {
            this.handlers.set(event, new Set());
        }
        this.handlers.get(event).add(handler);
        return () => this.off(event, handler);
    },
    
    // 移除事件
    off: function(event, handler) {
        if (this.handlers.has(event)) {
            if (handler) {
                this.handlers.get(event).delete(handler);
            } else {
                this.handlers.delete(event);
            }
        }
    },
    
    // 触发事件
    emit: function(event, data) {
        if (this.handlers.has(event)) {
            this.handlers.get(event).forEach(handler => {
                try {
                    handler(data);
                } catch (error) {
                    console.error(`Error in event handler for ${event}:`, error);
                }
            });
        }
    },
    
    // 一次性事件
    once: function(event, handler) {
        const wrapper = (data) => {
            handler(data);
            this.off(event, wrapper);
        };
        return this.on(event, wrapper);
    }
};

// 事件绑定处理器
ko.bindingHandlers.events = {
    init: function(element, valueAccessor) {
        const events = ko.unwrap(valueAccessor());
        const disposables = [];
        
        Object.keys(events).forEach(event => {
            const handler = events[event];
            disposables.push(ko.events.on(event, handler));
        });
        
        ko.utils.domNodeDisposal.addDisposeCallback(element, () => {
            disposables.forEach(dispose => dispose());
        });
    }
};
```

## 2. 高级模板系统

```javascript
// 模板管理器
ko.templates = {
    cache: new Map(),
    
    // 注册模板
    register: function(name, template) {
        this.cache.set(name, typeof template === 'function' ? template : () => template);
    },
    
    // 渲染模板
    render: function(name, data) {
        const template = this.cache.get(name);
        if (!template) {
            throw new Error(`Template "${name}" not found`);
        }
        return template(data);
    },
    
    // 动态模板
    dynamic: function(name, dataAccessor) {
        return ko.computed(() => {
            const data = ko.unwrap(dataAccessor());
            return this.render(name, data);
        });
    }
};

// 高级模板绑定
ko.bindingHandlers.template2 = {
    init: function(element, valueAccessor) {
        const options = valueAccessor();
        const name = ko.unwrap(options.name);
        const data = options.data;
        
        ko.computed(() => {
            element.innerHTML = ko.templates.render(name, ko.unwrap(data));
        });
    }
};
```

## 3. 数据验证增强

```javascript
// 验证规则扩展
ko.validation.rules = {
    ...ko.validation.rules,
    
    // 自定义格式
    pattern: function(value, pattern) {
        return new RegExp(pattern).test(value);
    },
    
    // 范围验证
    range: function(value, [min, max]) {
        const num = parseFloat(value);
        return num >= min && num <= max;
    },
    
    // 长度验证
    length: function(value, [min, max]) {
        return value.length >= min && value.length <= max;
    },
    
    // 自定义验证
    custom: function(value, validator) {
        return validator(value);
    },
    
    // 异步验证
    async: function(value, validator) {
        return new Promise((resolve) => {
            validator(value, resolve);
        });
    }
};

// 验证消息模板
ko.validation.messages = {
    required: '此字段是必填的',
    email: '请输入有效的电子邮件地址',
    pattern: '输入格式不正确',
    range: '请输入 {0} 到 {1} 之间的值',
    length: '长度必须在 {0} 到 {1} 之间',
    custom: '验证失败',
    async: '验证中...'
};

// 验证组扩展
ko.validationGroup = function(validatables) {
    return {
        errors: ko.computed(() => {
            const errors = [];
            validatables.forEach(validatable => {
                if (validatable.errors && validatable.errors().length) {
                    errors.push(...validatable.errors());
                }
            });
            return errors;
        }),
        
        isValid: ko.computed(() => {
            return validatables.every(validatable => {
                return !validatable.errors || !validatable.errors().length;
            });
        }),
        
        validate: function() {
            validatables.forEach(validatable => {
                if (validatable.validate) {
                    validatable.validate();
                }
            });
            return this.isValid();
        }
    };
};
```

## 4. 状态管理增强

```javascript
// 状态管理器增强
ko.state = {
    store: {},
    subscribers: new Set(),
    
    // 初始化状态
    init: function(initialState) {
        this.store = ko.observable(initialState);
    },
    
    // 获取状态
    get: function(path) {
        return ko.computed(() => {
            const state = this.store();
            return path.split('.').reduce((obj, key) => obj && obj[key], state);
        });
    },
    
    // 设置状态
    set: function(path, value) {
        const state = {...this.store()};
        const keys = path.split('.');
        const lastKey = keys.pop();
        const target = keys.reduce((obj, key) => obj[key] = obj[key] || {}, state);
        target[lastKey] = ko.unwrap(value);
        this.store(state);
    },
    
    // 监听变化
    watch: function(path, callback) {
        const computed = this.get(path);
        return computed.subscribe(callback);
    },
    
    // 批量更新
    batch: function(updates) {
        const state = {...this.store()};
        updates.forEach(([path, value]) => {
            const keys = path.split('.');
            const lastKey = keys.pop();
            const target = keys.reduce((obj, key) => obj[key] = obj[key] || {}, state);
            target[lastKey] = ko.unwrap(value);
        });
        this.store(state);
    }
};

// 状态绑定处理器
ko.bindingHandlers.state = {
    init: function(element, valueAccessor) {
        const options = valueAccessor();
        const path = options.path;
        const callback = options.callback;
        
        const subscription = ko.state.watch(path, callback);
        ko.utils.domNodeDisposal.addDisposeCallback(element, () => {
            subscription.dispose();
        });
    }
};
```

## 5. 组件生命周期增强

```javascript
// 组件生命周期钩子
ko.components.addLifecycleHooks = function(name, hooks) {
    const existing = ko.components.getConfig(name);
    if (!existing) return;
    
    const originalCreateViewModel = existing.viewModel.createViewModel;
    
    existing.viewModel.createViewModel = function(params, componentInfo) {
        const vm = originalCreateViewModel(params, componentInfo);
        
        // 添加生命周期钩子
        if (hooks.beforeMount) {
            hooks.beforeMount.call(vm, params);
        }
        
        // 挂载后
        ko.tasks.schedule(() => {
            if (hooks.mounted) {
                hooks.mounted.call(vm, params);
            }
        });
        
        // 更新前
        if (hooks.beforeUpdate) {
            Object.keys(params).forEach(key => {
                if (ko.isObservable(params[key])) {
                    params[key].subscribe(() => {
                        hooks.beforeUpdate.call(vm, params);
                    });
                }
            });
        }
        
        // 销毁前
        if (hooks.beforeDestroy) {
            ko.utils.domNodeDisposal.addDisposeCallback(componentInfo.element, () => {
                hooks.beforeDestroy.call(vm, params);
            });
        }
        
        return vm;
    };
};

// 组件通信增强
ko.components.communication = {
    events: new Map(),
    
    // 发送消息
    emit: function(componentName, eventName, data) {
        const key = `${componentName}:${eventName}`;
        if (this.events.has(key)) {
            this.events.get(key).forEach(handler => handler(data));
        }
    },
    
    // 监听消息
    on: function(componentName, eventName, handler) {
        const key = `${componentName}:${eventName}`;
        if (!this.events.has(key)) {
            this.events.set(key, new Set());
        }
        this.events.get(key).add(handler);
        
        return () => {
            this.events.get(key).delete(handler);
        };
    }
};
```

## 6. 数据加载和缓存

```javascript
// 数据加载管理器
ko.dataLoader = {
    cache: new Map(),
    loading: ko.observable(false),
    error: ko.observable(null),
    
    // 加载数据
    load: async function(key, loader, options = {}) {
        const {
            cache = true,
            ttl = 5 * 60 * 1000, // 5分钟缓存
            transform = data => data
        } = options;
        
        // 检查缓存
        if (cache && this.cache.has(key)) {
            const cached = this.cache.get(key);
            if (Date.now() - cached.timestamp < ttl) {
                return cached.data;
            }
        }
        
        try {
            this.loading(true);
            this.error(null);
            
            const data = await loader();
            const transformed = transform(data);
            
            if (cache) {
                this.cache.set(key, {
                    data: transformed,
                    timestamp: Date.now()
                });
            }
            
            return transformed;
        } catch (error) {
            this.error(error);
            throw error;
        } finally {
            this.loading(false);
        }
    },
    
    // 清除缓存
    clearCache: function(key) {
        if (key) {
            this.cache.delete(key);
        } else {
            this.cache.clear();
        }
    }
};

// 数据加载绑定处理器
ko.bindingHandlers.loader = {
    init: function(element, valueAccessor) {
        const options = valueAccessor();
        const {
            key,
            loader,
            success,
            error,
            loadingTemplate,
            errorTemplate
        } = options;
        
        const updateView = async () => {
            try {
                if (loadingTemplate) {
                    element.innerHTML = loadingTemplate;
                }
                
                const data = await ko.dataLoader.load(key, loader);
                element.innerHTML = success(data);
            } catch (err) {
                if (errorTemplate) {
                    element.innerHTML = errorTemplate(err);
                }
                if (error) {
                    error(err);
                }
            }
        };
        
        updateView();
    }
};
```

## 7. 性能优化工具

```javascript
// 性能监控扩展
ko.performance = {
    ...ko.performance,
    
    // 性能标记
    marks: new Map(),
    
    // 开始测量
    mark: function(name) {
        this.marks.set(name, performance.now());
    },
    
    // 结束测量
    measure: function(name, startMark) {
        const start = this.marks.get(startMark);
        const end = performance.now();
        const duration = end - start;
        
        this.marks.delete(startMark);
        return {
            name,
            duration,
            start,
            end
        };
    },
    
    // 性能报告
    report: function() {
        return {
            metrics: this.getMetrics(),
            marks: Array.from(this.marks.entries()),
            memory: window.performance.memory
        };
    }
};

// 性能优化绑定
ko.bindingHandlers.optimize = {
    init: function(element, valueAccessor) {
        const options = valueAccessor();
        const {
            debounce = 100,
            throttle = false,
            lazyUpdate = false
        } = options;
        
        // 防抖更新
        if (debounce) {
            let timeout;
            ko.computed(() => {
                clearTimeout(timeout);
                timeout = setTimeout(() => {
                    ko.tasks.schedule(() => {
                        ko.applyBindingsToNode(element, options.bindings);
                    });
                }, debounce);
            });
        }
        
        // 节流更新
        if (throttle) {
            let last = 0;
            ko.computed(() => {
                const now = Date.now();
                if (now - last >= throttle) {
                    last = now;
                    ko.tasks.schedule(() => {
                        ko.applyBindingsToNode(element, options.bindings);
                    });
                }
            });
        }
        
        // 延迟更新
        if (lazyUpdate) {
            ko.tasks.schedule(() => {
                ko.applyBindingsToNode(element, options.bindings);
            });
        }
    }
};
```

## 注意事项

1. **功能使用建议**
   - 根据实际需求选择合适的功能
   - 避免过度使用高级特性
   - 注意性能影响

2. **开发建议**
   - 保持代码简洁
   - 做好错误处理
   - 注意内存管理

3. **性能优化**
   - 合理使用缓存
   - 避免非必要的更新
   - 使用性能监控工具

4. **最佳实践**
   - 组件化开发
   - 状态管理集中化
   - 数据验证规范化 