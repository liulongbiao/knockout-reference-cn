# 组件和自定义元素 - 概览

**组件** 是一种将你的 UI 代码组织成自包含可重用块的强大清晰的方式。它们：

* ... 可以表示杜丽丽的控件/部件，或应用的整个部分
* ... 包含它们自己的视图，以及通常(但可选)包含它们自己的视图模型
* ... 即可以被预加载，也可以通过 AMD 或其它模块系统(按需)异步加载
* ... 可以接收参数，且可选择给它们回写变更或调用回调
* ... 可以被组合(嵌套)到一起或从其它组件集成
* ... 可很容易地打包供跨项目重用
* ... 让您可以对配置和加载定义自己的约定/逻辑

这种模式对大型应用更有好处，因为它通过清晰的组织和封装来 **简化开发**，
且通过按需增量加载应用代码和模板帮助 **提高运行时性能**。

**自定义元素** 是使用组件的可选但方便地语法。
取代需要用绑定将组件注入到占位的 `<div>`，
你可以用自定义元素名来使用更具描述性的标签(如， `<voting-button>` 或 `<product-editor>`)。
Knockout 会处理像 IE6 这样旧浏览器的兼容性。

### 示例： like/dislike 部件

可以从使用 `ko.components.register` 来注册组件开始。
(技术上注册是可选的，但它是最简单地开始方式)
一个组件的定义指定了一个 `viewModel` 和 `template`。如：

```javascript
ko.components.register('like-widget', {
    viewModel: function(params) {
        // Data: value is either null, 'like', or 'dislike'
        this.chosenValue = params.value;
         
        // Behaviors
        this.like = function() { this.chosenValue('like'); }.bind(this);
        this.dislike = function() { this.chosenValue('dislike'); }.bind(this);
    },
    template:
        '<div class="like-or-dislike" data-bind="visible: !chosenValue()">\
            <button data-bind="click: like">Like it</button>\
            <button data-bind="click: dislike">Dislike it</button>\
        </div>\
        <div class="result" data-bind="visible: chosenValue">\
            You <strong data-bind="text: chosenValue"></strong> it\
        </div>'
});
```

**通常，你可以从外部文件加载视图模型和模板**，而不是像这样内联声明它们。我们可以随后了解它们。

现在，要使用这个组件，你可以在应用中从任何其他视图中引用它，
不管是使用 [component 绑定](./component-binding.md) 或使用 
[自定义元素](./component-custom-element.md)。
以下是使用自定义元素的活动示例：

源码： 视图

```html
<ul data-bind="foreach: products">
    <li class="product">
        <strong data-bind="text: name"></strong>
        <like-widget params="value: userRating"></like-widget>
    </li>
</ul>
```

源码： 视图模型

```javascript
function Product(name, rating) {
    this.name = name;
    this.userRating = ko.observable(rating || null);
}
 
function MyViewModel() {
    this.products = [
        new Product('Garlic bread'),
        new Product('Pain au chocolat'),
        new Product('Seagull spaghetti', 'like') // This one was already 'liked'
    ];
}
 
ko.applyBindings(new MyViewModel());
```

这里，该组件即显示也编辑了 `Produt` 视图模型类上称为 `userRating` 属性。

### 示例： 从外部文件按需加载 like/dislike 部件

在大多数应用中，你可能想将组件的视图模型和模板放到外部文件中。
若你配置 Knockout 通过像 [require.js](http://requirejs.org/) 这样的 AMD 模块加载器类获取它们，
则它们可以被预加载(可能被打包/最小化)，或根据需要增量加载。

以下是一个示例配置：

```javascript
ko.components.register('like-or-dislike', {
    viewModel: { require: 'files/component-like-widget' },
    template: { require: 'text!files/component-like-widget.html' }
});
```

#### 必要条件

要让它能运行，文件 
[files/component-like-widget.js](http://knockoutjs.com/documentation/files/component-like-widget.js) 
和 [files/component-like-widget.html](http://knockoutjs.com/documentation/files/component-like-widget.html)
必需存在。将它们(和 .html 中的视图源码)检出 - 如你可见，
这是一种更清晰和更方便地在定义中包含行内代码的方式。

同样，你需要引用一个合适的模块加载器类库(如 [require.js](http://requirejs.org/))
或实现一个知道如何获取你的文件的 [自定义组件加载器](./component-loaders.md)。

#### 使用该组件

现在 `like-or-dislike` 可像前面一样使用，不管通过 [component 绑定](./component-binding.md) 或
[自定义元素](./component-custom-element.md)

源码： 视图

```html
<ul data-bind="foreach: products">
    <li class="product">
        <strong data-bind="text: name"></strong>
        <like-or-dislike params="value: userRating"></like-or-dislike>
    </li>
</ul>
<button data-bind="click: addProduct">Add a product</button>
```

源码： 视图模型

```javascript
function Product(name, rating) {
    this.name = name;
    this.userRating = ko.observable(rating || null);
}
 
function MyViewModel() {
    this.products = ko.observableArray(); // Start empty
}
 
MyViewModel.prototype.addProduct = function() {
    var name = 'Product ' + (this.products().length + 1);
    this.products.push(new Product(name));
};
 
ko.applyBindings(new MyViewModel());
```

若你在首次点击 *Add product* 前打开浏览器开发者工具的 **Network** 监察面板，
你可以看到当首次 required 的时候组件的 `.js/.html` 会被按需加载，
且随后会被保留以供重用。

### 学习更多

更多细节，查看：

* [组件的定义和注册](./component-registration.md)
* [使用 component 绑定](./component-binding.md)
* [使用自定义元素](./component-custom-elements.md)
* [高阶：自定义组件加载器](./component-loaders.md)