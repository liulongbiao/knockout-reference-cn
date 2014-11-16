# "css" 绑定

### 目的

`css` 绑定会添加一或多个具名的 CSS 类到关联的 DOM 元素上。
这很有用，如，在某些值变成负值时高亮它们。

注：如果你不想应用 CSS 类而希望直接赋予一个 `style` 属性，查看 [style 绑定](./style-binding.md) 。

静态类的示例

```html
<div data-bind="css: { profitWarning: currentProfit() < 0 }">
   Profit Information
</div>
 
<script type="text/javascript">
    var viewModel = {
        currentProfit: ko.observable(150000) // Positive value, so initially we don't apply the "profitWarning" class
    };
    viewModel.currentProfit(-50); // Causes the "profitWarning" class to be applied
</script>
```

它会在 `currentProfit` 值小于 0 时应用 `profitWarning`  CSS 类，
且在它大于 0 时移除该类。

动态类示例

```html
<div data-bind="css: profitStatus">
   Profit Information
</div>
 
<script type="text/javascript">
    var viewModel = {
        currentProfit: ko.observable(150000)
    };
 
    // Evalutes to a positive value, so initially we apply the "profitPositive" class
    viewModel.profitStatus = ko.pureComputed(function() {
        return this.currentProfit() < 0 ? "profitWarning" : "profitPositive";
    }, viewModel);
 
    // Causes the "profitPositive" class to be removed and "profitWarning" class to be added
    viewModel.currentProfit(-50);
</script>
```

它将会在 `currentProfit` 值为正时应用 `profitPositive` CSS 类，否则应用 `profitWarning` CSS 类。

### 参数

* 主参数

  如果你使用静态 CSS 类名，则你可以传入一个 JavaScript 对象，其中属性名为你的 CSS 类名，
而其值根据当前使用应被应用求值为 `true` 或 `false`。

  你可以一次性设置多个 CSS 类。如你有一个名为 `isSevere` 的属性，
  
```html
<div data-bind="css: { profitWarning: currentProfit() < 0, majorHighlight: isSevere }">
```

  你甚至可以通过将名称以引号括起来基于相同的条件设置多个 CSS 类，如：

```html
<div data-bind="css: { profitWarning: currentProfit() < 0, 'major highlight': isSevere }">
```

  非布尔值被松散地解释为布尔值。如，`0` 和 `null` 被当作 `false`，而 `21` 和非 `null` 被当作 `true`。
  
  如果你的参数引用一个 observable 值，该绑定会在 observable 值变更时添加和移除 CSS 类。
如果参数没有引用一个 observable 值，它仅会添加或移除类一次，后续不再会这么做。

  如果你想使用动态 CSS 类名，你可以传入你想添加到元素上的相应 CSS 类名的字符串。
若参数引用一个 observable 值，则该绑定会移除任何以前添加的类并根据新值添加类。

  像平常一样，你可以使用任意 JavaScript 表达式或函数作为参数值。
KO 将求值它们并使用结果值类决定应添加或移除的合适的 CSS 类。

* 额外参数

   * 没有
   
### 注：应用非合法 JavaScript 变量名的 CSS 类名

如果你想应用 CSS 类 `my-class` ，你不能写：

```html
<div data-bind="css: { my-class: someValue }">...</div>
```

因为 `my-class` 在这里不是合法的标识符。解决方案很简单：只要将标识符名用引号括起，这样它就会变成字符串字面量，
就可以再 JavaScript 对象字面量中合法使用。例如：

```html
<div data-bind="css: { 'my-class': someValue }">...</div>
```

### 依赖

没有，除了核心 Knockout 类库本身。