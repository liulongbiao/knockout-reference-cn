# "visible" 绑定

### 目的

`visible` 绑定让关联的 DOM 元素可以根据你所传入的被绑定的值而隐藏或显示。

示例

```html
<div data-bind="visible: shouldShowMessage">
    You will see this message only when "shouldShowMessage" holds a true value.
</div>

<script type="text/javascript">
    var viewModel = {
        shouldShowMessage: ko.observable(true) // Message initially visible
    };
    viewModel.shouldShowMessage(false); // ... now it's hidden
    viewModel.shouldShowMessage(true); // ... now it's visible again
</script>
```

### 参数

* 主参数

   * 当参数为 **逻辑假值** (如布尔值 `false` 或数字 `0` 或 `null` 或 `undefined`)时，该绑定
将 `yourElement.style.display` 设置为 `none` 以使得它被隐藏。它比你使用 CSS 所定义的优先级高。
   * 当参数为 **逻辑真值** (如布尔值 `true` 或非 `null` 对象或数组)，
该绑定移除 `yourElement.style.display` 的值，使得它可见。<br/>
注意任何在使用 CSS 所配置的 display 样式将会被应用 
(因此像 `display:table-row` 这样的 CSS 规则可以和该绑定很好地一起工作)。

  如果该参数是一个 observable 值，该绑定会在任何其值变更时更新元素的可见性。
如果参数不是 observable ，它仅会设置元素的可见性一次且再也不会更新它。

* 额外参数

   * 没有
   
### 注：使用函数和表达式来控制元素可见性

你也可以使用一个 JavaScript 函数或任意的 JavaScript 表达式作为参数值。
如果你这么做，KO 将会 运行你的函数/求值你的表达式 ，且将结果用来决定是否隐藏该元素。

如：

```html
<div data-bind="visible: myValues().length > 0">
    You will see this message only when 'myValues' has at least one member.
</div>

<script type="text/javascript">
    var viewModel = {
        myValues: ko.observableArray([]) // Initially empty, so message hidden
    };
    viewModel.myValues.push("some value"); // Now visible
</script>
```

### 依赖

没有，除了核心 Knockout 类库本身。