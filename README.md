# Knockout.js 中文文档

> 简单易用的 JavaScript MVVM 框架

## 什么是 Knockout.js？

Knockout.js 是一个轻量级的 JavaScript MVVM（Model-View-ViewModel）框架，它可以帮助你创建丰富的、响应式的用户界面。通过声明式绑定，你的用户界面会随着数据模型的变化而自动更新。

### 主要特性

- **声明式绑定** - 使用简洁的语法将数据模型与 UI 元素关联
- **自动 UI 刷新** - 当数据模型发生变化时，UI 会自动更新
- **依赖跟踪** - 自动检测数据模型之间的依赖关系
- **模板机制** - 支持可重用的 UI 组件
- **轻量级** - 核心库仅约 13kb（压缩后）

## 快速示例

```html
<p>姓名：<input data-bind="value: fullName" /></p>
<p>你好，<span data-bind="text: fullName"></span>！</p>

<script>
var viewModel = {
    fullName: ko.observable('张三')
};
ko.applyBindings(viewModel);
</script>
```

## 为什么选择 Knockout.js？

- **简单易学** - 基础概念少，容易掌握
- **灵活性强** - 可以与其他库/框架配合使用
- **浏览器兼容性好** - 支持所有主流浏览器
- **完善的文档和社区** - 丰富的学习资源和活跃的社区支持

## 开始使用

查看 [快速开始](guide/quickstart.md) 章节，开始你的 Knockout.js 学习之旅！

## 学习路线图

1. 首先阅读 [基本概念](guide/concepts.md) 了解 Knockout.js 的核心思想
2. 通过 [可观察对象](core/observables.md) 学习数据追踪机制
3. 掌握常用的 [数据绑定](bindings/text-appearance.md) 方式
4. 学习如何创建 [自定义绑定](advanced/custom-bindings.md) 和 [组件](advanced/components.md)

## 获取帮助

- 查看 [常见问题](faq.md)
- 参考详细的 [API 文档](api-reference.md)
- 访问 [Knockout.js 官网](http://knockoutjs.com/)

## 版权信息

本文档由 Knockout.js 中文社区维护，基于 [MIT 许可证](https://opensource.org/licenses/MIT) 发布。 