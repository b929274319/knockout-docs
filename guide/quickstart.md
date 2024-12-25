# 快速开始

本指南将帮助你快速上手 Knockout.js，了解其基本用法和核心概念。

## 安装

### 方法一：使用 CDN

```html
<script src="http://cdnjs.cloudflare.com/ajax/libs/knockout/3.5.1/knockout-min.js"></script>
```

### 方法二：使用 npm

```bash
npm install knockout
```

## 第一个示例

让我们创建一个简单的用户信息表单：

```html
<!DOCTYPE html>
<html>
<head>
    <title>Knockout.js 快速开始</title>
</head>
<body>
    <h2>用户信息</h2>
    
    <p>
        姓名：<input data-bind="value: name" />
    </p>
    <p>
        年龄：<input data-bind="value: age" type="number" />
    </p>
    <p>
        个人简介：<textarea data-bind="value: bio"></textarea>
    </p>
    
    <h3>预览</h3>
    <p>姓名：<span data-bind="text: name"></span></p>
    <p>年龄：<span data-bind="text: age"></span></p>
    <p>简介：<span data-bind="text: bio"></span></p>

    <script src="http://cdnjs.cloudflare.com/ajax/libs/knockout/3.5.1/knockout-min.js"></script>
    <script>
        // 创建 ViewModel
        function UserViewModel() {
            this.name = ko.observable("张三");
            this.age = ko.observable(25);
            this.bio = ko.observable("这是一段个人简介");
        }

        // 应用绑定
        ko.applyBindings(new UserViewModel());
    </script>
</body>
</html>
```

## 代码解释

1. **引入 Knockout.js**
   ```html
   <script src="http://cdnjs.cloudflare.com/ajax/libs/knockout/3.5.1/knockout-min.js"></script>
   ```

2. **创建数据模型（ViewModel）**
   ```javascript
   function UserViewModel() {
       this.name = ko.observable("张三");
       this.age = ko.observable(25);
       this.bio = ko.observable("这是一段个人简介");
   }
   ```
   - `ko.observable()` 创建可观察对象，当数据变化时会自动更新 UI

3. **使用数据绑定**
   ```html
   <input data-bind="value: name" />
   <span data-bind="text: name"></span>
   ```
   - `data-bind` 属性用于将 HTML 元素与 ViewModel 中的数据关联
   - `value` 绑定用于表单输入
   - `text` 绑定用于显示文本

4. **激活 Knockout**
   ```javascript
   ko.applyBindings(new UserViewModel());
   ```
   - 将 ViewModel 与页面进行绑定

## 运行效果

- 在输入框中输入文本，预览区域会实时更新
- 所有更改都是自动的，不需要手动刷新
- 数据在 ViewModel 和 UI 之间双向同步

## 下一步

- 学习 [基本概念](concepts.md) 深入理解 Knockout.js
- 了解更多 [数据绑定](../bindings/text-appearance.md) 的用法
- 探索 [可观察对象](../core/observables.md) 的高级特性

## 常见问题

1. **为什么要使用 `ko.observable()`？**
   - 创建响应式数据，实现自动 UI 更新
   - 跟踪数据变化
   - 支持计算属性和依赖关系

2. **如何处理表单提交？**
   ```javascript
   function UserViewModel() {
       // ... 前面的代码 ...
       
       this.submit = function() {
           var data = {
               name: this.name(),
               age: this.age(),
               bio: this.bio()
           };
           console.log('提交的数据：', data);
       };
   }
   ```

3. **如何获取可观察对象的值？**
   ```javascript
   var name = viewModel.name(); // 不要忘记括号
   ```

4. **如何设置可观察对象的值？**
   ```javascript
   viewModel.name("李四"); // 使用括号设置新值
   ``` 