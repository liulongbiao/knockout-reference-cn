# 使用 RequireJs 的 异步模块定义(AMD)

### AMD 概览

从 [用 AMD, CommonJS & ES Harmony 书写模块化 JavaScript](http://addyosmani.com/writing-modular-js/):

> 当我们说应用是模块化的时，我们常意味着它由一系列高度解耦的、以模块存储的功能的独立片段组成。
> 如你所知，松耦合的机制通过尽可能地移除依赖来简化应用的维护。
> 当其被高效地实现时，可以很容易看到系统的某个部分如何彼此影响。

> 然而不像其它传统的编程语言，当前的 JavaScript (ECMA-262) 并没有给开发者提供清晰、有组织的方式来
> 实现这种代码模块的方式。该规范的这一点没有得到很大关注，直到近些年更具组织的 JavaScript 应用的需求变得明显。

> 取之，现在的开发者需要回退到模块或对象字面量模式的的变体。
> 其中的很多，模块脚本和 DOM 中由单个全局对象所描述的命名空间捆在一起，
> 它还是可能在你的架构中发生命名冲突。
> 还不存在不用手动努力或第三方工具就能处理依赖管理的清晰的方式。

> 尽管这些问题的原生解决方案将在 ES Harmony 中出现，好消息是书写模块化 JavaScript 变得重未有过的容易
> 且你现在就可以开始做。

### 通过 RequireJs 加载 Knockout.js 和视图模型类

HTML

```html
<html>
    <head>
        <script type="text/javascript" data-main="scripts/init.js" src="scripts/require.js"></script>
    </head>
    <body>
        <p>First name: <input data-bind="value: firstName" /></p>
        <p>First name capitalized: <strong data-bind="text: firstNameCaps"></strong></p>
    </body>
</html>
```

scripts/init.js

```javascript
require(['knockout-x.y.z', 'appViewModel', 'domReady!'], function(ko, appViewModel) {
    ko.applyBindings(new appViewModel());
});
```

scripts/appViewModel.js

```javascript
// Main viewmodel class
define(['knockout-x.y.z'], function(ko) {
    return function appViewModel() {
        this.firstName = ko.observable('Bert');
        this.firstNameCaps = ko.pureComputed(function() {
            return this.firstName().toUpperCase();
        }, this);
    };
});
```

当然 `x.y.z` 应该被替换为你所加载的 knockout 脚本的版本号(如 knockout-3.2.0)

### 通过 RequireJs 加载 Knockout.js 、绑定处理器和视图模型类

绑定处理器的通用文档可在 [这里](./custom-bindings.md) 中找到。
本节仅想演示 AMD 提供的维护你的自定义处理器的能力。
我们将使用绑定处理器文档中的 `ko.bindingHandlers.hasFocus` 示例做演示。
通过将该处理器封装成自己的模块，你可以限制其仅在页面需要时使用。被封装的模块如：

```javascript
define(['knockout-x.y.z'], function(ko){
    ko.bindingHandlers.hasFocus = {
        init: function(element, valueAccessor) { ... },
        update: function(element, valueAccessor) { ... }
    }
});
```

定义模块后，更新输入元素的 HTML 示例变为：

```html
<p>First name: <input data-bind="value: firstName, hasFocus: editingName" /><span data-bind="visible: editingName"> You're editing the name!</span></p>
```

在你的视图模型中将该模块包含在依赖列表中：

```javascript
define(['knockout-x.y.z', 'customBindingHandlers/hasFocus'], function(ko) {
    return function appViewModel(){
        ...
        // Add an editingName observable
        this.editingName = ko.observable();
    };
});
```

注意，自定义绑定处理器模块不给你的视图模型模块注入任何东西，因为它不返回任何东西。
它只是给 knockout 模块添加额外的行为。

### RequireJs 下载

RequireJs 可从 http://requirejs.org/docs/download.html 下载。