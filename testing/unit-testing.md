# Knockout.js 单元测试指南

## 概述

本指南介绍如何为 Knockout.js 应用编写单元测试，包括测试框架的选择、测试用例的编写和最佳实践。

## 测试框架选择

### 1. Jest

推荐使用 Jest 作为主要测试框架：

```bash
# 安装 Jest
npm install --save-dev jest @types/jest
```

配置 `package.json`：
```json
{
  "scripts": {
    "test": "jest",
    "test:watch": "jest --watch"
  },
  "jest": {
    "testEnvironment": "jsdom"
  }
}
```

### 2. Jasmine

另一个流行的选择是 Jasmine：

```bash
# 安装 Jasmine
npm install --save-dev jasmine jasmine-core
```

### 3. Mocha + Chai

也可以使用 Mocha 和 Chai 的组合：

```bash
# 安装 Mocha 和 Chai
npm install --save-dev mocha chai
```

## 测试用例编写

### 1. 可观察对象测试

```js
describe('Observable Tests', () => {
  it('should update observable value', () => {
    // 创建视图模型
    const viewModel = {
      name: ko.observable('John')
    };
    
    // 测试初始值
    expect(viewModel.name()).toBe('John');
    
    // 更新值
    viewModel.name('Jane');
    expect(viewModel.name()).toBe('Jane');
  });
  
  it('should notify subscribers', () => {
    const observable = ko.observable('initial');
    let notified = false;
    
    // 订阅变化
    observable.subscribe(() => {
      notified = true;
    });
    
    // 触发变化
    observable('new value');
    expect(notified).toBe(true);
  });
});
```

### 2. 计算属性测试

```js
describe('Computed Observable Tests', () => {
  it('should compute derived value', () => {
    const viewModel = {
      firstName: ko.observable('John'),
      lastName: ko.observable('Doe'),
      fullName: ko.computed(function() {
        return this.firstName() + ' ' + this.lastName();
      })
    };
    
    expect(viewModel.fullName()).toBe('John Doe');
    
    viewModel.firstName('Jane');
    expect(viewModel.fullName()).toBe('Jane Doe');
  });
  
  it('should handle dependencies correctly', () => {
    const viewModel = {
      price: ko.observable(100),
      quantity: ko.observable(2),
      total: ko.computed(function() {
        return this.price() * this.quantity();
      })
    };
    
    expect(viewModel.total()).toBe(200);
    
    viewModel.quantity(3);
    expect(viewModel.total()).toBe(300);
  });
});
```

### 3. 组件测试

```js
describe('Component Tests', () => {
  beforeEach(() => {
    // 注册测试组件
    ko.components.register('test-component', {
      viewModel: function(params) {
        this.value = ko.observable(params.initialValue);
        this.increment = function() {
          this.value(this.value() + 1);
        };
      },
      template: '<div><span data-bind="text: value"></span></div>'
    });
  });
  
  afterEach(() => {
    // 清理组件注册
    ko.components.unregister('test-component');
  });
  
  it('should initialize with params', () => {
    const viewModel = new ko.components.get('test-component').viewModel({
      initialValue: 10
    });
    
    expect(viewModel.value()).toBe(10);
  });
  
  it('should handle user interactions', () => {
    const viewModel = new ko.components.get('test-component').viewModel({
      initialValue: 0
    });
    
    viewModel.increment();
    expect(viewModel.value()).toBe(1);
  });
});
```

### 4. 绑定处理器测试

```js
describe('Custom Binding Tests', () => {
  it('should apply custom binding', () => {
    // 创建自定义绑定
    ko.bindingHandlers.testBinding = {
      init: function(element, valueAccessor) {
        const value = ko.unwrap(valueAccessor());
        element.textContent = value;
      },
      update: function(element, valueAccessor) {
        const value = ko.unwrap(valueAccessor());
        element.textContent = value;
      }
    };
    
    // 创建测试元素
    const element = document.createElement('div');
    const observable = ko.observable('test');
    
    // 应用绑定
    ko.applyBindingsToNode(element, {
      testBinding: observable
    });
    
    expect(element.textContent).toBe('test');
    
    // 更新值
    observable('updated');
    expect(element.textContent).toBe('updated');
  });
});
```

## 测试工具和辅助函数

### 1. DOM 模拟

```js
// 创建测试容器
function createTestContainer() {
  const container = document.createElement('div');
  document.body.appendChild(container);
  return container;
}

// 清理测试容器
function removeTestContainer(container) {
  ko.cleanNode(container);
  container.remove();
}
```

### 2. 绑定上下文模拟

```js
function createBindingContext(data) {
  return {
    $data: data,
    $parent: null,
    $root: data,
    $parentContext: null
  };
}
```

### 3. 异步测试辅助

```js
async function waitForBinding(element) {
  return new Promise(resolve => {
    setTimeout(() => {
      resolve();
    }, 0);
  });
}
```

## 最佳实践

### 1. 测试隔离

```js
describe('Isolated Tests', () => {
  let container;
  
  beforeEach(() => {
    container = createTestContainer();
  });
  
  afterEach(() => {
    removeTestContainer(container);
  });
  
  it('should test in isolation', () => {
    // 测试代码
  });
});
```

### 2. 模拟依赖

```js
describe('Mocked Dependencies', () => {
  it('should mock ajax calls', () => {
    const mockAjax = jest.fn().mockResolvedValue({
      data: 'test'
    });
    
    // 注入模拟的 ajax 调用
    const viewModel = new ViewModel({
      ajax: mockAjax
    });
    
    // 测试代码
  });
});
```

### 3. 测试覆盖率

```bash
# 运行测试并生成覆盖率报告
jest --coverage
```

## 常见问题

### 1. 异步测试

```js
describe('Async Tests', () => {
  it('should handle async operations', async () => {
    const viewModel = new AsyncViewModel();
    
    await viewModel.loadData();
    expect(viewModel.data()).not.toBeNull();
  });
});
```

### 2. 事件测试

```js
describe('Event Tests', () => {
  it('should handle click events', () => {
    const viewModel = new ViewModel();
    const button = document.createElement('button');
    
    ko.applyBindingsToNode(button, {
      click: viewModel.handleClick
    });
    
    button.click();
    expect(viewModel.clicked()).toBe(true);
  });
});
```

## 工具和资源

1. **测试工具**
   - Jest
   - Jasmine
   - Mocha + Chai

2. **代码覆盖率工具**
   - Istanbul
   - Jest Coverage

3. **调试工具**
   - Chrome DevTools
   - VS Code Debugger

## 总结

编写单元测试的关键点：

1. 选择合适的测试框架
2. 保持测试代码的简洁和可维护性
3. 注重测试覆盖率
4. 使用适当的模拟和存根
5. 遵循测试最佳实践 