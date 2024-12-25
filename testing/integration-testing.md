# Knockout.js 集成测试指南

## 概述

本指南介绍如何为 Knockout.js 应用编写集成测试，包括端到端测试、组件集成测试和测试环境搭建。

## 测试环境搭建

### 1. 安装依赖

```bash
# 安装测试工具
npm install --save-dev cypress @testing-library/cypress
npm install --save-dev @cypress/webpack-preprocessor
```

### 2. 配置 Cypress

```js
// cypress.config.js
const { defineConfig } = require('cypress');

module.exports = defineConfig({
  e2e: {
    baseUrl: 'http://localhost:3000',
    supportFile: 'cypress/support/e2e.js',
    specPattern: 'cypress/e2e/**/*.cy.{js,jsx,ts,tsx}'
  }
});
```

### 3. 设置测试环境

```js
// cypress/support/e2e.js
import '@testing-library/cypress/add-commands';

// 添加 Knockout.js 支持
Cypress.Commands.add('koApplyBindings', (viewModel, element) => {
  return cy.window().then((win) => {
    win.ko.applyBindings(viewModel, element);
  });
});
```

## 端到端测试

### 1. 页面导航测试

```js
describe('Navigation Tests', () => {
  beforeEach(() => {
    cy.visit('/');
  });

  it('should navigate between pages', () => {
    // 点击导航链接
    cy.get('[data-test="nav-home"]').click();
    cy.url().should('include', '/home');

    cy.get('[data-test="nav-about"]').click();
    cy.url().should('include', '/about');
  });

  it('should load page content', () => {
    cy.get('[data-test="page-title"]')
      .should('be.visible')
      .and('contain', 'Welcome');
  });
});
```

### 2. 表单交互测试

```js
describe('Form Integration', () => {
  beforeEach(() => {
    cy.visit('/form');
  });

  it('should submit form data', () => {
    // 填写表单
    cy.get('[data-test="name-input"]')
      .type('John Doe');
    
    cy.get('[data-test="email-input"]')
      .type('john@example.com');
    
    cy.get('[data-test="submit-button"]')
      .click();
    
    // 验证结果
    cy.get('[data-test="success-message"]')
      .should('be.visible')
      .and('contain', 'Form submitted');
  });

  it('should validate form fields', () => {
    cy.get('[data-test="submit-button"]').click();
    
    cy.get('[data-test="error-message"]')
      .should('be.visible')
      .and('contain', 'Please fill all fields');
  });
});
```

### 3. 数据加载测试

```js
describe('Data Loading', () => {
  beforeEach(() => {
    cy.intercept('GET', '/api/data', {
      fixture: 'testData.json'
    }).as('getData');
    
    cy.visit('/data');
  });

  it('should load and display data', () => {
    // 等待数据加载
    cy.wait('@getData');
    
    // 验证数据显示
    cy.get('[data-test="data-list"]')
      .children()
      .should('have.length', 3);
    
    cy.get('[data-test="data-item"]')
      .first()
      .should('contain', 'Test Item 1');
  });

  it('should handle loading state', () => {
    cy.get('[data-test="loading-indicator"]')
      .should('be.visible');
    
    cy.wait('@getData');
    
    cy.get('[data-test="loading-indicator"]')
      .should('not.exist');
  });
});
```

## 组件集成测试

### 1. 组件交互测试

```js
describe('Component Integration', () => {
  beforeEach(() => {
    cy.visit('/components');
  });

  it('should interact between components', () => {
    // 父组件操作
    cy.get('[data-test="parent-input"]')
      .type('New Value');
    
    // 验证子组件更新
    cy.get('[data-test="child-display"]')
      .should('contain', 'New Value');
    
    // 子组件操作
    cy.get('[data-test="child-button"]')
      .click();
    
    // 验证父组件更新
    cy.get('[data-test="parent-counter"]')
      .should('contain', '1');
  });
});
```

### 2. 状态管理测试

```js
describe('State Management', () => {
  beforeEach(() => {
    cy.visit('/state');
  });

  it('should manage global state', () => {
    // 更新状态
    cy.get('[data-test="increment-button"]')
      .click();
    
    // 验证多个组件更新
    cy.get('[data-test="counter-display-1"]')
      .should('contain', '1');
    
    cy.get('[data-test="counter-display-2"]')
      .should('contain', '1');
  });
});
```

## 性能测试

### 1. 加载时间测试

```js
describe('Performance Tests', () => {
  it('should load page within threshold', () => {
    cy.visit('/', {
      onBeforeLoad: (win) => {
        win.performance.mark('start');
      }
    });

    cy.window().then((win) => {
      win.performance.mark('end');
      const measure = win.performance.measure('pageLoad', 'start', 'end');
      expect(measure.duration).to.be.lessThan(1000);
    });
  });
});
```

### 2. 渲染性能测试

```js
describe('Rendering Performance', () => {
  it('should render large lists efficiently', () => {
    cy.visit('/list');
    
    const startTime = Date.now();
    
    cy.get('[data-test="add-items"]')
      .click();
    
    cy.get('[data-test="list-item"]')
      .should('have.length', 1000)
      .then(() => {
        const endTime = Date.now();
        expect(endTime - startTime).to.be.lessThan(500);
      });
  });
});
```

## 测试最佳实践

### 1. 测试数据管理

```js
// cypress/fixtures/testData.json
{
  "users": [
    {
      "id": 1,
      "name": "Test User 1"
    },
    {
      "id": 2,
      "name": "Test User 2"
    }
  ]
}
```

### 2. 自定义命令

```js
// cypress/support/commands.js
Cypress.Commands.add('login', (username, password) => {
  cy.get('[data-test="username"]').type(username);
  cy.get('[data-test="password"]').type(password);
  cy.get('[data-test="login-button"]').click();
});
```

### 3. 测试隔离

```js
describe('Isolated Tests', () => {
  beforeEach(() => {
    cy.clearLocalStorage();
    cy.clearCookies();
  });

  it('should run in isolation', () => {
    // 测试代码
  });
});
```

## 常见问题解决

### 1. 异步操作处理

```js
describe('Async Operations', () => {
  it('should handle async data loading', () => {
    cy.intercept('GET', '/api/data').as('getData');
    cy.visit('/data');
    
    cy.wait('@getData')
      .its('response.statusCode')
      .should('eq', 200);
    
    cy.get('[data-test="data-container"]')
      .should('be.visible');
  });
});
```

### 2. 网络请求模拟

```js
describe('Network Mocking', () => {
  it('should handle API errors', () => {
    cy.intercept('GET', '/api/data', {
      statusCode: 500,
      body: { error: 'Server Error' }
    }).as('getDataError');
    
    cy.visit('/data');
    
    cy.get('[data-test="error-message"]')
      .should('contain', 'Failed to load data');
  });
});
```

## 工具和资源

1. **测试工具**
   - Cypress
   - TestCafe
   - Playwright

2. **测试辅助工具**
   - Cypress Testing Library
   - Cypress Recorder
   - Chrome DevTools

3. **持续集成工具**
   - Jenkins
   - GitHub Actions
   - CircleCI

## 总结

集成测试的关键点：

1. 合理设置测试环境
2. 编写全面的端到端测试
3. 注重组件间的交互测试
4. 关注性能测试
5. 保持测试代码的可维护性 