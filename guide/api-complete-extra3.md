# Knockout.js API 补充文档 3

本文档补充了更多实用的功能和工具。

## 1. 动画效果支持

```javascript
// 动画管理器
ko.animation = {
    transitions: new Map(),
    
    // 注册动画
    register: function(name, config) {
        this.transitions.set(name, {
            enter: config.enter || ((el) => Promise.resolve()),
            leave: config.leave || ((el) => Promise.resolve()),
            duration: config.duration || 300
        });
    },
    
    // 执行动画
    animate: async function(element, name, type = 'enter') {
        const transition = this.transitions.get(name);
        if (!transition) return;
        
        element.style.animation = 'none';
        await transition[type](element);
        return new Promise(resolve => setTimeout(resolve, transition.duration));
    }
};

// 预定义动画
ko.animation.register('fade', {
    enter: (el) => {
        el.style.opacity = '0';
        el.style.transition = 'opacity 0.3s ease-in-out';
        requestAnimationFrame(() => el.style.opacity = '1');
    },
    leave: (el) => {
        el.style.opacity = '1';
        el.style.transition = 'opacity 0.3s ease-in-out';
        requestAnimationFrame(() => el.style.opacity = '0');
    },
    duration: 300
});

// 动画绑定处理器
ko.bindingHandlers.animate = {
    init: function(element, valueAccessor) {
        const options = valueAccessor();
        const name = ko.unwrap(options.name);
        const trigger = options.trigger;
        
        if (trigger) {
            ko.computed(() => {
                if (ko.unwrap(trigger)) {
                    ko.animation.animate(element, name, 'enter');
                } else {
                    ko.animation.animate(element, name, 'leave');
                }
            });
        }
    }
};
```

## 2. 高级过滤和排序

```javascript
// 过滤和排序管理器
ko.filtering = {
    filters: new Map(),
    sorters: new Map(),
    
    // 注册过滤器
    registerFilter: function(name, filter) {
        this.filters.set(name, filter);
    },
    
    // 注册排序器
    registerSorter: function(name, sorter) {
        this.sorters.set(name, sorter);
    },
    
    // 应用过滤
    applyFilter: function(array, filterName, ...args) {
        const filter = this.filters.get(filterName);
        return filter ? array.filter(item => filter(item, ...args)) : array;
    },
    
    // 应用排序
    applySort: function(array, sorterName, ...args) {
        const sorter = this.sorters.get(sorterName);
        return sorter ? array.slice().sort((a, b) => sorter(a, b, ...args)) : array;
    },
    
    // 链式处理
    chain: function(array) {
        return {
            data: array,
            filter: function(name, ...args) {
                this.data = ko.filtering.applyFilter(this.data, name, ...args);
                return this;
            },
            sort: function(name, ...args) {
                this.data = ko.filtering.applySort(this.data, name, ...args);
                return this;
            },
            value: function() {
                return this.data;
            }
        };
    }
};

// 预定义过滤器
ko.filtering.registerFilter('search', (item, term, fields) => {
    if (!term) return true;
    term = term.toLowerCase();
    return fields.some(field => {
        const value = ko.unwrap(item[field]);
        return value && value.toString().toLowerCase().includes(term);
    });
});

// 预定义排序器
ko.filtering.registerSorter('field', (a, b, field, direction = 'asc') => {
    const valueA = ko.unwrap(a[field]);
    const valueB = ko.unwrap(b[field]);
    const modifier = direction === 'asc' ? 1 : -1;
    return valueA < valueB ? -modifier : valueA > valueB ? modifier : 0;
});

// 过滤排序绑定处理器
ko.bindingHandlers.filterSort = {
    init: function(element, valueAccessor) {
        const options = valueAccessor();
        const {
            data,
            filters = [],
            sorters = []
        } = options;
        
        ko.computed(() => {
            let result = ko.unwrap(data);
            
            // 应用过滤器
            filters.forEach(([name, ...args]) => {
                result = ko.filtering.applyFilter(result, name, ...args.map(ko.unwrap));
            });
            
            // 应用排序器
            sorters.forEach(([name, ...args]) => {
                result = ko.filtering.applySort(result, name, ...args.map(ko.unwrap));
            });
            
            // 更新视图
            ko.virtualElements.emptyNode(element);
            ko.virtualElements.setDomNodeChildren(element, [
                document.createTextNode(JSON.stringify(result))
            ]);
        });
    }
};
```

