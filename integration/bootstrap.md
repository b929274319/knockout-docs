# Bootstrap 集成指南

## 概述
本指南将帮助你在 Knockout.js 项目中集成和使用 Bootstrap，实现美观的界面和响应式布局。

## 基础配置

### 引入依赖
```html
<!-- 引入 Bootstrap CSS -->
<link href="http://cdn.bootcdn.net/ajax/libs/twitter-bootstrap/3.4.1/css/bootstrap.min.css" rel="stylesheet">

<!-- 引入 jQuery (Bootstrap 的依赖) -->
<script src="http://cdn.bootcdn.net/ajax/libs/jquery/1.12.4/jquery.min.js"></script>

<!-- 引入 Bootstrap JavaScript -->
<script src="http://cdn.bootcdn.net/ajax/libs/twitter-bootstrap/3.4.1/js/bootstrap.min.js"></script>

<!-- 引入 Knockout.js -->
<script src="http://cdn.bootcdn.net/ajax/libs/knockout/3.5.1/knockout-min.js"></script>
```

## 常见组件集成

### 模态框
```html
<div class="modal fade" id="myModal" tabindex="-1" role="dialog">
  <div class="modal-dialog" role="document">
    <div class="modal-content">
      <div class="modal-header">
        <button type="button" class="close" data-dismiss="modal"><span>&times;</span></button>
        <h4 class="modal-title" data-bind="text: modalTitle"></h4>
      </div>
      <div class="modal-body">
        <p data-bind="text: modalContent"></p>
      </div>
      <div class="modal-footer">
        <button type="button" class="btn btn-default" data-dismiss="modal">关闭</button>
        <button type="button" class="btn btn-primary" data-bind="click: saveModal">保存</button>
      </div>
    </div>
  </div>
</div>
```

```javascript
function ViewModel() {
    var self = this;
    
    self.modalTitle = ko.observable('标题');
    self.modalContent = ko.observable('内容');
    
    self.showModal = function() {
        $('#myModal').modal('show');
    };
    
    self.saveModal = function() {
        // 处理保存逻辑
        $('#myModal').modal('hide');
    };
}

ko.applyBindings(new ViewModel());
```

### 表单
```html
<form class="form-horizontal" data-bind="submit: submitForm">
  <div class="form-group">
    <label class="col-sm-2 control-label">用户名</label>
    <div class="col-sm-10">
      <input type="text" class="form-control" data-bind="value: username">
    </div>
  </div>
  <div class="form-group">
    <label class="col-sm-2 control-label">密码</label>
    <div class="col-sm-10">
      <input type="password" class="form-control" data-bind="value: password">
    </div>
  </div>
  <div class="form-group">
    <div class="col-sm-offset-2 col-sm-10">
      <button type="submit" class="btn btn-primary">提交</button>
    </div>
  </div>
</form>
```

```javascript
function FormViewModel() {
    var self = this;
    
    self.username = ko.observable('');
    self.password = ko.observable('');
    
    self.submitForm = function() {
        var data = {
            username: self.username(),
            password: self.password()
        };
        // 处理表单提交
        console.log('提交的数据:', data);
        return false; // 阻止表单默认提交
    };
}

ko.applyBindings(new FormViewModel());
```

### 表格
```html
<table class="table table-striped">
  <thead>
    <tr>
      <th>ID</th>
      <th>名称</th>
      <th>操作</th>
    </tr>
  </thead>
  <tbody data-bind="foreach: items">
    <tr>
      <td data-bind="text: id"></td>
      <td data-bind="text: name"></td>
      <td>
        <button class="btn btn-xs btn-primary" data-bind="click: $parent.editItem">编辑</button>
        <button class="btn btn-xs btn-danger" data-bind="click: $parent.removeItem">删除</button>
      </td>
    </tr>
  </tbody>
</table>
```

```javascript
function TableViewModel() {
    var self = this;
    
    self.items = ko.observableArray([
        { id: 1, name: '项目1' },
        { id: 2, name: '项目2' }
    ]);
    
    self.editItem = function(item) {
        console.log('编辑项目:', item);
    };
    
    self.removeItem = function(item) {
        self.items.remove(item);
    };
}

ko.applyBindings(new TableViewModel());
```

## 自定义绑定处理器

### Tooltip 绑定
```javascript
ko.bindingHandlers.tooltip = {
    init: function(element, valueAccessor) {
        var options = valueAccessor() || {};
        $(element).tooltip(options);
    },
    update: function(element, valueAccessor) {
        var options = valueAccessor() || {};
        $(element).tooltip('destroy').tooltip(options);
    }
};
```

使用示例：
```html
<button class="btn btn-default" 
        data-bind="tooltip: { title: tooltipText, placement: 'top' }">
    悬停查看提示
</button>
```

```javascript
function TooltipViewModel() {
    var self = this;
    self.tooltipText = ko.observable('这是一个提示信息');
}

ko.applyBindings(new TooltipViewModel());
```

