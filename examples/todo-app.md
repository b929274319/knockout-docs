# Knockout.js Todo 应用示例

本文将通过创建一个 Todo 应用来展示 Knockout.js 的主要特性和最佳实践。

## 完整代码

### HTML 结构

```html
<!DOCTYPE html>
<html>
<head>
    <title>Knockout.js Todo App</title>
    <style>
        .completed { text-decoration: line-through; }
        .todo-item { margin: 5px 0; }
        .todo-controls { margin-top: 20px; }
    </style>
</head>
<body>
    <div id="app">
        <!-- 添加新任务 -->
        <div class="add-todo">
            <input type="text" data-bind="value: newTodoText, valueUpdate: 'afterkeydown', event: { keypress: handleEnter }">
            <button data-bind="click: addTodo">添加</button>
        </div>

        <!-- 任务列表 -->
        <ul data-bind="foreach: filteredTodos">
            <li class="todo-item">
                <input type="checkbox" data-bind="checked: completed">
                <span data-bind="text: text, css: { completed: completed }"></span>
                <button data-bind="click: $parent.removeTodo">删除</button>
            </li>
        </ul>

        <!-- 控制面板 -->
        <div class="todo-controls">
            <div>
                剩余任务: <span data-bind="text: remainingCount"></span>
            </div>
            <div>
                <label>
                    <input type="radio" name="filter" value="all" data-bind="checked: filter">
                    全部
                </label>
                <label>
                    <input type="radio" name="filter" value="active" data-bind="checked: filter">
                    未完成
                </label>
                <label>
                    <input type="radio" name="filter" value="completed" data-bind="checked: filter">
                    已完成
                </label>
            </div>
            <button data-bind="click: clearCompleted, visible: hasCompleted">清除已完成</button>
        </div>
    </div>

    <script src="https://cdnjs.cloudflare.com/ajax/libs/knockout/3.5.1/knockout-min.js"></script>
    <script src="app.js"></script>
</body>
</html>
```

### JavaScript 实现

```javascript
// app.js
class TodoViewModel {
    constructor() {
        // 可观察属性
        this.todos = ko.observableArray([]);
        this.newTodoText = ko.observable('');
        this.filter = ko.observable('all');

        // 计算属性
        this.filteredTodos = ko.computed(() => {
            const filter = this.filter();
            const todos = this.todos();

            switch (filter) {
                case 'active':
                    return todos.filter(todo => !todo.completed());
                case 'completed':
                    return todos.filter(todo => todo.completed());
                default:
                    return todos;
            }
        });

        this.remainingCount = ko.computed(() => {
            return this.todos().filter(todo => !todo.completed()).length;
        });

        this.hasCompleted = ko.computed(() => {
            return this.todos().some(todo => todo.completed());
        });

        // 从本地存储加载数据
        this.loadTodos();
    }

    // 方法
    addTodo = () => {
        const text = this.newTodoText().trim();
        if (!text) return;

        this.todos.push({
            text: ko.observable(text),
            completed: ko.observable(false)
        });

        this.newTodoText('');
        this.saveTodos();
    };

    removeTodo = (todo) => {
        this.todos.remove(todo);
        this.saveTodos();
    };

    clearCompleted = () => {
        const incompleteTodos = this.todos().filter(todo => !todo.completed());
        this.todos(incompleteTodos);
        this.saveTodos();
    };

    handleEnter = (data, event) => {
        if (event.keyCode === 13) {
            this.addTodo();
        }
        return true;
    };

    // 本地存储
    saveTodos() {
        const todosData = this.todos().map(todo => ({
            text: todo.text(),
            completed: todo.completed()
        }));
        localStorage.setItem('todos', JSON.stringify(todosData));
    }

    loadTodos() {
        const savedTodos = localStorage.getItem('todos');
        if (savedTodos) {
            const todosData = JSON.parse(savedTodos);
            const todos = todosData.map(todo => ({
                text: ko.observable(todo.text),
                completed: ko.observable(todo.completed)
            }));
            this.todos(todos);
        }
    }
}

// 初始化应用
ko.applyBindings(new TodoViewModel());
```

## 功能说明

### 1. 基本功能

- 添加新任务
- 标记任务完成状态
- 删除任务
- 显示剩余任务数量

### 2. 过滤功能

- 显示所有任务
- 只显示未完成任务
- 只显示已完成任务

### 3. 批量操作

- 清除所有已完成任务

### 4. 数据持久化

- 使用 localStorage 保存任务数据
- 页面加载时恢复任务列表

## 技术要点

### 1. 可观察对象

```javascript
// 简单可观察对象
this.newTodoText = ko.observable('');

// 可观察数组
this.todos = ko.observableArray([]);
```

### 2. 计算属性

