# Knockout.js 测试指南

本文介绍如何对 Knockout.js 应用进行测试，包括单元测试、集成测试和端到端测试。

## 单元测试

### 1. 测试可观察对象

```javascript
describe('Observable Tests', () => {
    it('should update value when changed', () => {
        // 创建可观察对象
        const observable = ko.observable('initial');
        
        // 验证初始值
        expect(observable()).toBe('initial');
        
        // 更新值
        observable('updated');
        
        // 验证更新后的值
        expect(observable()).toBe('updated');
    });
    
    it('should notify subscribers', () => {
        const observable = ko.observable(0);
        let notified = false;
        
        // 添加订阅
        observable.subscribe(value => {
            notified = true;
        });
        
        // 更新值
        observable(1);
        
        // 验证订阅者是否被通知
        expect(notified).toBe(true);
    });
});
```

### 2. 测试计算属性

```javascript
describe('Computed Tests', () => {
    it('should update when dependencies change', () => {
        const firstName = ko.observable('John');
        const lastName = ko.observable('Doe');
        
        const fullName = ko.computed(() => {
            return `${firstName()} ${lastName()}`;
        });
        
        // 验证初始值
        expect(fullName()).toBe('John Doe');
        
        // 更新依赖
        firstName('Jane');
        
        // 验证计算属性是否更新
        expect(fullName()).toBe('Jane Doe');
    });
});
```

### 3. 测试视图模型

```javascript
describe('ViewModel Tests', () => {
    let viewModel;
    
    beforeEach(() => {
        viewModel = new ViewModel();
    });
    
    it('should initialize with default values', () => {
        expect(viewModel.items().length).toBe(0);
        expect(viewModel.selectedItem()).toBeNull();
    });
    
    it('should add items correctly', () => {
        viewModel.addItem('New Item');
        expect(viewModel.items().length).toBe(1);
        expect(viewModel.items()[0].name()).toBe('New Item');
    });
});
```

## 集成测试

### 1. 测试绑定

```javascript
describe('Binding Tests', () => {
    let element;
    
    beforeEach(() => {
        element = document.createElement('div');
        document.body.appendChild(element);
    });
    
    afterEach(() => {
        document.body.removeChild(element);
    });
    
    it('should update text binding', () => {
        const viewModel = {
            message: ko.observable('Hello')
        };
        
        element.setAttribute('data-bind', 'text: message');
        ko.applyBindings(viewModel, element);
        
        // 验证初始渲染
        expect(element.textContent).toBe('Hello');
        
        // 更新可观察对象
        viewModel.message('Updated');
        
        // 验证 DOM 更新
        expect(element.textContent).toBe('Updated');
    });
});
```

### 2. 测试组件

```javascript
describe('Component Tests', () => {
    beforeAll(() => {
        ko.components.register('test-component', {
            template: '<div data-bind="text: message"></div>',
            viewModel: function(params) {
                this.message = ko.observable(params.initialText);
            }
        });
    });
    
    it('should render component', (done) => {
        const element = document.createElement('div');
        element.innerHTML = '<test-component params="initialText: \'Test\'"></test-component>';
        
        ko.applyBindings({}, element);
        
        // 等待组件渲染
        setTimeout(() => {
            expect(element.textContent).toBe('Test');
            done();
        }, 0);
    });
});
```

## 端到端测试

### 1. 使用 Cypress

```javascript
// cypress/integration/app.spec.js
describe('App Tests', () => {
    beforeEach(() => {
        cy.visit('/');
    });
    
    it('should add new item', () => {
        // 输入新项目
        cy.get('#new-item')
            .type('Test Item');
            
        // 点击添加按钮
        cy.get('#add-button')
            .click();
            
        // 验证列表更新
        cy.get('.item-list')
            .should('contain', 'Test Item');
    });
});
```

### 2. 使用 Selenium

