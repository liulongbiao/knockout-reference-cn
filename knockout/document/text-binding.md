# "text" 绑定

### 目的

`text` 绑定让关联的 DOM 元素可以显示你所传入的参数的文本值。

通常它用在像 `<span>` 或 `<em>` 这样传统用于显示文本的元素非常有用，但技术上你可以在任何元素上使用。

示例

```html
Today's message is: <span data-bind="text: myMessage"></span>

<script type="text/javascript">
    var viewModel = {
        myMessage: ko.observable() // Initially blank
    };
    viewModel.myMessage("Hello, world!"); // Text appears
</script>
```

### 参数

* 主参数

  Knockout 将元素的内容设置为带有你的参数值的文本节点。任何以前的文本将被重写。
  
  若该参数是一个 observable 值，该绑定将在值变更时更新元素的文本。如果参数不是 observable，
它将仅设置元素的文本一次且不再会更新它。

  如果你提供了数字或字符串以为的东西(如传入了一个对象或数组)，显示文本将等价于 `yourParameter.toString()`。

* 额外参数

   * 没有
   
### 注1：使用函数和表达式来确定文本值

如果你想编程式地决定文本，一个选择是创建一个 [computed observable](./computedObservables.md)，
并且将其求值函数作为算出显示什么文本的代码的位置。

如：

```html
The item is <span data-bind="text: priceRating"></span> today.

<script type="text/javascript">
    var viewModel = {
        price: ko.observable(24.95)
    };
    viewModel.priceRating = ko.pureComputed(function() {
        return this.price() > 50 ? "expensive" : "affordable";
    }, viewModel);
</script>
```

现在，其文本将在 `price` 变更时根据需要在 "expensive" 和 "affordable" 间切换。

或者如果你只是要像做像这样简单地事情是，你并不需要创建一个 computed observable。
你可以给 `text` 绑定传入任意 JavaScript 表达式，如：

```html
The item is <span data-bind="text: price() > 50 ? 'expensive' : 'affordable'"></span> today.
```

它们具有相同的结果，而不用引入 `priceRating` computed observable。

### 注2：关于 HTML 转码

因为该绑定使用文本节点来设置你的文本值，你可以设置任何字符串值而不用担心 HTML 或 脚本注入。
如，你写：

```javascript
viewModel.myMessage("<i>Hello, world!</i>");
```

它不会将其渲染为斜体文本，而是渲染为带可见的尖括号的文本。

如果你确实需要设置 HTML 文本，请看 [html 绑定](./html-binding.md)。

### 注3：不带容器元素使用 "text"

有时你可能希望使用 Knockout 来设置文本而不为 `text` 绑定引入一个额外的元素。
例如，你不能再 `option` 元素内部包含其他元素，因此下面代码不能工作。

```html
<select data-bind="foreach: items">
    <option>Item <span data-bind="text: name"></span></option>
</select>
```

要处理这个，你可以使用 *无容器语法*，它基于注释标签。

```html
<select data-bind="foreach: items">
    <option>Item <!--ko text: name--><!--/ko--></option>
</select>
```

其中 `<!--ko-->` 和 `<!--/ko-->` 注释用作起/止标记，定义了用于包含内部标记的 "虚拟元素"。
Knockout 理解这种虚拟元素语法并像你具有真实的容器元素一样绑定。

### 注4：关于 IE6 空白怪异行为

IE6 具有一种怪异行为，有时它会忽略空 span 的空白。
这和 Knockout 本身没有直接关系，但如果你想写：

```html
Welcome, <span data-bind="text: userName"></span> to our web site.
```

这时 IE6 在 `to our web site` 前不会显示空白，你可以通过在 `<span>` 中放置文本来避免该问题，如：

```html
Welcome, <span data-bind="text: userName">&nbsp;</span> to our web site.
```

其他浏览器和更新版本的 IE 没有这种怪异行为。

### 依赖

没有，除了核心 Knockout 类库本身。