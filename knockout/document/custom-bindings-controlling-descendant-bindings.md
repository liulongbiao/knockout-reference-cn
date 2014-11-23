# 创建控制后代绑定的自定义绑定

*注：这是一个高级的技术，通常仅在创建可重用绑定类库时会用到。在你使用 Knockout 构建应用时通常不需要它。*

默认，绑定仅会影响它们所应用的元素上。
但如果你也想影响所有后代元素呢？这也是可行的。
你的绑定可以告诉 Knockout 完全 *不* 去绑定后代元素，
然后你的自定义元素可以以不同的方式来绑定它们。

要这么做，简单地在你的绑定的 `init` 函数中返回 `{ controlsDescendantBindings: true }`。

### 示例： 控制后代绑定是否被应用

对一个简单示例，这里名为 `allowBindings` 的自定义绑定仅在其值为 `true` 时允许后代绑定被应用。
若值为 `false`，则 `allowBindings` 将告诉 Knockout 它会负责给后代绑定，这样它们不会像平常一样被绑定。

```javascript
ko.bindingHandlers.allowBindings = {
    init: function(elem, valueAccessor) {
        // Let bindings proceed as normal *only if* my value is false
        var shouldAllowBindings = ko.unwrap(valueAccessor());
        return { controlsDescendantBindings: !shouldAllowBindings };
    }
};
```

要看它起效果，如下用例：

```html
<div data-bind="allowBindings: true">
    <!-- This will display Replacement, because bindings are applied -->
    <div data-bind="text: 'Replacement'">Original</div>
</div>
 
<div data-bind="allowBindings: false">
    <!-- This will display Original, because bindings are not applied -->
    <div data-bind="text: 'Replacement'">Original</div>
</div>
```

### 示例：给后代绑定提供额外的值

通常，使用了 `controlsDescendantBindings` 的绑定也会调用
`ko.applyBindingsToDescendants(someBindingContext, element)` 来给后代绑定应用
某些修改后的 [绑定上下文](./binding-context.md)。
例如，你可能有个名为 `withProperties` 的绑定给绑定上下文添加一些额外的属性，
它们可在所有后代绑定中可用：

```javascript
ko.bindingHandlers.withProperties = {
    init: function(element, valueAccessor, allBindings, viewModel, bindingContext) {
        // Make a modified binding context, with a extra properties, and apply it to descendant elements
        var innerBindingContext = bindingContext.extend(valueAccessor);
        ko.applyBindingsToDescendants(innerBindingContext, element);
 
        // Also tell KO *not* to bind the descendants itself, otherwise they will be bound twice
        return { controlsDescendantBindings: true };
    }
};
```

可以看到，绑定上下文有一个 `extend` 函数可生成带额外属性的克隆。
`extend` 函数接收一个用于拷贝属性的对象或者一个返回这种对象的函数。
函数语法是更优先的，这样绑定值上的进一步变更总是会更新绑定上下文。
这个过程不会影响原始的绑定上下文，因为影响兄弟层级的元素没有危险 - 它仅会影响后代元素。

下面是使用上述自定义绑定的示例：

```html
<div data-bind="withProperties: { emotion: 'happy' }">
    Today I feel <span data-bind="text: emotion"></span>. <!-- Displays: happy -->
</div>
<div data-bind="withProperties: { emotion: 'whimsical' }">
    Today I feel <span data-bind="text: emotion"></span>. <!-- Displays: whimsical -->
</div>
```

### 示例：在 绑定上下文层级中添加额外的层

像 [with](./with-binding.md) 和 [foreach](./foreach-binding.md) 这样的绑定
会在绑定上下文层级中创建额外的层级。
这意味着其后代可以通过使用 `$parent, $parents, $root` 或 `$parentContext` 来访问外部层级的数据。

如果你想在自定义绑定中来做，则不是使用 `bindingContext.extend()` 而是使用
`bindingContext.createChildContext(someData)` 来创建绑定上下文。
它会返回一个新的绑定上下文，其视图模型是 `someData`，且 `$parentContext` 是 `bindingContext`。
如果你想，你可以使用 `ko.utils.extend` 来进一步在子上下文上扩展其他额外属性。如：

```javascript
ko.bindingHandlers.withProperties = {
    init: function(element, valueAccessor, allBindings, viewModel, bindingContext) {
        // Make a modified binding context, with a extra properties, and apply it to descendant elements
        var childBindingContext = bindingContext.createChildContext(
            bindingContext.$rawData, 
            null, // Optionally, pass a string here as an alias for the data item in descendant contexts
            function(context) {
                ko.utils.extend(context, valueAccessor());
            });
        ko.applyBindingsToDescendants(childBindingContext, element);
 
        // Also tell KO *not* to bind the descendants itself, otherwise they will be bound twice
        return { controlsDescendantBindings: true };
    }
};
```

这个更新的 `withProperties` 绑定现在能以嵌套的方式来使用，
每个嵌套的层级能够通过 `$parentContext` 来访问父级：

```html
<div data-bind="withProperties: { displayMode: 'twoColumn' }">
    The outer display mode is <span data-bind="text: displayMode"></span>.
    <div data-bind="withProperties: { displayMode: 'doubleWidth' }">
        The inner display mode is <span data-bind="text: displayMode"></span>, but I haven't forgotten
        that the outer display mode is <span data-bind="text: $parentContext.displayMode"></span>.
    </div>
</div>
```

通过修改绑定上下文和控制后代绑定，你就有了强大而高级的工具来创建你自己的自定义绑定机制。