### Popover 绑定
```javascript
ko.bindingHandlers.popover = {
    init: function(element, valueAccessor) {
        var options = valueAccessor() || {};
        $(element).popover(options);
    },
    update: function(element, valueAccessor) {
        var options = valueAccessor() || {};
        $(element).popover('destroy').popover(options);
    }
};
```

使用示例：
```html
<button class="btn btn-default" 
        data-bind="popover: { title: popoverTitle, content: popoverContent, trigger: 'hover' }">
    悬停查看详情
</button>
```

```javascript
function PopoverViewModel() {
    var self = this;
    self.popoverTitle = ko.observable('详细信息');
    self.popoverContent = ko.observable('这是一段详细的说明文字');
}

ko.applyBindings(new PopoverViewModel());
```

## 最佳实践

### 1. 组件化开发
将常用的 Bootstrap 组件封装为 Knockout 组件，便于复用：

```javascript
ko.components.register('bs-modal', {
    template: 
        '<div class="modal fade" tabindex="-1" role="dialog">\
            <div class="modal-dialog" role="document">\
                <div class="modal-content">\
                    <div class="modal-header">\
                        <button type="button" class="close" data-dismiss="modal">\
                            <span>&times;</span>\
                        </button>\
                        <h4 class="modal-title" data-bind="text: title"></h4>\
                    </div>\
                    <div class="modal-body" data-bind="template: { nodes: $componentTemplateNodes }"></div>\
                    <div class="modal-footer">\
                        <button type="button" class="btn btn-default" data-dismiss="modal">关闭</button>\
                        <button type="button" class="btn btn-primary" data-bind="click: save">保存</button>\
                    </div>\
                </div>\
            </div>\
        </div>',
    viewModel: function(params) {
        var self = this;
        self.title = params.title || '标题';
        self.save = params.save || function() {};
        
        self.show = function() {
            $(self.element).modal('show');
        };
        
        self.hide = function() {
            $(self.element).modal('hide');
        };
    }
});
```

使用组件：
```html
<bs-modal params="title: modalTitle, save: saveModal">
    <p data-bind="text: modalContent"></p>
</bs-modal>
```

### 2. 响应式设计
利用 Bootstrap 的栅格系统实现响应式布局：

```html
<div class="container">
    <div class="row">
        <div class="col-xs-12 col-sm-6 col-md-4" data-bind="foreach: items">
            <div class="panel panel-default">
                <div class="panel-heading" data-bind="text: title"></div>
                <div class="panel-body" data-bind="text: content"></div>
            </div>
        </div>
    </div>
</div>
```

### 3. 表单验证
结合 Bootstrap 的表单样式和 Knockout 的验证：

```javascript
ko.validation.init({
    insertMessages: false,
    decorateElement: true,
    errorElementClass: 'has-error'
});

function ValidatedFormViewModel() {
    var self = this;
    
    self.username = ko.observable().extend({
        required: true,
        minLength: 3
    });
    
    self.email = ko.observable().extend({
        required: true,
        email: true
    });
    
    self.isValid = ko.computed(function() {
        return self.username.isValid() && self.email.isValid();
    });
}
```

```html
<form class="form-horizontal" data-bind="submit: submitForm">
    <div class="form-group" data-bind="validationElement: username">
        <label class="col-sm-2 control-label">用户名</label>
        <div class="col-sm-10">
            <input type="text" class="form-control" data-bind="value: username">
            <span class="help-block" data-bind="validationMessage: username"></span>
        </div>
    </div>
</form>
```

## 常见问题

### 1. 模态框关闭后数据未清空
解决方案：在模态框的 hidden.bs.modal 事件中清空数据

```javascript
$('#myModal').on('hidden.bs.modal', function() {
    viewModel.resetModalData();
});
```

### 2. Bootstrap 组件与 Knockout 绑定冲突
解决方案：确保在正确的时机初始化 Bootstrap 插件

```javascript
ko.bindingHandlers.datepicker = {
    init: function(element, valueAccessor, allBindings, viewModel, bindingContext) {
        // 等待 Knockout 完成绑定后再初始化 datepicker
        setTimeout(function() {
            $(element).datepicker(valueAccessor());
        }, 0);
    }
};
```

### 3. 动态加载的内容未应用 Bootstrap 样式
解决方案：在内容加载完成后重新初始化 Bootstrap 插件

```javascript
self.loadContent = function() {
    // 加载内容
    self.content(newContent);
    // 重新初始化工具提示
    $('[data-toggle="tooltip"]').tooltip();
};
```

## 总结
- Bootstrap 与 Knockout.js 的集成主要涉及 UI 组件的绑定和交互
- 合理使用自定义绑定处理器可以简化代码并提高复用性
- 注意 Bootstrap 插件的初始化时机和生命周期管理
- 保持良好的组件化开发习惯，提高代码的可维护性 