```javascript
const { Builder, By, until } = require('selenium-webdriver');

describe('Selenium Tests', () => {
    let driver;
    
    beforeAll(async () => {
        driver = await new Builder().forBrowser('chrome').build();
    });
    
    afterAll(async () => {
        await driver.quit();
    });
    
    it('should login successfully', async () => {
        await driver.get('http://localhost:8080');
        
        // 输入登录信息
        await driver.findElement(By.id('username')).sendKeys('user');
        await driver.findElement(By.id('password')).sendKeys('pass');
        
        // 点击登录按钮
        await driver.findElement(By.id('login-button')).click();
        
        // 等待登录成功
        const welcomeMessage = await driver.wait(
            until.elementLocated(By.className('welcome')),
            5000
        );
        
        const text = await welcomeMessage.getText();
        expect(text).toContain('Welcome');
    });
});
```

## 测试工具配置

### 1. Jest 配置

```javascript
// jest.config.js
module.exports = {
    setupFilesAfterEnv: ['<rootDir>/test/setup.js'],
    testEnvironment: 'jsdom',
    moduleNameMapper: {
        '^@/(.*)$': '<rootDir>/src/$1'
    }
};
```

### 2. Cypress 配置

```javascript
// cypress.json
{
    "baseUrl": "http://localhost:8080",
    "viewportWidth": 1280,
    "viewportHeight": 720,
    "video": false
}
```

## 测试最佳实践

### 1. 组织测试代码

```javascript
// tests/unit/viewModels/userViewModel.test.js
describe('UserViewModel', () => {
    describe('initialization', () => {
        // 初始化测试
    });
    
    describe('operations', () => {
        // 操作测试
    });
    
    describe('validation', () => {
        // 验证测试
    });
});
```

### 2. 使用测试工具函数

```javascript
// tests/helpers/testUtils.js
export function createTestViewModel(options = {}) {
    return new ViewModel({
        data: options.data || [],
        config: options.config || {}
    });
}

export function simulateUserAction(element, action) {
    // 模拟用户操作
}
```

## 测试覆盖率

### 1. 配置覆盖率报告

```javascript
// jest.config.js
module.exports = {
    collectCoverage: true,
    coverageDirectory: 'coverage',
    coverageReporters: ['text', 'lcov'],
    collectCoverageFrom: [
        'src/**/*.js',
        '!src/vendor/**'
    ]
};
```

### 2. 分析覆盖率报告

```bash
# 运行测试并生成覆盖率报告
npm test -- --coverage

# 查看详细报告
open coverage/lcov-report/index.html
```

## 持续集成测试

### 1. GitHub Actions 配置

```yaml
# .github/workflows/test.yml
name: Tests

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v2
    - name: Use Node.js
      uses: actions/setup-node@v2
      with:
        node-version: '14.x'
    - run: npm ci
    - run: npm test
    - name: Upload coverage
      uses: codecov/codecov-action@v2
```

### 2. Jenkins 配置

```groovy
// Jenkinsfile
pipeline {
    agent any
    stages {
        stage('Test') {
            steps {
                sh 'npm ci'
                sh 'npm test'
            }
            post {
                always {
                    junit 'test-results/*.xml'
                    publishHTML(target: [
                        reportDir: 'coverage',
                        reportFiles: 'index.html',
                        reportName: 'Coverage Report'
                    ])
                }
            }
        }
    }
}
```

## 调试测试

### 1. 使用调试工具

```javascript
describe('Debug Test', () => {
    it('should debug test', () => {
        debugger; // 设置断点
        const result = someFunction();
        expect(result).toBe(expected);
    });
});
```

### 2. 使用测试日志

```javascript
describe('Logging Test', () => {
    beforeEach(() => {
        console.log('Test started');
    });
    
    afterEach(() => {
        console.log('Test completed');
    });
    
    it('should log test steps', () => {
        console.log('Step 1');
        // 测试步骤
        console.log('Step 2');
        // 更多步骤
    });
});
```

## 常见问题

1. **异步测试**
   - 使用 async/await
   - 正确处理 Promise
   - 设置合理的超时时间

2. **DOM 测试**
   - 使用 jsdom
   - 清理测试环境
   - 模拟浏览器事件

3. **测试性能**
   - 优化测试套件
   - 并行运行测试
   - 使用测试缓存

## 总结

测试的关键点：

1. 编写全面的测试用例
2. 保持测试代码质量
3. 自动化测试流程
4. 维护测试覆盖率
5. 持续改进测试策略 