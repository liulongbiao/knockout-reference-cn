# 自定义清理逻辑

在典型的 Knockout 应用中， DOM 元素会被动态地添加和移除，如使用 [templdate 绑定](./templdate-binding.md)
或通过控制流绑定(`if, ifnot, with, foreach`)。
当创建自定义绑定时，常需要添加当你的自定义绑定所关联的元素被 Knockout 所移除时
虚运行的清理逻辑。

### 在元素清理时注册回调

要注册在节点被移除时虚运行的函数，你可以调用 `ko.utils.domNodeDisposal.addDisposeCallback(node, callback)`。
例如，假设你创建一个自定义绑定来实例化一个 widget。
当其绑定的元素被移除时，你可能想调用 widget 上的 `destroy` 方法：

```javascript
ko.bindingHandlers.myWidget = {
    init: function(element, valueAccessor) {
        var options = ko.unwrap(valueAccessor()),
            $el = $(element);
 
        $el.myWidget(options);
 
        ko.utils.domNodeDisposal.addDisposeCallback(element, function() {
            // This will be called when the element is removed by Knockout or
            // if some other part of your code calls ko.removeNode(element)
            $el.myWidget("destroy");
        });
    }
};
```

### 重写外部数据的清理

当移除元素时， Knockout 运行逻辑来清理任何该元素关联的数据。
作为该逻辑的一部分， Knockout 在 jQuery 在被页面加载时会调用 jQuery 的 `cleanData` 方法。
高级场景中，你可能想阻止或定制数据该如何从应用中移除。
Knockout 暴露了函数 `ko.utils.domNodeDisposal.cleanExternalData(node)`，
它可被重写以支持自定义逻辑。
例如，要避免 `cleanData` 被调用，可以提供一个空函数来替换标准的 
`cleanExternalData` 实现：

```javascript
ko.utils.domNodeDisposal.cleanExternalData = function () {
    // Do nothing. Now any jQuery data associated with elements will
    // not be cleaned up when the elements are removed from the DOM.
};
```