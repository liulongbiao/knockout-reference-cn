# 创建自定义绑定

你不限于使用想 `click`, `value` 等这样的内建绑定 - 你可以创建自己的绑定。
它时如何控制 observables 和 DOM 元素交互方式，且给你大量的灵活性来
将复杂的行为以一种易用的方式来封装起来。

例如，你可以以自定义绑定的形式来创建像 grids、 tabsets 等等交互组件。
(见 [grid 示例](http://knockoutjs.com/examples/grid.html))

### 注册自己的绑定

要注册绑定，将其添加为 `ko.bindingHandlers` 的子属性：

```javascript
ko.bindingHandlers.yourBindingName = {
    init: function(element, valueAccessor, allBindings, viewModel, bindingContext) {
        // This will be called when the binding is first applied to an element
        // Set up any initial state, event handlers, etc. here
    },
    update: function(element, valueAccessor, allBindings, viewModel, bindingContext) {
        // This will be called once when the binding is first applied to an element,
        // and again whenever any observables/computeds that are accessed change
        // Update the DOM element based on the supplied values here.
    }
};
```

... 随后你可以在任何数量的 DOM 元素上使用它：

```html
<div data-bind="yourBindingName: someValue"> </div>
```

注：实际上对 `init` 和 `update` 回调你不需要两个都提供 - 你可以根据需要提供一个或另一个。

### <a name="the-update-callback"></a> "update" 回调

Knockout 将在绑定被应用到元素上时初始调用 `update` 回调并跟踪你所访问的任何依赖(observables/computeds)。
当任何这些依赖变更时， `update` 回掉将再次被调用。以下参数会被传入：

* element - 本绑定中设计的 DOM 元素
* valueAccessor - 一个你可以调用以获取该绑定中涉及的当前模型属性的 JavaScript 函数。
不带参数地调用它(即，调用`valueAccessor()`) 来获取当前模型属性值。
要简单地既能接收 observable 也能接收普通值，可以在返回值上调用 `ko.unwrap`。
* allBindings - 一个可用于访问该 DOM 元素上所有绑定模型值得 JavaScript 对象。
调用 `allBindings.get('name')` 来获取 `name` 绑定的值(如果绑定不存在则返回 undefiend);
或 `allBindings.has('name')` 来判断当前元素上 `name` 绑定是否存在。
* viewModel - 该参数在 Knockout 3.x 中被废弃。使用 `bindingContext.$data` 或 `bindingContext.$rawData`
来访问视图模型。
* bindingContext - 一个持有该元素绑定可用的 [绑定上下文](./binding-context.md) 的对象。
该对象包含了 `$parent,$parents,$root`，可用于访问该上下文先祖上所绑定的数据。

例如，你可能已经可以通过 `visible` 绑定来控制元素的可见性，但现在你想更进一步控制转换动画效果。
你想让元素根据某个 observable 的值来滑入或滑出。
你可以通过创建一个调用 jQuery 的 `slideUp/sliceDown` 函数的自定义绑定来完成：

```javascript
ko.bindingHandlers.slideVisible = {
    update: function(element, valueAccessor, allBindings) {
        // First get the latest data that we're bound to
        var value = valueAccessor();
 
        // Next, whether or not the supplied model property is observable, get its current value
        var valueUnwrapped = ko.unwrap(value);
 
        // Grab some more data from another binding property
        var duration = allBindings.get('slideDuration') || 400; // 400ms is default duration unless otherwise specified
 
        // Now manipulate the DOM element
        if (valueUnwrapped == true)
            $(element).slideDown(duration); // Make the element visible
        else
            $(element).slideUp(duration);   // Make the element invisible
    }
};
```

现在你可以像下面一样使用它：

```html
<div data-bind="slideVisible: giftWrap, slideDuration:600">You have selected the option</div>
<label><input type="checkbox" data-bind="checked: giftWrap" /> Gift wrap</label>
 
<script type="text/javascript">
    var viewModel = {
        giftWrap: ko.observable(true)
    };
    ko.applyBindings(viewModel);
</script>
```

当然，初看起来它有很多代码，但一旦你创建了自定义绑定，它们可以很容易地在很多地方被重用。

### "init" 回调

Knockout 会在你每次开始使用该绑定的 DOM 元素上时调用你的 `init` 函数。
`init` 主要有两种用处：

* 给 DOM 元素设置任何初始状态
* 注册任何事件处理器，这样，比如说当用户点击或修改 DOM 元素时，你可以改变关联的 observable 的状态。

KO 会给它传入和 [update 回调](#the-update-callback) 一样的参数集。

继续上例，你可能想 `slideVisible` 在页面首次出现时(没有任何动画滑动)就将元素设置为立即可见或不可见，
这样动画仅在用户改变模型状态时运行。你可以如下写：

```javascript
ko.bindingHandlers.slideVisible = {
    init: function(element, valueAccessor) {
        var value = ko.unwrap(valueAccessor()); // Get the current value of the current property we're bound to
        $(element).toggle(value); // jQuery will hide/show the element depending on whether "value" or true or false
    },
    update: function(element, valueAccessor, allBindings) {
        // Leave as before
    }
};
```

这意味着如果 `giftWrap` 被定义为带初始状态 `false`(即 `giftWrap: ko.observable(false)`)，
则关联的 DIV 将初始被隐藏，且将在后续用户选中框时滑入视图。

### 在 DOM 事件后修改 observables

你已经看到了如何使用 `update` 因此，当 observable 变更时，你可以更新关联的 DOM 元素。
但另一个方向的事件呢？当用户在 DOM 元素上执行了某些动作，你可能希望能更新关联的 observable 值。

你可以使用 `init` 回调作为注册事件处理器以导致对关联 observable 的变更的地方。如：

```javascript
ko.bindingHandlers.hasFocus = {
    init: function(element, valueAccessor) {
        $(element).focus(function() {
            var value = valueAccessor();
            value(true);
        });
        $(element).blur(function() {
            var value = valueAccessor();
            value(false);
        });
    },
    update: function(element, valueAccessor) {
        var value = valueAccessor();
        if (ko.unwrap(value))
            element.focus();
        else
            element.blur();
    }
};
```

现在你通过给它绑定一个 observable 既可以读也可以写元素的 "focusedness"：

```html
<p>Name: <input data-bind="hasFocus: editingName" /></p>
 
<!-- Showing that we can both read and write the focus state -->
<div data-bind="visible: editingName">You're editing the name</div>
<button data-bind="enable: !editingName(), click:function() { editingName(true) }">Edit name</button>
 
<script type="text/javascript">
    var viewModel = {
        editingName: ko.observable()
    };
    ko.applyBindings(viewModel);
</script>
```

### 注：支持虚元素

如果你想自定义绑定能够通过 Knockout 的 *虚元素语法* 来使用，如：

```html
<!-- ko mybinding: somedata --> ... <!-- /ko -->
```

... 那可以查看 [虚元素的文档](./custom-bindings-for-virtual-elements.md)