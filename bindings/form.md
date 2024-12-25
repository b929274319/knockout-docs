# 表单控件绑定

本文介绍 Knockout.js 中用于处理表单控件的绑定。这些绑定可以帮助你实现表单数据的双向绑定和用户输入处理。

## 基本输入绑定

### value 绑定

用于文本输入框和文本区域。

```html
<input data-bind="value: userName" />
<textarea data-bind="value: description"></textarea>

<script>
var viewModel = {
    userName: ko.observable("张三"),
    description: ko.observable("这是一段描述")
};
</script>
```

### textInput 绑定

实时更新的文本输入绑定。

```html
<input data-bind="textInput: searchTerm" />

<script>
var viewModel = {
    searchTerm: ko.observable(""),
    // textInput 会在每次按键时更新，而不是失去焦点时
};
</script>
```

## 选择控件绑定

### checked 绑定

用于复选框和单选按钮。

```html
<!-- 单个复选框 -->
<input type="checkbox" data-bind="checked: isAgree" />
<label>我同意服务条款</label>

<!-- 多个复选框 -->
<div>
    <input type="checkbox" value="reading" data-bind="checked: hobbies" />
    <label>阅读</label>
    <input type="checkbox" value="sports" data-bind="checked: hobbies" />
    <label>运动</label>
    <input type="checkbox" value="music" data-bind="checked: hobbies" />
    <label>音乐</label>
</div>

<script>
var viewModel = {
    isAgree: ko.observable(false),
    hobbies: ko.observableArray(["reading"]) // 预选中阅读
};
</script>
```

### options 绑定

用于下拉列表和多选列表。

```html
<!-- 基本下拉列表 -->
<select data-bind="
    options: availableCountries,
    value: selectedCountry,
    optionsCaption: '请选择...'
"></select>

<!-- 使用对象数组 -->
<select data-bind="
    options: users,
    optionsText: 'name',
    optionsValue: 'id',
    value: selectedUserId
"></select>

<script>
var viewModel = {
    // 简单数组
    availableCountries: ["中国", "美国", "日本"],
    selectedCountry: ko.observable(),
    
    // 对象数组
    users: [
        { id: 1, name: "张三" },
        { id: 2, name: "李四" },
        { id: 3, name: "王五" }
    ],
    selectedUserId: ko.observable()
};
</script>
```

## 表单提交绑定

### submit 绑定

处理表单提交。

```html
<form data-bind="submit: handleSubmit">
    <input data-bind="value: userName" />
    <button type="submit">提交</button>
</form>

<script>
var viewModel = {
    userName: ko.observable(""),
    handleSubmit: function(formElement) {
        // 阻止默认提交
        // formElement 是 DOM 表单元素
        alert("提交的用户名：" + this.userName());
        return false;
    }
};
</script>
```

## 高级用法

### 1. 自定义验证

```html
<div class="form-group">
    <input data-bind="
        value: email,
        css: { error: !isEmailValid() }
    " />
    <span data-bind="visible: !isEmailValid()" class="error-message">
        请输入有效的邮箱地址
    </span>
</div>

<script>
function ViewModel() {
    this.email = ko.observable("");
    
    this.isEmailValid = ko.computed(function() {
        var email = this.email();
        return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email);
    }, this);
}
</script>
```

### 2. 动态表单字段

```html
<div data-bind="foreach: formFields">
    <div class="form-group">
        <label data-bind="text: label"></label>
        <input data-bind="
            attr: { type: type },
            value: value,
            css: { error: hasError }
        " />
    </div>
</div>

<script>
function ViewModel() {
    this.formFields = ko.observableArray([
        {
            label: "用户名",
            type: "text",
            value: ko.observable(""),
            hasError: ko.observable(false)
        },
        {
            label: "密码",
            type: "password",
            value: ko.observable(""),
            hasError: ko.observable(false)
        }
    ]);
}
</script>
```

### 3. 文件上传

```html
<input type="file" data-bind="event: { change: handleFileSelect }" />
<div data-bind="if: selectedFile">
    <p>已选择文件：<span data-bind="text: selectedFile().name"></span></p>
    <p>文件大小：<span data-bind="text: formattedFileSize"></span></p>
</div>

<script>
function ViewModel() {
    this.selectedFile = ko.observable();
    
    this.formattedFileSize = ko.computed(function() {
        var file = this.selectedFile();
        if (!file) return "";
        
        var size = file.size;
        var units = ['B', 'KB', 'MB', 'GB'];
        var i = 0;
        while (size >= 1024 && i < units.length - 1) {
            size /= 1024;
            i++;
        }
        return Math.round(size * 100) / 100 + ' ' + units[i];
    }, this);
    
    this.handleFileSelect = function(data, event) {
        var file = event.target.files[0];
        this.selectedFile(file);
    };
}
</script>
```

## 最佳实践

### 1. 表单验证

```javascript
function FormViewModel() {
    var self = this;
    
    // 表单字段
    self.username = ko.observable("").extend({
        required: true,
        minLength: 3
    });
    
    self.email = ko.observable("").extend({
        required: true,
        email: true
    });
    
    // 验证状态
    self.errors = ko.validation.group(self);
    
    // 提交处理
    self.submit = function() {
        if (self.errors().length === 0) {
            // 表单验证通过
            self.submitForm();
        } else {
            // 显示错误
            self.errors.showAllMessages();
        }
    };
}
```

### 2. 表单重置

```javascript
function FormViewModel() {
    this.formData = {
        name: ko.observable(""),
        email: ko.observable(""),
        message: ko.observable("")
    };
    
    this.resetForm = function() {
        Object.keys(this.formData).forEach(function(key) {
            this.formData[key]("");
        }, this);
    };
}
```

### 3. 防止重复提交

```javascript
function FormViewModel() {
    this.isSubmitting = ko.observable(false);
    
    this.submit = function() {
        if (this.isSubmitting()) return;
        
        this.isSubmitting(true);
        
        // 异步提交
        submitToServer()
            .then(function(response) {
                // 处理响应
            })
            .finally(function() {
                this.isSubmitting(false);
            }.bind(this));
    };
}
```

## 常见问题

### 1. 值更新时机

```html
<!-- 失去焦点时更新 -->
<input data-bind="value: name" />

<!-- 实时更新 -->
<input data-bind="textInput: name" />

<script>
var viewModel = {
    name: ko.observable("")
};
</script>
```

### 2. 选择控件默认值

```javascript
// 下拉列表默认值
function ViewModel() {
    this.countries = ["中国", "美国", "日本"];
    this.selectedCountry = ko.observable("中国");
    
    // 或者使用 optionsCaption
    this.selectedCountry = ko.observable(); // 未选择状态
}
```

### 3. 表单数据序列化

```javascript
function ViewModel() {
    this.formData = {
        name: ko.observable(""),
        email: ko.observable("")
    };
    
    this.getFormData = function() {
        var data = {};
        Object.keys(this.formData).forEach(function(key) {
            data[key] = this.formData[key]();
        }, this);
        return data;
    };
}
```

## 下一步

- 学习 [流程控制绑定](flow-control.md)
- 了解 [自定义绑定](../advanced/custom-bindings.md)
- 探索 [组件开发](../advanced/components.md) 