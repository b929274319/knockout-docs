# 可观察数组（Observable Arrays）

可观察数组是 Knockout.js 中用于处理集合数据的特殊类型，它能够追踪数组的变化并自动更新 UI。本文将详细介绍可观察数组的使用方法和高级特性。

## 基础概念

### 什么是可观察数组？

可观察数组是一个特殊的可观察对象，它包装了一个普通数组，并提供了额外的功能来监控数组的变化。

```javascript
// 创建可观察数组
var items = ko.observableArray(["项目1", "项目2", "项目3"]);

// 读取数组
console.log(items()); // 输出: ["项目1", "项目2", "项目3"]

// 修改数组
items.push("项目4"); // UI 会自动更新
```

### 为什么需要可观察数组？

1. **自动 UI 更新**
   - 数组变化时自动更新界面
   - 支持列表、表格等复杂视图

2. **丰富的操作方法**
   - 提供标准数组方法
   - 额外的便捷操作方法

3. **性能优化**
   - 智能地更新 DOM
   - 最小化重绘范围

## 基本操作

### 1. 创建和初始化

```javascript
// 空数组
var emptyArray = ko.observableArray();

// 带初始值的数组
var fruits = ko.observableArray(["苹果", "香蕉", "橙子"]);

// 对象数组
var users = ko.observableArray([
    { name: "张三", age: 25 },
    { name: "李四", age: 30 }
]);
```

### 2. 读取和修改

```javascript
// 读取整个数组
var allItems = items();

// 读取单个元素
var firstItem = items()[0];

// 修改数组
items.push("新项目");
items.pop();
items.unshift("开头新项目");
items.shift();

// 替换整个数组
items(["完全", "新的", "数组"]);
```

### 3. 数组方法

```javascript
// 标准数组方法
items.push("新项目");
items.pop();
items.unshift("首项");
items.shift();
items.reverse();
items.sort();
items.splice(1, 1, "替换项");

// Knockout 特有方法
items.remove("要删除的项");
items.removeAll();
items.destroy("要销毁的项");
```

## 高级特性

### 1. 过滤和映射

```javascript
function ViewModel() {
    var self = this;
    self.allItems = ko.observableArray([
        { id: 1, name: "项目1", active: true },
        { id: 2, name: "项目2", active: false },
        { id: 3, name: "项目3", active: true }
    ]);
    
    // 过滤活动项目
    self.activeItems = ko.computed(function() {
        return ko.utils.arrayFilter(self.allItems(), function(item) {
            return item.active;
        });
    });
    
    // 映射为简单值
    self.itemNames = ko.computed(function() {
        return ko.utils.arrayMap(self.allItems(), function(item) {
            return item.name;
        });
    });
}
```

### 2. 排序

```javascript
function ViewModel() {
    var self = this;
    self.items = ko.observableArray([
        { name: "张三", age: 25 },
        { name: "李四", age: 30 },
        { name: "王五", age: 20 }
    ]);
    
    // 按年龄排序
    self.sortByAge = function() {
        self.items.sort(function(a, b) {
            return a.age - b.age;
        });
    };
    
    // 按名字排序
    self.sortByName = function() {
        self.items.sort(function(a, b) {
            return a.name.localeCompare(b.name);
        });
    };
}
```

### 3. 分页

```javascript
function PaginatedViewModel() {
    var self = this;
    self.items = ko.observableArray(/* 大量数据 */);
    self.pageSize = ko.observable(10);
    self.currentPage = ko.observable(0);
    
    self.pagedItems = ko.computed(function() {
        var size = self.pageSize();
        var start = self.currentPage() * size;
        return self.items().slice(start, start + size);
    });
    
    self.totalPages = ko.computed(function() {
        return Math.ceil(self.items().length / self.pageSize());
    });
    
    self.nextPage = function() {
        if (self.currentPage() < self.totalPages() - 1) {
            self.currentPage(self.currentPage() + 1);
        }
    };
    
    self.previousPage = function() {
        if (self.currentPage() > 0) {
            self.currentPage(self.currentPage() - 1);
        }
    };
}
```

## 绑定用法

