# data-bind 语法

Knockout 的声明式绑定系统提供了简洁而强大的方式将数据链到 UI 上。
通常绑定简单数据属性或使用单个绑定很容易且显而易见。
对更多复杂绑定，它能帮助更好地理解行为和 Knockout 绑定系统的语法。

### 绑定语法

一个绑定由两个部分组成，绑定 *名称* 和 *值*， 由冒号分隔。以下是单个简单绑定的示例：

```html
Today's message is: <span data-bind="text: myMessage"></span>
```

一个元素可包含多个绑定(相关或不相关)，其中每个绑定由逗号分隔。如下示例：

```html
<!-- related bindings: valueUpdate is a parameter for value -->
Your value: <input data-bind="value: someValue, valueUpdate: 'afterkeydown'" />
 
<!-- unrelated bindings -->
Cellphone: <input data-bind="value: cellphoneNumber, enable: hasCellphone" />
```

绑定 *名称* 通常应该匹配一个注册的绑定处理器(不管是内建的还是 [自定义的](./custom-bindings.md))
或是成为另一个绑定的参数。
若名称不匹配其中任何一个， Knockout 将忽略它 (而没有任何错误或警告)。
因此如果绑定看起来没有工作，首先检查名称是否正确。

#### 绑定值

绑定的 *值* 可以是单个 [值、变量或字面量](https://developer.mozilla.org/en-US/docs/JavaScript/Guide/Values,_variables,_and_literals)
或几乎任何有效的 [JavaScript 表达式](https://developer.mozilla.org/en-US/docs/JavaScript/Guide/Expressions_and_Operators)。
以下是多种绑定值的示例：

```html
<!-- variable (usually a property of the current view model -->
<div data-bind="visible: shouldShowMessage">...</div>
 
<!-- comparison and conditional -->
The item is <span data-bind="text: price() > 50 ? 'expensive' : 'cheap'"></span>.
 
<!-- function call and comparison -->
<button data-bind="enable: parseAreaCode(cellphoneNumber()) != '555'">...</button>
 
<!-- function expression -->
<div data-bind="click: function (data) { myFunction('param1', data) }">...</div>
 
<!-- object literal (with unquoted and quoted property names) -->
<div data-bind="with: {emotion: 'happy', 'facial-expression': 'smile'}">...</div>
```

这些例子显示了值可以值可以是任何 JavaScript 表达式。
甚至当它由花括号、中括号或括号括起时逗号也是可行的。
当值是一个对象字面量时，对象的属性名必需时有效地 JavaScript 标识符或以引号括起。
若绑定值是一个非有效表达式或引用任何未知变量， Knockout 将输出一个错误并停止处理绑定。

#### 空白

绑定可以包含任意数量的 *空白* (空格、制表符和换行符)，因此你可以任意使用它来排版你的绑定。
以下示例时等价的：

```html
<!-- no spaces -->
<select data-bind="options:availableCountries,optionsText:'countryName',value:selectedCountry,optionsCaption:'Choose...'"></select>
 
<!-- some spaces -->
<select data-bind="options : availableCountries, optionsText : 'countryName', value : selectedCountry, optionsCaption : 'Choose...'"></select>
 
<!-- spaces and newlines -->
<select data-bind="
    options: availableCountries,
    optionsText: 'countryName',
    value: selectedCountry,
    optionsCaption: 'Choose...'"></select>
```

#### 跳过绑定值

从 Knockout 3.0 开始，你可以不带值来指定绑定，它会给绑定一个 `undefined` 值。如：

```html
<span data-bind="text">Text that will be cleared when bindings are applied.</span>
```

当联合 [绑定预处理](./banding-preprocessing.md) 一起使用时，这种能力特别有用，
它可以给某个绑定赋予一个默认值。