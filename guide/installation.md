# 安装和配置

本文将介绍如何在你的项目中安装和配置 Knockout.js。

## 安装方式

### 1. 使用 CDN

最简单的方式是使用 CDN：

```html
<script src="http://cdnjs.cloudflare.com/ajax/libs/knockout/3.5.1/knockout-min.js"></script>
```

### 2. 使用 npm

如果你的项目使用 npm 管理依赖，可以通过以下命令安装：

```bash
npm install knockout --save
```

然后在你的代码中引入：

```javascript
import ko from 'knockout';
```

### 3. 下载到本地

你也可以下载 Knockout.js 文件到本地：

1. 访问 [Knockout.js 官网](http://knockoutjs.com/downloads/index.html)
2. 下载最新版本
3. 将文件放到你的项目目录中
4. 在 HTML 中引入：

```html
<script src="path/to/knockout.js"></script>
```

## 版本选择

当前最新稳定版本是 3.5.1，建议使用此版本。

### 各版本特性对比

| 版本   | 主要特性                    | 最低兼容浏览器要求          |
|--------|----------------------------|---------------------------|
| 3.5.1  | 完整的 ES5 支持            | IE9+                      |
| 3.4.2  | 性能优化                   | IE8+                      |
| 3.3.0  | 组件增强                   | IE8+                      |

## 浏览器兼容性

Knockout.js 支持所有主流浏览器：

- Chrome（最新版）
- Firefox（最新版）
- Safari（最新版）
- Edge（所有版本）
- Internet Explorer 9+

## 开发环境配置

### 1. 基本配置

```html
<!DOCTYPE html>
<html>
<head>
    <title>Knockout.js 项目</title>
    <script src="http://cdnjs.cloudflare.com/ajax/libs/knockout/3.5.1/knockout-min.js"></script>
</head>
<body>
    <!-- 你的代码 -->
    <script src="你的脚本文件.js"></script>
</body>
</html>
```

### 2. 使用 TypeScript

如果你想使用 TypeScript，首先安装类型定义：

```bash
npm install --save-dev @types/knockout
```

然后在你的 TypeScript 文件中：

```typescript
import * as ko from 'knockout';

class ViewModel {
    name: KnockoutObservable<string>;
    
    constructor() {
        this.name = ko.observable('张三');
    }
}
```

### 3. 开发工具推荐

- Visual Studio Code
  - 安装 "Knockout.js Snippets" 插件
  - 安装 "JavaScript (ES6) code snippets" 插件

- Chrome 开发者工具
  - 安装 "Knockout Context Debugger" 扩展

## 调试技巧

### 1. 开启调试信息

```javascript
ko.options.deferUpdates = true;
```

### 2. 使用 ko.toJS() 查看数据

```javascript
// 在控制台中查看数据模型的当前状态
console.log(ko.toJS(viewModel));
```

### 3. 监听变化

```javascript
myObservable.subscribe(function(newValue) {
    console.log('值变化为：', newValue);
});
```

## 常见问题

### 1. 引入顺序问题

确保 Knockout.js 在其他依赖它的脚本之前引入：

```html
<!-- 正确的顺序 -->
<script src="knockout.js"></script>
<script src="你的脚本.js"></script>

<!-- 错误的顺序 -->
<script src="你的脚本.js"></script>
<script src="knockout.js"></script>
```

### 2. 绑定错误

如果遇到绑定错误，检查控制台信息，常见原因：

- 变量名拼写错误
- 忘记使用 ko.observable()
- 绑定语法错误

### 3. 性能问题

- 避免在一个页面中创建过多的观察者
- 使用 `ko.computed` 时注意依赖收集
- 大列表考虑使用虚拟滚动

## 下一步

- 阅读 [基本概念](concepts.md) 了解核心原理
- 查看 [快速开始](quickstart.md) 动手实践
- 探索 [可观察对象](../core/observables.md) 的用法 