```javascript
// 过滤后的任务列表
this.filteredTodos = ko.computed(() => {
    const filter = this.filter();
    const todos = this.todos();

    switch (filter) {
        case 'active':
            return todos.filter(todo => !todo.completed());
        case 'completed':
            return todos.filter(todo => todo.completed());
        default:
            return todos;
    }
});
```

### 3. 数据绑定

```html
<!-- 值绑定 -->
<input data-bind="value: newTodoText">

<!-- 事件绑定 -->
<button data-bind="click: addTodo">添加</button>

<!-- 列表绑定 -->
<ul data-bind="foreach: filteredTodos">
    <li>...</li>
</ul>
```

### 4. 本地存储

```javascript
// 保存数据
saveTodos() {
    const todosData = this.todos().map(todo => ({
        text: todo.text(),
        completed: todo.completed()
    }));
    localStorage.setItem('todos', JSON.stringify(todosData));
}

// 加载数据
loadTodos() {
    const savedTodos = localStorage.getItem('todos');
    if (savedTodos) {
        const todosData = JSON.parse(savedTodos);
        const todos = todosData.map(todo => ({
            text: ko.observable(todo.text),
            completed: ko.observable(todo.completed)
        }));
        this.todos(todos);
    }
}
```

## 扩展功能

### 1. 任务编辑

```javascript
// 视图模型添加
class TodoViewModel {
    // ... 其他代码

    startEditing(todo) {
        todo.editing = ko.observable(true);
        todo.editText = ko.observable(todo.text());
    }

    finishEditing(todo) {
        const newText = todo.editText().trim();
        if (newText) {
            todo.text(newText);
        }
        todo.editing(false);
        this.saveTodos();
    }
}

// HTML 模板
<span data-bind="visible: !editing(), text: text, dblclick: $parent.startEditing"></span>
<input data-bind="visible: editing, value: editText, hasFocus: editing, event: { blur: $parent.finishEditing }">
```

### 2. 任务优先级

```javascript
// 视图模型添加
class TodoViewModel {
    // ... 其他代码

    setPriority(todo, priority) {
        todo.priority(priority);
        this.saveTodos();
    }
}

// HTML 模板
<select data-bind="value: priority">
    <option value="low">低</option>
    <option value="medium">中</option>
    <option value="high">高</option>
</select>
```

### 3. 任务分类

```javascript
// 视图模型添加
class TodoViewModel {
    // ... 其他代码

    categories = ko.observableArray(['工作', '生活', '学习']);
    
    addCategory(category) {
        this.categories.push(category);
    }
}

// HTML 模板
<select data-bind="value: category, options: $parent.categories"></select>
```

## 性能优化

### 1. 使用纯计算属性

```javascript
this.remainingCount = ko.pureComputed(() => {
    return this.todos().filter(todo => !todo.completed()).length;
});
```

### 2. 避免不必要的重渲染

```javascript
// 使用 throttle
this.filteredTodos = ko.computed(() => {
    // ... 过滤逻辑
}).extend({ throttle: 100 });
```

### 3. 合理使用订阅

```javascript
// 监听变化并保存
this.todos.subscribe(this.saveTodos.bind(this));
```

## 测试

### 1. 单元测试

```javascript
describe('TodoViewModel', () => {
    let viewModel;

    beforeEach(() => {
        viewModel = new TodoViewModel();
    });

    it('should add new todo', () => {
        viewModel.newTodoText('Test Todo');
        viewModel.addTodo();

        expect(viewModel.todos().length).toBe(1);
        expect(viewModel.todos()[0].text()).toBe('Test Todo');
    });
});
```

### 2. 集成测试

```javascript
describe('Todo App Integration', () => {
    beforeEach(() => {
        document.body.innerHTML = `
            <div id="app">
                <!-- Todo App HTML -->
            </div>
        `;
        ko.applyBindings(new TodoViewModel());
    });

    it('should render todo list', () => {
        // 测试 DOM 渲染
    });
});
```

## 部署

### 1. 构建配置

```javascript
// webpack.config.js
module.exports = {
    entry: './src/app.js',
    output: {
        filename: 'bundle.js',
        path: path.resolve(__dirname, 'dist')
    },
    optimization: {
        minimize: true
    }
};
```

### 2. 发布步骤

```bash
# 安装依赖
npm install

# 运行测试
npm test

# 构建生产版本
npm run build

# 部署到服务器
npm run deploy
```

## 最佳实践

1. **代码组织**
   - 使用类组织视图模型
   - 分离业务逻辑和 UI 逻辑
   - 保持组件的单一职责

2. **性能考虑**
   - 使用纯计算属性
   - 避免深层嵌套绑定
   - 合理使用订阅

3. **用户体验**
   - 添加适当的动画效果
   - 提供即时反馈
   - 保持界面响应性

## 总结

本示例展示了：

1. Knockout.js 的核心概念
2. MVVM 模式的实践
3. 数据持久化方案
4. 性能优化技巧
5. 测试和部署策略 