# Knockout.js 基本概念

本文将介绍 Knockout.js 的核心概念，帮助你理解框架的工作原理。

## MVVM 模式

Knockout.js 采用 MVVM（Model-View-ViewModel）架构模式：

- **Model（模型）**：原始数据
- **View（视图）**：用户界面（HTML）
- **ViewModel（视图模型）**：连接视图和模型的桥梁

```javascript
// Model - 原始数据
var data = {
    name: "张三",
    age: 25
};

// ViewModel - 将原始数据转换为可观察对象
function PersonViewModel() {
    this.name = ko.observable(data.name);
    this.age = ko.observable(data.age);
    
    // 计算属性
    this.isAdult = ko.computed(function() {
        return this.age() >= 18;
    }, this);
}

// View - HTML 中的绑定
// <div data-bind="text: name"></div>
// <div data-bind="text: isAdult() ? '成年人' : '未成年'"></div>
```

## 可观察对象（Observables）

可观察对象是 Knockout.js 的核心特性之一：

### 1. 基本可观察对象

```javascript
// 创建可观察对象
var name = ko.observable("张三");

// 读取值
console.log(name()); // 输出: "张三"

// 设置值
name("李四");
```

### 2. 可观察数组

```javascript
// 创建可观察数组
var fruits = ko.observableArray(["苹果", "香蕉"]);

// 数组操作
fruits.push("橙子");     // 添加元素
fruits.remove("香蕉");   // 删除元素
fruits.removeAll();      // 清空数组
```

### 3. 计算属性

```javascript
function ViewModel() {
    this.firstName = ko.observable("张");
    this.lastName = ko.observable("三");
    
    this.fullName = ko.computed(function() {
        return this.firstName() + this.lastName();
    }, this);
}
```

## 数据绑定

Knockout.js 提供多种数据绑定方式：

### 1. 文本和显示绑定

```html
<!-- 文本绑定 -->
<span data-bind="text: name"></span>

<!-- HTML 绑定 -->
<div data-bind="html: description"></div>

<!-- 可见性绑定 -->
<div data-bind="visible: isVisible"></div>
```

### 2. 表单绑定

```html
<!-- 值绑定 -->
<input data-bind="value: userName" />

<!-- 检查框绑定 -->
<input type="checkbox" data-bind="checked: isAgree" />

<!-- 选项绑定 -->
<select data-bind="options: countries, value: selectedCountry"></select>
```

### 3. 事件绑定

```html
<!-- 点击事件绑定 -->
<button data-bind="click: save">保存</button>

<!-- 表单提交绑定 -->
<form data-bind="submit: handleSubmit">
    <!-- 表单内容 -->
</form>
```

## 模板

Knockout.js 支持模板化：

```html
<!-- 使用 foreach 绑定 -->
<ul data-bind="foreach: items">
    <li>
        <span data-bind="text: name"></span>
        <span data-bind="text: price"></span>
    </li>
</ul>

<!-- 使用 template 绑定 -->
<script type="text/html" id="personTemplate">
    <h3 data-bind="text: name"></h3>
    <p data-bind="text: bio"></p>
</script>

<div data-bind="template: { name: 'personTemplate', data: person }"></div>
```

## 生命周期

Knockout.js 的主要生命周期事件：

1. **初始化**
   ```javascript
   ko.applyBindings(new ViewModel());
   ```

2. **更新**
   - 当可观察对象值改变时自动触发
   - 计算属性自动重新计算
   - UI 自动更新

3. **清理**
   ```javascript
   ko.cleanNode(element);
   ```

## 最佳实践

1. **使用纯 JavaScript 对象初始化 ViewModel**
   ```javascript
   var data = { name: "张三", age: 25 };
   
   function ViewModel(data) {
       this.name = ko.observable(data.name);
       this.age = ko.observable(data.age);
   }
   ```

2. **合理使用计算属性**
   ```javascript
   this.fullName = ko.computed({
       read: function() {
           return this.firstName() + " " + this.lastName();
       },
       write: function(value) {
           var parts = value.split(" ");
           this.firstName(parts[0]);
           this.lastName(parts[1]);
       }
   }, this);
   ```

3. **避免过度使用可观察对象**
   - 只将需要监听变化的数据设为可观察对象
   - 静态数据使用普通属性

4. **及时清理不需要的订阅**
   ```javascript
   var subscription = myObservable.subscribe(function(newValue) {
       // 处理变化
   });
   
   // 清理订阅
   subscription.dispose();
   ```

## 下一步

- 深入学习 [可观察对象](../core/observables.md)
- 了解更多 [数据绑定](../bindings/text-appearance.md) 用法
- 探索 [组件开发](../advanced/components.md) 