## 3. 高级表单处理

```javascript
// 表单管理器
ko.forms = {
    validators: new Map(),
    formatters: new Map(),
    
    // 注册验证器
    registerValidator: function(name, validator) {
        this.validators.set(name, validator);
    },
    
    // 注册格式化器
    registerFormatter: function(name, formatter) {
        this.formatters.set(name, formatter);
    },
    
    // 创建表单模型
    createForm: function(config) {
        const form = {
            fields: {},
            errors: ko.observableArray([]),
            isDirty: ko.observable(false),
            isValid: ko.computed(() => form.errors().length === 0)
        };
        
        // 初始化字段
        Object.keys(config.fields).forEach(fieldName => {
            const fieldConfig = config.fields[fieldName];
            const field = {
                value: ko.observable(fieldConfig.default),
                errors: ko.observableArray([]),
                isDirty: ko.observable(false),
                isValid: ko.computed(() => field.errors().length === 0)
            };
            
            // 添加验证
            if (fieldConfig.validators) {
                field.value.subscribe(() => {
                    field.errors.removeAll();
                    fieldConfig.validators.forEach(([validatorName, ...args]) => {
                        const validator = this.validators.get(validatorName);
                        if (validator && !validator(field.value(), ...args)) {
                            field.errors.push(`${fieldName} ${validatorName} validation failed`);
                        }
                    });
                });
            }
            
            // 添加格式化
            if (fieldConfig.formatter) {
                const formatter = this.formatters.get(fieldConfig.formatter);
                if (formatter) {
                    field.formatted = ko.computed({
                        read: () => formatter.format(field.value()),
                        write: (value) => field.value(formatter.parse(value))
                    });
                }
            }
            
            form.fields[fieldName] = field;
        });
        
        // 表单方法
        form.reset = function() {
            Object.keys(config.fields).forEach(fieldName => {
                const field = form.fields[fieldName];
                field.value(config.fields[fieldName].default);
                field.isDirty(false);
                field.errors.removeAll();
            });
            form.isDirty(false);
            form.errors.removeAll();
        };
        
        form.validate = function() {
            form.errors.removeAll();
            Object.keys(form.fields).forEach(fieldName => {
                const field = form.fields[fieldName];
                if (!field.isValid()) {
                    form.errors.push(...field.errors());
                }
            });
            return form.isValid();
        };
        
        form.getData = function() {
            const data = {};
            Object.keys(form.fields).forEach(fieldName => {
                data[fieldName] = ko.unwrap(form.fields[fieldName].value);
            });
            return data;
        };
        
        return form;
    }
};

// 预定义验证器
ko.forms.registerValidator('required', value => value != null && value !== '');
ko.forms.registerValidator('email', value => /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(value));
ko.forms.registerValidator('minLength', (value, min) => value.length >= min);
ko.forms.registerValidator('maxLength', (value, max) => value.length <= max);

// 预定义格式化器
ko.forms.registerFormatter('number', {
    format: value => value.toLocaleString(),
    parse: value => parseFloat(value.replace(/,/g, ''))
});

ko.forms.registerFormatter('date', {
    format: value => new Date(value).toLocaleDateString(),
    parse: value => new Date(value).toISOString()
});

// 高级表单绑定
ko.bindingHandlers.form = {
    init: function(element, valueAccessor) {
        const options = valueAccessor();
        const form = ko.forms.createForm(options);
        
        // 处理提交
        element.addEventListener('submit', (e) => {
            e.preventDefault();
            if (form.validate() && options.onSubmit) {
                options.onSubmit(form.getData());
            }
        });
        
        // 绑定到元素
        ko.utils.extend(element, { form });
    }
};
```

