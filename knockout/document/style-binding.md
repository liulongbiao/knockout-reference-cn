# "style" 绑定

### 目的

`style` 绑定给关联的 DOM 元素添加或移除一或多个样式值。
这在像要在某些值变成负值时以红色高亮或设置进度条的宽度以匹配数字的变更时非常有用。

注：如果你不想直接设置样式值而是想应用某个 CSS 类，查看 [style 绑定](./css-binding.md) 。

示例

```html
<div data-bind="style: { color: currentProfit() < 0 ? 'red' : 'black' }">
   Profit Information
</div>
 
<script type="text/javascript">
    var viewModel = {
        currentProfit: ko.observable(150000) // Positive value, so initially black
    };
    viewModel.currentProfit(-50); // Causes the DIV's contents to go red
</script>
```

它会在 `currentProfit` 值小于 0 时将元素的 `style.color` 设为 `red`，而在大于 0 时设为 `black`。

### 参数

* 主参数

  你应该传入一个 JavaScript 对象，其中属性名对应于样式名，而值对应于你希望设置的样式值。
  
  你可以一次性设置多个样式值。比如所你的视图模型中有个名为 `isSevere` 的属性。
  
```html
<div data-bind="style: { color: currentProfit() < 0 ? 'red' : 'black', fontWeight: isSevere() ? 'bold' : '' }">...</div>
```
  
  如果你的参数引用一个 observable 值，该绑定会在 observable 值变更时更新样式值。
如果参数没有引用一个 observable 值，它仅会更新样式值一次，后续不再会更新。

  像平常一样，你可以使用任意 JavaScript 表达式或函数作为参数值。
KO 将求值它们并使用结果值类决定应设置的样式值。
  
* 额外参数

   * 没有
   
### 注：应用非合法 JavaScript 变量名的 CSS 类名

如果你想应用 `font-weight` 或 `text-decoration` 样式，或其它名称不是合法 JavaScript 标识符的样式
(如包含中划线)，你必需使用该样式的 *JavaScript 名称*。例如：

* 不要写 `{ font-weight: someValue }`， 而要写 `{ fontWeight: someValue }`
* 不要写 `{ text-decoration: someValue }`， 而要写 `{ textDecoration: someValue }`

更多 [样式名及其 JavaScript 等价名称列表](http://www.comptechdoc.org/independent/web/cgi/javamanual/javastyle.html)。

### 依赖

没有，除了核心 Knockout 类库本身。