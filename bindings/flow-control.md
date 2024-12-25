# 流程控制绑定

本文介绍 Knockout.js 中用于控制 UI 渲染流程的绑定。这些绑定可以帮助你实现条件渲染、循环渲染等复杂的界面逻辑。

## 条件渲染

### if 绑定

根据条件决定是否渲染元素。

```html
<div data-bind="if: isLoggedIn">
    <h2>欢迎回来，<span data-bind="text: userName"></span>！</h2>
</div>

<script>
var viewModel = {
    isLoggedIn: ko.observable(true),
    userName: ko.observable("张三")
};
</script>
```

### ifnot 绑定

与 if 绑定相反的条件渲染。

```html
<div data-bind="ifnot: isLoggedIn">
    <p>请先登录</p>
</div>
```

### with 绑定

创建新的绑定上下文。

```html
<div data-bind="with: userProfile">
    <p>姓名：<span data-bind="text: name"></span></p>
    <p>年龄：<span data-bind="text: age"></span></p>
</div>

<script>
var viewModel = {
    userProfile: {
        name: ko.observable("张三"),
        age: ko.observable(25)
    }
};
</script>
```

## 循环渲染

### foreach 绑定

遍历数组或对象集合。

```html
<!-- 基本用法 -->
<ul data-bind="foreach: items">
    <li data-bind="text: $data"></li>
</ul>

<!-- 对象数组 -->
<table>
    <tbody data-bind="foreach: users">
        <tr>
            <td data-bind="text: name"></td>
            <td data-bind="text: age"></td>
        </tr>
    </tbody>
</table>

<script>
var viewModel = {
    // 简单数组
    items: ["项目1", "项目2", "项目3"],
    
    // 对象数组
    users: [
        { name: "张三", age: 25 },
        { name: "李四", age: 30 }
    ]
};
</script>
```

### 特殊变量

foreach 绑定中可用的特殊变量。

```html
<ul data-bind="foreach: items">
    <li>
        <!-- $data 代表当前项 -->
        <span data-bind="text: $data"></span>
        
        <!-- $index 代表索引 -->
        <span data-bind="text: '索引：' + $index()"></span>
        
        <!-- $parent 代表父级上下文 -->
        <button data-bind="click: $parent.removeItem">删除</button>
        
        <!-- $root 代表根级视图模型 -->
        <span data-bind="text: $root.title"></span>
    </li>
</ul>
```

## 模板渲染

### template 绑定

使用模板渲染内容。

```html
<!-- 定义模板 -->
<script type="text/html" id="person-template">
    <div class="person">
        <h3 data-bind="text: name"></h3>
        <p data-bind="text: bio"></p>
    </div>
</script>

<!-- 使用模板 -->
<div data-bind="template: { name: 'person-template', data: person }"></div>

<!-- 循环使用模板 -->
<div data-bind="template: { name: 'person-template', foreach: people }"></div>

<script>
var viewModel = {
    person: {
        name: "张三",
        bio: "这是一段个人简介"
    },
    people: [
        { name: "张三", bio: "简介1" },
        { name: "李四", bio: "简介2" }
    ]
};
</script>
```

## 高级用法

### 1. 嵌套循环

```html
<div data-bind="foreach: categories">
    <h2 data-bind="text: name"></h2>
    <ul data-bind="foreach: items">
        <li>
            <span data-bind="text: name"></span>
            <span data-bind="text: '(' + $parent.name + '分类)'"></span>
        </li>
    </ul>
</div>

<script>
var viewModel = {
    categories: [
        {
            name: "水果",
            items: [
                { name: "苹果" },
                { name: "香蕉" }
            ]
        },
        {
            name: "蔬菜",
            items: [
                { name: "胡萝卜" },
                { name: "白菜" }
            ]
        }
    ]
};
</script>
```

### 2. 动态模板

```html
<div data-bind="template: { 
    name: getTemplateByType,
    data: item 
}"></div>

<script type="text/html" id="text-template">
    <div class="text-item">
        <p data-bind="text: content"></p>
    </div>
</script>

<script type="text/html" id="image-template">
    <div class="image-item">
        <img data-bind="attr: { src: url, alt: description }" />
    </div>
</script>

<script>
function ViewModel() {
    this.item = {
        type: "text",
        content: "这是文本内容"
    };
    
    this.getTemplateByType = function(item) {
        return item.type + '-template';
    };
}
</script>
```

### 3. 条件组合

```html
<div>
    <!-- 多重条件 -->
    <div data-bind="if: isLoggedIn() && hasPermission()">
        <h2>管理面板</h2>
    </div>
    
    <!-- 条件嵌套 -->
    <div data-bind="if: isLoggedIn">
        <div data-bind="if: isAdmin">
            <h2>管理员面板</h2>
        </div>
        <div data-bind="ifnot: isAdmin">
            <h2>用户面板</h2>
        </div>
    </div>
</div>
```

## 最佳实践

### 1. 性能优化

```javascript
// 使用 as 绑定提高性能
<div data-bind="foreach: { data: items, as: 'item' }">
    <span data-bind="text: item.name"></span>
</div>

// 使用 afterRender 回调
<div data-bind="foreach: { 
    data: items,
    afterRender: afterItemRender
}">
    <!-- 内容 -->
</div>

function ViewModel() {
    this.afterItemRender = function(elements) {
        // 处理渲染后的元素
    };
}
```

### 2. 模板组织

```html
<!-- 将模板分组 -->
<div id="templates" style="display: none">
    <script type="text/html" id="item-template">
        <!-- 模板内容 -->
    </script>
    
    <script type="text/html" id="detail-template">
        <!-- 模板内容 -->
    </script>
</div>

<!-- 使用外部模板文件 -->
<div data-bind="template: { name: 'external-template' }"></div>
```

### 3. 上下文管理

```javascript
function ViewModel() {
    var self = this;
    
    this.items = ko.observableArray([/*...*/]);
    
    this.removeItem = function(item) {
        // 使用 self 而不是 this
        self.items.remove(item);
    };
}
```

## 常见问题

### 1. 上下文问题

```html
<!-- 错误：找不到正确的上下文 -->
<div data-bind="foreach: items">
    <button data-bind="click: removeItem">删除</button>
</div>

<!-- 正确：使用 $parent -->
<div data-bind="foreach: items">
    <button data-bind="click: $parent.removeItem">删除</button>
</div>
```

### 2. 性能问题

```javascript
// 避免在循环中创建函数
<!-- 不推荐 -->
<div data-bind="foreach: items">
    <div data-bind="click: function() { $parent.select($data) }"></div>
</div>

<!-- 推荐 -->
<div data-bind="foreach: items">
    <div data-bind="click: $parent.select"></div>
</div>
```

### 3. 模板复用

```html
<!-- 定义可复用的子模板 -->
<script type="text/html" id="user-info">
    <div class="user-info">
        <span data-bind="text: name"></span>
        <span data-bind="text: email"></span>
    </div>
</script>

<!-- 在不同地方复用 -->
<div data-bind="template: { name: 'user-info', data: currentUser }"></div>
<div data-bind="foreach: users">
    <div data-bind="template: { name: 'user-info', data: $data }"></div>
</div>
```

## 下一步

- 学习 [自定义绑定](../advanced/custom-bindings.md)
- 了解 [组件开发](../advanced/components.md)
- 探索 [性能优化](../advanced/performance.md) 