## 4. 高级路由功能

```javascript
// 路由增强
ko.router.extend({
    // 路由中间件
    middleware: [],
    
    // 添加中间件
    use: function(middleware) {
        this.middleware.push(middleware);
    },
    
    // 执行中间件
    runMiddleware: async function(to, from) {
        for (const middleware of this.middleware) {
            const result = await middleware(to, from);
            if (result === false) return false;
        }
        return true;
    },
    
    // 路由守卫
    beforeEach: null,
    afterEach: null,
    
    // 导航守卫
    navigate: async function(path, options = {}) {
        const from = this.current();
        const match = this.match(path);
        
        if (!match) return false;
        
        // 执行全局前置守卫
        if (this.beforeEach) {
            const canProceed = await this.beforeEach(match.route, from);
            if (!canProceed) return false;
        }
        
        // 执行中间件
        const canContinue = await this.runMiddleware(match.route, from);
        if (!canContinue) return false;
        
        // 执行原始导航
        const result = await this.constructor.prototype.navigate.call(this, path, options);
        
        // 执行全局后置守卫
        if (result && this.afterEach) {
            this.afterEach(match.route, from);
        }
        
        return result;
    },
    
    // 路由元信息
    meta: new Map(),
    
    // 设置路由元信息
    setMeta: function(path, meta) {
        this.meta.set(path, meta);
    },
    
    // 获取路由元信息
    getMeta: function(path) {
        return this.meta.get(path);
    }
});

// 预定义中间件
ko.router.use(async (to, from) => {
    // 权限检查中间件
    const meta = ko.router.getMeta(to.path);
    if (meta && meta.requiresAuth) {
        // 检查用户是否已登录
        const isAuthenticated = await checkAuth();
        if (!isAuthenticated) {
            ko.router.navigate('/login', { 
                query: { redirect: to.path } 
            });
            return false;
        }
    }
    return true;
});

// 路由元信息示例
ko.router.setMeta('/admin', {
    requiresAuth: true,
    roles: ['admin']
});
```

## 5. 高级组件功能

```javascript
// 组件增强
ko.components.extend({
    // 组件插槽
    slots: {
        register: function(name, template) {
            this.templates = this.templates || new Map();
            this.templates.set(name, template);
        },
        
        get: function(name) {
            return this.templates && this.templates.get(name);
        }
    },
    
    // 组件混入
    mixins: {
        apply: function(target, mixin) {
            Object.keys(mixin).forEach(key => {
                if (key === 'created' || key === 'disposed') {
                    const original = target[key];
                    target[key] = function() {
                        mixin[key].call(this);
                        if (original) original.call(this);
                    };
                } else if (typeof mixin[key] === 'function') {
                    target[key] = mixin[key];
                } else {
                    target[key] = ko.unwrap(mixin[key]);
                }
            });
        }
    },
    
    // 组件继承
    extend: function(base, extension) {
        const result = Object.create(base);
        Object.keys(extension).forEach(key => {
            if (typeof extension[key] === 'function') {
                const original = base[key];
                result[key] = function() {
                    if (original) original.call(this);
                    return extension[key].call(this);
                };
            } else {
                result[key] = extension[key];
            }
        });
        return result;
    }
});

// 组件装饰器
ko.components.decorator = function(component, decorators) {
    return {
        viewModel: function(params) {
            const vm = new component.viewModel(params);
            decorators.forEach(decorator => decorator(vm));
            return vm;
        },
        template: component.template
    };
};

// 预定义混入
const loggingMixin = {
    created: function() {
        console.log(`Component ${this.constructor.name} created`);
    },
    disposed: function() {
        console.log(`Component ${this.constructor.name} disposed`);
    },
    log: function(message) {
        console.log(`[${this.constructor.name}] ${message}`);
    }
};

// 预定义装饰器
const withLogging = (vm) => {
    const original = vm.dispose;
    vm.dispose = function() {
        console.log(`Disposing ${vm.constructor.name}`);
        if (original) original.call(vm);
    };
    return vm;
};
```

