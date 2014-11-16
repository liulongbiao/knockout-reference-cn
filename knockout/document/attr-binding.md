# "attr" 绑定

### 目的

`attr` 绑定提供了给关联的 DOM 元素设置任意属性的一种通用方式。
这在像需要根据你的视图模型中的值来设置元素的 `title` 属性或 `img` 标签的 `src` 属性或链接的 `href` 值，
且属性值会在相应模型属性变更时自动被更新。

示例

```html
<a data-bind="attr: { href: url, title: details }">
    Report
</a>
 
<script type="text/javascript">
    var viewModel = {
        url: ko.observable("year-end.html"),
        details: ko.observable("Report including final year-end statistics")
    };
</script>
```

它会将元素的 `href` 设置为 `year-end.html`，而 `title` 设置为 `Report including final year-end statistics`。

### 参数

* 主参数

  你应该传入一个 JavaScript 对象，其中属性名对应于属性名，而值对应于你希望设置的属性值。
  
  如果你的参数引用一个 observable 值，该绑定会在 observable 值变更时更新属性值。
如果参数没有引用一个 observable 值，它仅会更新属性值一次，后续不再会更新。
  
* 额外参数

   * 没有
   
### 注：应用非合法 JavaScript 变量名的 CSS 类名

如果你想应用属性值 `data-something`，你不能写：

```html
<div data-bind="attr: { data-something: someValue }">...</div>
```

因为 `data-something` 在这里不是合法的标识符名称。解决方案很简单：只要将标识符名用引号括起，这样它就会变成字符串字面量，
就可以再 JavaScript 对象字面量中合法使用。例如：

```html
<div data-bind="attr: { 'data-something': someValue }">...</div>
```

### 依赖

没有，除了核心 Knockout 类库本身。