# 绑定上下文

*绑定上下文* 是一个持有你可以在你的绑定中引用的数据的对象。
当应用绑定时， Knockout 会自动创建和管理一个绑定上下文的层级。
该层级的根级引用的是你提供给 `ko.applyBindings(viewModel)` 的 `viewModel` 参数。
则每次你使用诸如 [with](./with-binding.md) 或 [foreach](./foreach-binding.md) 这样的控制流绑定时，
它会创建一个子绑定上下文，它引用嵌套的视图模型数据。

绑定上下文提供了以下特殊属性，你可以再任何绑定中使用：

* $parent

  这是在父上下文中的视图模型对象，就是在当前上下文外部紧邻的一个。在根上下文中，它是 undefined。

```html
<h1 data-bind="text: name"></h1>
 
<div data-bind="with: manager">
    <!-- Now we're inside a nested binding context -->
    <span data-bind="text: name"></span> is the
    manager of <span data-bind="text: $parent.name"></span>
</div>
```

* $parents

  它是表示所有父级视图模型的数组：
  
  `$parents[0]` 就是当前父上下文的视图模型(即，它等同于 `$parent`)
  
  `$parents[1]` 就是祖父上下文的视图模型
  
  `$parents[2]` 就是祖祖父上下文的视图模型
  
  ... 等待
  
* $root

  它时根上下文中的主视图模型，即，最顶级父上下文。它通常是传递给 `ko.applyBindings` 的对象。
它也等价于 `$parents[$parents.length - 1]`。

* $data

  这时当前上下文中的视图模型对象。在根上下文中，`$data` 和 `$root` 是等价的。
在嵌套的绑定上下文中，该参数会被设置为当前数据项(即在 `with: person` 绑定内部，`$data` 将被设置为 `person`)。
`$data` 在你想引用视图模型本身而不是视图模型上某个熟悉时很有用。如：

```html
<ul data-bind="foreach: ['cats', 'dogs', 'fish']">
    <li>The value is <span data-bind="text: $data"></span></li>
</ul>
```

* $index (仅在 foreach 绑定中可用)

  这时由 `foreach` 绑定渲染的当前数组项的基于 0 的索引。
不像其他绑定上下文属性， `$index` 是一个 observable 和当项的索引变更时被更新(如，当项被添加或从数组中移除时)

* $parentContext

  它引用父级的绑定上下文。它和 $parent 不同，它引用的时父级的数据(而不是绑定上下文)。
这在，例如，你需要从一个内部上下文中访问外部 `foreach` 项的索引时(用例： `$parentContext.$index`) 很有用。

* $rawData

  这时当前上下文中的原始视图模型值。通常它等价于 `$data`，但若提供给 Knockout 的视图模型
被封装为一个 observable， `$data` 将是非封装的视图模型，且 `$rawData` 将是 observable 本身。

以下特殊变量在绑定中也可用，但不是绑定上下文的一部分：

* $context

  它引用当前绑定上下文对象。它在你想访问上下文的属性且它们也存在于视图模型，
或者你想传递该上下文对象给你视图模型中的帮助函数时很有用。

* $element

  这是当前绑定的 DOM 元素对象(对虚元素，它将是注释 DOM 对象)。
它在绑定需要访问当前元素的属性时很有用。如：

```html
<div id="item1" data-bind="text: $element.id"></div>
```

### 在自定义绑定中控制或修改绑定上下文

就像内建绑定 [with](./with-binding.md) 或 [foreach](./foreach-binding.md) 一样，
自定义绑定可以给其后代元素变更绑定上下文，或通过扩展绑定上下文对象来提供特殊属性。
这在 [创建控制后代绑定的自定义绑定](./custom-bindings-controlling-descendant-bindings.md) 中有细节描述。