# Knockout.js 数据可视化

本文档展示如何使用 Knockout.js 结合常用图表库实现数据可视化功能。

## 与 ECharts 集成

### 1. 基本设置

```html
<!DOCTYPE html>
<html>
<head>
    <title>Knockout.js 数据可视化示例</title>
    <script src="https://cdn.jsdelivr.net/npm/knockout@3.5.1/build/output/knockout-latest.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/echarts@5.4.3/dist/echarts.min.js"></script>
</head>
<body>
    <div id="app">
        <!-- 图表容器 -->
        <div class="chart-container">
            <div id="lineChart" style="width: 600px; height: 400px;"></div>
            <div id="pieChart" style="width: 600px; height: 400px;"></div>
        </div>

        <!-- 数据控制面板 -->
        <div class="control-panel">
            <div class="form-group">
                <label>数据更新间隔（秒）</label>
                <input type="number" data-bind="value: updateInterval">
            </div>
            <button data-bind="click: toggleUpdate">
                <span data-bind="text: isUpdating() ? '停止更新' : '开始更新'"></span>
            </button>
        </div>
    </div>
</body>
</html>
```

### 2. ViewModel 实现

```javascript
class ChartViewModel {
    constructor() {
        // 基础数据
        this.updateInterval = ko.observable(5);
        this.isUpdating = ko.observable(false);
        this.updateTimer = null;
        
        // 图表实例
        this.lineChart = null;
        this.pieChart = null;
        
        // 数据源
        this.lineData = ko.observableArray([]);
        this.pieData = ko.observableArray([]);
        
        // 初始化
        this.initCharts();
        this.loadInitialData();
        
        // 订阅数据变化
        this.lineData.subscribe(() => this.updateLineChart());
        this.pieData.subscribe(() => this.updatePieChart());
    }
    
    // 初始化图表
    initCharts() {
        // 初始化折线图
        this.lineChart = echarts.init(document.getElementById('lineChart'));
        this.lineChart.setOption({
            title: {
                text: '销售趋势'
            },
            tooltip: {
                trigger: 'axis'
            },
            xAxis: {
                type: 'category',
                data: []
            },
            yAxis: {
                type: 'value'
            },
            series: [{
                type: 'line',
                data: []
            }]
        });
        
        // 初始化饼图
        this.pieChart = echarts.init(document.getElementById('pieChart'));
        this.pieChart.setOption({
            title: {
                text: '销售分布'
            },
            tooltip: {
                trigger: 'item'
            },
            series: [{
                type: 'pie',
                radius: '50%',
                data: []
            }]
        });
        
        // 响应窗口大小变化
        window.addEventListener('resize', () => {
            this.lineChart.resize();
            this.pieChart.resize();
        });
    }
    
    // 加载初始数据
    loadInitialData() {
        // 模拟初始数据
        const initialLineData = Array.from({length: 7}, (_, i) => ({
            date: new Date(Date.now() - (6 - i) * 24 * 3600 * 1000).toLocaleDateString(),
            value: Math.floor(Math.random() * 1000)
        }));
        
        const initialPieData = [
            { name: '产品A', value: Math.floor(Math.random() * 1000) },
            { name: '产品B', value: Math.floor(Math.random() * 1000) },
            { name: '产品C', value: Math.floor(Math.random() * 1000) },
            { name: '产品D', value: Math.floor(Math.random() * 1000) }
        ];
        
        this.lineData(initialLineData);
        this.pieData(initialPieData);
    }
    
    // 更新图表数据
    updateLineChart() {
        const data = this.lineData();
        this.lineChart.setOption({
            xAxis: {
                data: data.map(item => item.date)
            },
            series: [{
                data: data.map(item => item.value)
            }]
        });
    }
    
    updatePieChart() {
        const data = this.pieData();
        this.pieChart.setOption({
            series: [{
                data: data
            }]
        });
    }
    
    // 模拟数据更新
    updateData() {
        // 更新折线图数据
        const newLineData = [...this.lineData()];
        newLineData.shift();
        newLineData.push({
            date: new Date().toLocaleDateString(),
            value: Math.floor(Math.random() * 1000)
        });
        this.lineData(newLineData);
        
        // 更新饼图数据
        const newPieData = this.pieData().map(item => ({
            name: item.name,
            value: Math.floor(Math.random() * 1000)
        }));
        this.pieData(newPieData);
    }
    
    // 控制自动更新
    toggleUpdate() {
        if (this.isUpdating()) {
            clearInterval(this.updateTimer);
            this.isUpdating(false);
        } else {
            this.updateTimer = setInterval(() => {
                this.updateData();
            }, this.updateInterval() * 1000);
            this.isUpdating(true);
        }
    }
    
    // 清理方法
    dispose() {
        if (this.updateTimer) {
            clearInterval(this.updateTimer);
        }
        if (this.lineChart) {
            this.lineChart.dispose();
        }
        if (this.pieChart) {
            this.pieChart.dispose();
        }
    }
}

// 应用绑定
const viewModel = new ChartViewModel();
ko.applyBindings(viewModel);

// 页面卸载时清理
window.addEventListener('unload', () => viewModel.dispose());
```

### 3. 样式定义

```css
.chart-container {
    display: flex;
    flex-wrap: wrap;
    gap: 20px;
    margin-bottom: 20px;
}

.control-panel {
    padding: 20px;
    background-color: #f5f5f5;
    border-radius: 4px;
}

.form-group {
    margin-bottom: 15px;
}

.form-group label {
    display: block;
    margin-bottom: 5px;
}

.form-group input {
    padding: 8px;
    border: 1px solid #ddd;
    border-radius: 4px;
    width: 100px;
}

button {
    padding: 8px 16px;
    background-color: #42b983;
    color: white;
    border: none;
    border-radius: 4px;
    cursor: pointer;
}

button:hover {
    background-color: #3aa876;
}
```

## 与 D3.js 集成

### 1. 基本设置

```html
<script src="https://d3js.org/d3.v7.min.js"></script>

<div id="d3Chart" style="width: 600px; height: 400px;"></div>
```

### 2. 实现示例

```javascript
class D3ChartViewModel {
    constructor() {
        this.data = ko.observableArray([]);
        this.svg = null;
        this.initChart();
        this.loadData();
        
        // 订阅数据变化
        this.data.subscribe(() => this.updateChart());
    }
    
    initChart() {
        const width = 600;
        const height = 400;
        const margin = {top: 20, right: 20, bottom: 30, left: 40};
        
        this.svg = d3.select('#d3Chart')
            .append('svg')
            .attr('width', width)
            .attr('height', height);
            
        // 添加坐标轴等基本元素
        // ...
    }
    
    updateChart() {
        // 更新图表逻辑
        // ...
    }
}
```

## 注意事项

1. **性能优化**
   - 避免频繁更新
   - 使用防抖和节流
   - 及时清理图表实例

2. **响应式设计**
   - 监听窗口大小变化
   - 自适应容器大小
   - 移动端优化

3. **数据处理**
   - 数据格式转换
   - 异常值处理
   - 大数据集优化

4. **用户体验**
   - 添加加载状态
   - 提供交互反馈
   - 支持自定义配置 