## 6. 高级调试工具

```javascript
// 调试工具增强
ko.debug.extend({
    // 性能分析
    profiler: {
        active: false,
        data: [],
        
        start: function() {
            this.active = true;
            this.data = [];
        },
        
        stop: function() {
            this.active = false;
            return this.generateReport();
        },
        
        track: function(name, fn) {
            if (!this.active) return fn();
            
            const start = performance.now();
            const result = fn();
            const duration = performance.now() - start;
            
            this.data.push({
                name,
                duration,
                timestamp: new Date()
            });
            
            return result;
        },
        
        generateReport: function() {
            return {
                totalTime: this.data.reduce((sum, item) => sum + item.duration, 0),
                calls: this.data.length,
                average: this.data.reduce((sum, item) => sum + item.duration, 0) / this.data.length,
                details: this.data
            };
        }
    },
    
    // 状态快照
    snapshot: {
        history: [],
        
        take: function(name) {
            const state = ko.toJS(ko.dataFor(document.body));
            this.history.push({
                name,
                state,
                timestamp: new Date()
            });
        },
        
        compare: function(snapshot1, snapshot2) {
            const diff = {};
            const state1 = this.history.find(s => s.name === snapshot1).state;
            const state2 = this.history.find(s => s.name === snapshot2).state;
            
            this._compareObjects(state1, state2, diff);
            return diff;
        },
        
        _compareObjects: function(obj1, obj2, diff, path = '') {
            Object.keys(obj1).forEach(key => {
                const currentPath = path ? `${path}.${key}` : key;
                
                if (typeof obj1[key] === 'object' && obj1[key] !== null) {
                    this._compareObjects(obj1[key], obj2[key], diff, currentPath);
                } else if (obj1[key] !== obj2[key]) {
                    diff[currentPath] = {
                        before: obj1[key],
                        after: obj2[key]
                    };
                }
            });
        }
    },
    
    // 依赖追踪
    dependencies: {
        track: function(observable) {
            const dependencies = new Set();
            
            ko.computed(() => {
                observable();
                const current = ko.computedContext.getDependencies();
                current.forEach(dep => dependencies.add(dep));
            });
            
            return Array.from(dependencies);
        },
        
        graph: function(root) {
            const graph = new Map();
            const visited = new Set();
            
            const traverse = (observable) => {
                if (visited.has(observable)) return;
                visited.add(observable);
                
                const dependencies = this.track(observable);
                graph.set(observable, dependencies);
                
                dependencies.forEach(dep => traverse(dep));
            };
            
            traverse(root);
            return graph;
        }
    }
});

// 调试绑定处理器
ko.bindingHandlers.debug = {
    init: function(element, valueAccessor) {
        const options = valueAccessor();
        const vm = ko.dataFor(element);
        
        if (options.profile) {
            ko.debug.profiler.start();
        }
        
        if (options.snapshot) {
            ko.debug.snapshot.take('initial');
        }
        
        if (options.track) {
            console.log('Dependencies:', ko.debug.dependencies.track(vm));
        }
    }
};
```

## 注意事项

1. **功能使用建议**
   - 根据实际需求选择功能
   - 避免过度使用高级特性
   - 注意性能影响

2. **开发建议**
   - 合理使用调试工具
   - 做好性能监控
   - 保持代码可维护性

3. **性能优化**
   - 使用性能分析工具
   - 优化组件生命周期
   - 减少不必要的更新

4. **最佳实践**
   - 组件化开发
   - 使用路由中间件
   - 合理使用表单验证 