### 1. foreach 绑定

```html
<ul data-bind="foreach: items">
    <li>
        <span data-bind="text: name"></span>
        <button data-bind="click: $parent.removeItem">删除</button>
    </li>
</ul>

<script>
function ViewModel() {
    var self = this;
    self.items = ko.observableArray([
        { name: "项目1" },
        { name: "项目2" }
    ]);
    
    self.removeItem = function(item) {
        self.items.remove(item);
    };
}
</script>
```

### 2. 模板绑定

```html
<script type="text/html" id="item-template">
    <div class="item">
        <h3 data-bind="text: name"></h3>
        <p data-bind="text: description"></p>
    </div>
</script>

<div data-bind="template: { name: 'item-template', foreach: items }"></div>
```

## 最佳实践

### 1. 性能优化

```javascript
// 批量更新
function updateItems() {
    var self = this;
    var items = self.items();
    ko.utils.arrayForEach(newItems, function(item) {
        items.push(item);
    });
    self.items.valueHasMutated();
}

// 使用 rateLimit
var self = this;
self.filteredItems = ko.computed(function() {
    // 复杂过滤逻辑
    return ko.utils.arrayFilter(self.items(), function(item) {
        return item.isValid;
    });
}).extend({ rateLimit: 500 });
```

### 2. 数据追踪

```javascript
// 跟踪选中项
function ViewModel() {
    var self = this;
    self.items = ko.observableArray([]);
    self.selectedItems = ko.observableArray([]);
    
    self.toggleSelection = function(item) {
        if (ko.utils.arrayIndexOf(self.selectedItems(), item) > -1) {
            self.selectedItems.remove(item);
        } else {
            self.selectedItems.push(item);
        }
    };
}
```

### 3. 数组操作辅助函数

```javascript
function ArrayHelper() {
    var self = this;
    
    // 添加项目
    self.addItem = function(array, item) {
        if (array && ko.isObservable(array) && array.push) {
            array.push(item);
        }
    };
    
    // 移除项目
    self.removeItem = function(array, item) {
        if (array && ko.isObservable(array) && array.remove) {
            array.remove(item);
        }
    };
    
    // 清空数组
    self.clearArray = function(array) {
        if (array && ko.isObservable(array) && array.removeAll) {
            array.removeAll();
        }
    };
    
    // 更新数组
    self.updateArray = function(array, newItems) {
        if (array && ko.isObservable(array)) {
            array(newItems);
        }
    };
}
```

## 常见问题

### 1. 数组更新不触发 UI 更新

```javascript
// 错误方式
var items = viewModel.items();
items.push(newItem); // 不会触发 UI 更新

// 正确方式
viewModel.items.push(newItem); // 会触发 UI 更新
```

### 2. 大数组性能优化

```javascript
function LargeListViewModel() {
    var self = this;
    self.items = ko.observableArray([]);
    
    // 批量添加
    self.addItems = function(newItems) {
        var currentItems = self.items();
        ko.utils.arrayPushAll(currentItems, newItems);
        self.items.valueHasMutated();
    };
    
    // 分批处理
    self.processItems = function(items) {
        var batchSize = 100;
        var index = 0;
        
        function processBatch() {
            var batch = items.slice(index, index + batchSize);
            if (batch.length === 0) {
                return;
            }
            
            self.addItems(batch);
            index += batchSize;
            
            setTimeout(processBatch, 0);
        }
        
        processBatch();
    };
}
```

### 3. 数组元素的依赖追踪

```javascript
function ItemViewModel(data) {
    var self = this;
    self.name = ko.observable(data.name);
    self.quantity = ko.observable(data.quantity);
    
    self.total = ko.computed(function() {
        return self.quantity() * data.price;
    });
}

function ListViewModel() {
    var self = this;
    self.items = ko.observableArray([]);
    
    self.addItem = function(data) {
        self.items.push(new ItemViewModel(data));
    };
    
    self.grandTotal = ko.computed(function() {
        var total = 0;
        ko.utils.arrayForEach(self.items(), function(item) {
            total += item.total();
        });
        return total;
    });
}
``` 