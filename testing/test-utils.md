# Knockout.js 测试工具和辅助函数指南

## 概述

本指南介绍 Knockout.js 测试中常用的工具和辅助函数，帮助开发者更高效地编写和维护测试代码。

## 测试工具集

### 1. 测试数据生成器

```js
// utils/testDataGenerator.js
class TestDataGenerator {
  static createObservable(initialValue) {
    return ko.observable(initialValue);
  }
  
  static createObservableArray(items = []) {
    return ko.observableArray(items);
  }
  
  static createComputed(func) {
    return ko.computed(func);
  }
  
  static createViewModel() {
    return {
      name: this.createObservable('Test'),
      items: this.createObservableArray([1, 2, 3]),
      total: this.createComputed(() => {
        return this.items().reduce((sum, item) => sum + item, 0);
      })
    };
  }
}
```

### 2. DOM 辅助工具

```js
// utils/domHelpers.js
class DOMHelpers {
  static createTestContainer() {
    const container = document.createElement('div');
    container.id = 'test-container';
    document.body.appendChild(container);
    return container;
  }
  
  static removeTestContainer(container) {
    ko.cleanNode(container);
    container.remove();
  }
  
  static simulateClick(element) {
    const event = new MouseEvent('click', {
      bubbles: true,
      cancelable: true,
      view: window
    });
    element.dispatchEvent(event);
  }
  
  static simulateInput(element, value) {
    element.value = value;
    element.dispatchEvent(new Event('input', { bubbles: true }));
  }
}
```

### 3. 绑定测试工具

```js
// utils/bindingHelpers.js
class BindingHelpers {
  static applyBindings(viewModel, template) {
    const container = DOMHelpers.createTestContainer();
    container.innerHTML = template;
    ko.applyBindings(viewModel, container);
    return container;
  }
  
  static getBindingValue(element, bindingName) {
    const context = ko.contextFor(element);
    const bindings = ko.bindingProvider.instance.getBindings(element, context);
    return bindings[bindingName];
  }
  
  static createBindingContext(data) {
    return {
      $data: data,
      $parent: null,
      $root: data,
      $parentContext: null
    };
  }
}
```

### 4. 异步测试工具

```js
// utils/asyncHelpers.js
class AsyncHelpers {
  static waitForBinding(timeout = 100) {
    return new Promise(resolve => {
      setTimeout(resolve, timeout);
    });
  }
  
  static async waitForComputed(computed) {
    return new Promise(resolve => {
      computed.subscribe(resolve);
    });
  }
  
  static async simulateAjax(response, delay = 100) {
    return new Promise(resolve => {
      setTimeout(() => resolve(response), delay);
    });
  }
}
```

## 测试辅助函数

### 1. 观察者测试

```js
// utils/observableHelpers.js
function trackObservableChanges(observable) {
  const changes = [];
  observable.subscribe(newValue => {
    changes.push(newValue);
  });
  return changes;
}

function createObservableChain() {
  const source = ko.observable('initial');
  const computed = ko.computed(() => source() + ' computed');
  const dependent = ko.computed(() => computed() + ' dependent');
  
  return { source, computed, dependent };
}
```

### 2. 组件测试

```js
// utils/componentHelpers.js
function registerTestComponent(name, config) {
  ko.components.register(name, config);
  return {
    unregister: () => ko.components.unregister(name),
    getConfig: () => ko.components.get(name)
  };
}

function createComponentTest(name, template, viewModel) {
  const container = DOMHelpers.createTestContainer();
  container.innerHTML = template;
  
  const component = registerTestComponent(name, {
    template: template,
    viewModel: viewModel
  });
  
  ko.applyBindings({}, container);
  
  return {
    container,
    cleanup: () => {
      component.unregister();
      DOMHelpers.removeTestContainer(container);
    }
  };
}
```

### 3. 事件测试

```js
// utils/eventHelpers.js
function createEventSpy() {
  const events = [];
  return {
    handler: (event) => events.push(event),
    getEvents: () => events,
    clear: () => events.length = 0
  };
}

function simulateEvents(element, eventTypes) {
  const events = {};
  eventTypes.forEach(type => {
    const event = new Event(type, { bubbles: true });
    element.dispatchEvent(event);
    events[type] = event;
  });
  return events;
}
```

## 测试数据模拟

### 1. 模拟数据生成器

```js
// utils/mockDataGenerator.js
class MockDataGenerator {
  static createUser(overrides = {}) {
    return {
      id: Math.random().toString(36).substr(2, 9),
      name: 'Test User',
      email: 'test@example.com',
      ...overrides
    };
  }
  
  static createList(factory, count = 3, overrides = {}) {
    return Array.from({ length: count }, (_, index) => 
      factory({ ...overrides, id: index + 1 })
    );
  }
}
```

### 2. API 模拟

```js
// utils/apiMock.js
class APIMock {
  static success(data, delay = 100) {
    return AsyncHelpers.simulateAjax({ 
      status: 'success', 
      data 
    }, delay);
  }
  
  static error(message = 'Error', delay = 100) {
    return AsyncHelpers.simulateAjax({
      status: 'error',
      message
    }, delay);
  }
}
```

## 测试场景辅助

### 1. 表单测试

```js
// utils/formHelpers.js
class FormHelpers {
  static fillForm(container, data) {
    Object.entries(data).forEach(([name, value]) => {
      const input = container.querySelector(`[name="${name}"]`);
      if (input) {
        DOMHelpers.simulateInput(input, value);
      }
    });
  }
  
  static getFormData(container) {
    const data = {};
    container.querySelectorAll('input, select, textarea').forEach(input => {
      if (input.name) {
        data[input.name] = input.value;
      }
    });
    return data;
  }
}
```

### 2. 路由测试

```js
// utils/routeHelpers.js
class RouteHelpers {
  static navigateTo(path) {
    window.location.hash = path;
    return AsyncHelpers.waitForBinding();
  }
  
  static getCurrentRoute() {
    return window.location.hash.slice(1);
  }
}
```

## 使用示例

### 1. 完整测试套件

```js
describe('Complete Test Suite', () => {
  let container;
  let viewModel;
  
  beforeEach(() => {
    container = DOMHelpers.createTestContainer();
    viewModel = TestDataGenerator.createViewModel();
  });
  
  afterEach(() => {
    DOMHelpers.removeTestContainer(container);
  });
  
  it('should track observable changes', () => {
    const changes = trackObservableChanges(viewModel.name);
    viewModel.name('New Name');
    expect(changes).toContain('New Name');
  });
  
  it('should handle form submission', async () => {
    const form = FormHelpers.fillForm(container, {
      name: 'Test User',
      email: 'test@example.com'
    });
    
    const spy = createEventSpy();
    form.addEventListener('submit', spy.handler);
    
    DOMHelpers.simulateClick(form.querySelector('button[type="submit"]'));
    expect(spy.getEvents()).toHaveLength(1);
  });
});
```

## 最佳实践

1. **工具组织**
   - 按功能分类工具
   - 保持工具函数简单
   - 提供清晰的文档

2. **代码复用**
   - 抽取常用操作
   - 创建通用辅助函数
   - 维护工具库

3. **测试可维护性**
   - 使用描述性命名
   - 添加适当注释
   - 保持代码整洁

## 总结

测试工具和辅助函数的关键点：

1. 提供可重用的测试工具
2. 简化测试代码编写
3. 提高测试可维护性
4. 确保测试可靠性
5. 支持不同测试场景 