# "value" 绑定

### 目的

`value` 绑定将关联的 DOM 元素的值和视图模型中的某个属性关联起来。
它在像 `input`、`select` 和 `textarea` 这样的表单元素中很有用。

当用户编辑关联的表单控件时，它会更新你的视图模型的值。
类似地，当你更新视图模型的值时，它也会更新屏幕上表单控件的值。

注： 如果你在使用 checkbox 或 radio 按钮，请使用 [checked 绑定](./checked-binding.md) 来读写
元素的 checked 状态，而不是 value 绑定。

### 示例

```html
<p>Login name: <input data-bind="value: userName" /></p>
<p>Password: <input type="password" data-bind="value: userPassword" /></p>
 
<script type="text/javascript">
    var viewModel = {
        userName: ko.observable(""),        // Initially blank
        userPassword: ko.observable("abc"), // Prepopulate
    };
</script>
```

### 参数

* 主参数

  KO 设置元素的 `value` 属性为你的参数的值。任何以前的值将会被覆盖。
  
  如果你的参数是一个 observable 值，该绑定会在 observable 值变更时更新元素的值。
如果参数没有引用一个 observable 值，它仅会设置元素的值一次，后续不会再更新。

  如果你提供的参数不是数字或字符串(如传入了对象或数组)，其显示文本将等价于 `yourParameter.toString()`。
(这通常不是很有用，所以最好提供字符串或数字值)

  当用户编辑关联表单控件上的值时， KO 将会更新视图模型上的属性。
KO 在值被改变且用户转移关注到另一个 DOM 节点(即在 `change` 事件)时总是会试图更新你的视图模型，
但你可以通过下述的 `valueUpdate` 来基于其他事件触发更新。

* 额外参数

  * valueUpdate
    
	如果你的绑定也包含了一个名为 `valueUpdate` 的参数，它定义了 KO 应在 `change` 事件外用于侦测变更的
额外的浏览器事件。常用的选择有以下字符串值：

    * "input" - 当 `<input>` 或 `<textarea>` 元素的值变更时更新视图模型。注意该事件仅在现代浏览器中触发(如IE9+)
	* "keyup" - 当用户释放键时更新视图模型
	* "keypress" - 当用户敲击键时更新视图模型。不像 `keyup`，它在用户按住某个键时会持续地更新。
	* "afterkeydown" - 在用户开始键入某个字符时立即更新。它通过捕获浏览器的 `keydown` 事件并异步地处理它来完成。
它在某些移动浏览器中可能不能工作。

  * valueAllowUnset
  
    见以下 [注2](#note2) 。注意 valueAllowUnset 仅在使用 `value` 来控制 `<select>` 元素的选择项时可用。
其它元素上没有效果。

### 注1： 让值在输入时持续更新

如果你想让 `<input type="text" />` 或 `<textarea>` 持续地更新视图模型，使用 [textInput 绑定](./textinput-binding.md)。
它比任何 `valueUpdate` 选项的组合都能提供更好的浏览器边界情况的支持。

### <a name="note2"></a>注2： 用于下拉列表(即 `<select>` 元素)

Knockout 对下拉列表(即 `<select>` 元素)有特殊的支持。
`value` 绑定可连同 `options` 绑定一起工作，让你可以读写任意的 JavaScript 对象，而不只是字符串值。
这在你想让用户在一系列模型对象中选择非常有用。
其示例可见 [options 绑定](./options-binding.md) 或 [selectedOptions 绑定](./selectedOptions-binding.md)
文档中的多选列表。

你也可以再没有 `options` 绑定的 `<select>` 元素上使用 `value` 绑定。
这时，你可以选择在标签中指定你的 `<option>` 元素或者使用 `foreach` 或 `template` 绑定来构建它们。
你甚至可以再 `<optgroup>` 元素中嵌套选项，而 Knockout 将相应地设置被选项。

#### 在 `<select>` 元素上使用 `valueAllowUnset`

通常当你在 `<select>` 元素上使用 `value` 绑定时，它意味着你想关联的模型值来描述
`<select>` 中的哪一个项被选中。
但如果你设置的模型值在列表的元素中没有对应的项时会发生什么？
默认行为是 Knockout 会覆盖你的模型值将其重新设置为下拉框中已被选中的任何值，
因此会导致模型和 UI 没有同步。

然而，有时你可能不想要这个行为。
如果你希望 Knockout 允许你的模型 observable 具有在 `<select>` 中没有对应项的值，
则可以指定 `valueAllowUnset: true`。
这时，当你的模型值不能被 `<select>` 所展示时，
则 `<select>` 在这时简单地没有选中的值，其可视化表现为空白。
当用户随后从下列框中选择了一项时，它会像往常一样写到你的模型中。如：

```html
<p>
    Select a country:
    <select data-bind="options: countries,
                       optionsCaption: 'Choose one...',
                       value: selectedCountry,
                       valueAllowUnset: true"></select>
</p>
 
<script type="text/javascript">
    var viewModel = {
        countries: ['Japan', 'Bolivia', 'New Zealand'],
        selectedCountry: ko.observable('Latvia')
    };
</script>
```

上例中， `selectedCountry` 会保留至 `'Latvia'` ，而下拉框将是空白的，因为没有对应的选项。

如果没有企业 `valueAllowUnset` 的话， Knockout 将会重写 `selectedCountry` 的值为 `undefiened`，
这样它会匹配 `'Choose one...'` 提示项。

### 注3： 更新 observable 和非 observable 属性的值

如果你使用 `value` 来将表单元素链到某个 observable 属性时， KO 就能够设置双向绑定，
这样它们彼此的更新都会影响另一个。

然而，当你使用 `value` 将表单元素链到非 observable 属性(如纯字符串或任意 JavaScript 表达式)时，
KO 将会：

* 如果引用一个 *简单属性*，即它只是视图模型上的常规属性， KO 将会设置表单元素的初始状态为该属性的值，
且当表单元素被编辑时，KO 会将变更回写到你的属性上。
它不能侦测属性的变更(因为它不是 observable)，因此这仅是单向绑定。
* 当你的引用不是一个简单属性时，如函数调用或比较操作的结果，KO 将会设置表单元素的初始状态为该值，
但在用户编辑表单元素时不能回写任何变更。
这种情况下它只是一个一次性的值设置器，而不会响应任何变更。

示例：

```html
<!-- Two-way binding. Populates textbox; syncs both ways. -->
<p>First value: <input data-bind="value: firstValue" /></p>
 
<!-- One-way binding. Populates textbox; syncs only from textbox to model. -->
<p>Second value: <input data-bind="value: secondValue" /></p>
 
<!-- No binding. Populates textbox, but doesn't react to any changes. -->
<p>Third value: <input data-bind="value: secondValue.length > 8" /></p>
 
<script type="text/javascript">
    var viewModel = {
        firstValue: ko.observable("hello"), // Observable
        secondValue: "hello, again"         // Not observable
    };
</script>
```

### 注4： 连同 checked 绑定来使用 value 绑定。

[checked 绑定](./checked-binding.md) 应被用于将视图模型属性绑定到
某个 checkbox (`<input type='checkbox'>`)或 radio 按钮 (`<input type='radio'>`) 的值上。
如果你在 checked 绑定上使用了 value 绑定，则该 value 绑定简单地像可用于 `checked` 绑定的
`checkedValue` 选项一样工作，它将用于控制用于更新你的视图模型的值。

### 依赖

没有，除了核心 Knockout 类库本身。