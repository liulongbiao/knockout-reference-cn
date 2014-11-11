# Observables

Knockout 建立在三个核心特性上：

1. Observables 和依赖跟踪
2. 声明式绑定
3. 模板

本页，你会学到三个中的第一个特性。但在那之前，我们需要了解一下 MVVM 模式和视图模型的概念。

## MVVM 和 View Models

Model-View-ViewModel (MVVM) 是一种用于构建 UI 的设计模式。
它描述了如果通过将其分成三个部分使得一个潜在复杂的 UI 保持简单：

* Model ： 应用存储的数据。它表示了业务领域中的对象和操作 (如可以执行金钱流转的银行账户)且它是独立于任何 UI 的。
当使用 KO 时，你通常会使用 Ajax 请求某些后端代码来读写这些存储的模型数据。
* ViewModel ：一个 UI 上数据和操作的纯代码表示。例如，在你实现一个列表编辑器时，你的视图模型可能是一个持有
一个项目列表的对象，且暴露方法来添加和移除项目。<br/>
注意，它不是 UI 本身：它没有任何按钮或显示样式的概念。它也不是持久化数据模型 - 它持有用户所操纵的未保存数据。
当使用 KO 时，你的视图模型是不具有 HTML 知识的纯 JavaScript 对象。
让视图模型以这种方式抽象让它可以保持简单，这样你可以管理更复杂的行为而不会迷失。
* View ： 一个可视化的交互式的 UI，用于表示视图模型状态。它显示视图模型上的信息，发送命令给视图模型(如用户点击按钮)
且在视图模型变更时做相应更新。<br/>
当使用 KO 时，你的视图仅是带有声明式绑定以将其链接到视图模型上的 HTML 文档。
或者，你可以使用模板来从视图模型中生成 HTML。

要在 KO 中创建视图模型，只需要声明任意的 JavaScript 对象即可，如：

```javascript
var myViewModel = {
    personName: 'Bob',
    personAge: 123
};
```

然后你可以额用声明式绑定创建该视图模型的简单视图。如，下列标签显示了 `personName` 的值：

```html
The name is <span data-bind="text: personName"></span>
```

### 激活 Knockout

`data-bind` 属性不是 HTML 原生的，但它完全是 OK 的
(它严格兼容 HTML5，且在 HTML4 中也不会导致问题，当然某些验证器会指出这是一个不认识的属性)。
但鉴于浏览器并不知道它的意义，你需要激活 Knockout ，以使其产生效果。

要激活 Knockout，在某个 `<script>` 块中添加以下代码：

```javascript
ko.applyBindings(myViewModel);
```

你可以将其放到 HTML 文档的末尾或可以放到文档顶部并封装在在 DOM-ready 处理器中，如 jQuery 的 `$` 函数。

大功告成！现在你的视图将好像你写了以下 HTML 一样显示：

```html
The name is <span>Bob</span>
```

如果你好奇 `ko.applyBindings` 的参数都做些什么：

* 第一个参数说的是你需要给它所激活的声明式绑定使用哪个视图模型对象
* 可选的，你可以传入第二个参数来定义你希望在文档的哪一部分来搜索  `data-bind` 属性。
例如 `ko.applyBindings(myViewModel, document.getElementById('someElementId'))` 。
它会限制仅在 ID 为 `someElementId` 及其后代元素中激活，这在你希望有多个视图模型且给页面中不同部分分别关联时很有用。

真的非常简单。

## Observables

好的，现在你看到了如何创建一个基础的视图模型及如何使用绑定来显示其属性。
但 KO 的一个关键好处是它在视图模型变更时自动更新你的 UI。
KO 怎么能知道你的视图模型的那一部分有变更呢？
答案：你需要将你的模型属性声明为可观察对象(Observables)。
它们是特殊的 JavaScript 对象，可以通知订阅者变更信息，且可以自动侦测依赖。

例如，将前述视图模型对象重写为：

```javascript
var myViewModel = {
    personName: ko.observable('Bob'),
    personAge: ko.observable(123)
};
```

你完全不需要改变视图 - 相同的 `data-bind` 语法依旧可用。
不同的是现在它能够侦测变更，且在变更时，它会自动更新视图。

### Observables 的读写

并非所有的浏览器都支持 JavaScirpt getters 和 setters (咳 IE 咳) ，因此为了兼容性起见，
`ko.observable` 对象实际上是函数。

* 要 **读取** Observable 的当前值，只需要不带参数地调用该 Observable 即可。
本例中， `myViewModel.personName()` 会返回 `'bob'`，而 `myViewModel.personAge()` 会返回 `123`。
* 要 **写入** 值到 Observable，调用该 Observable 并传入新值作为参数。
例如，调用 `myViewModel.personName('Mary')` 会将名字的值改变为 `'Mary'`。
* 要给模型对象的多个 Observable 属性写值，可以使用 **链式语法** 。
如 `myViewModel.personName('Mary').personAge(50)` 会将名字的值改变为 `'Mary'`，年龄改变为 `50`。

Observables 的关键是它们可以被观察，即，其它代码可以说它希望在它们变更时得到通知。
这也是很多 KO 内建绑定内部所做的事情。
因此当你书写 `data-bind="text: personName"` 时，`text` 绑定将其本身注册为需在 `personName` 变更时
被通知到(假设它是一个 observable 值，就像它现在一样)。

当你通过调用 `myViewModel.personName('Mary')` 将名称值改变为 `'Mary'` 时，`text` 绑定
将自动更新关联 DOM 元素的文本值。
这也是视图模型的变更如何传播到视图的方式。

### 显式订阅 Observables

*你通常不需要手动设置订阅，因此初学者可以跳过本节。*

对高阶用户，如果你希望注册自己的订阅，以在 Observables 变更时得到通知，你可以调用它们的 `subscribe` 函数，如：

```javascript
myViewModel.personName.subscribe(function(newValue) {
    alert("The person's new name is " + newValue);
});
```

`subscribe` 函数就是 KO 中很多部分内部运作的方式。多数时候你不需要使用它，因为内建的绑定和模板系统会注意订阅的管理。

`subscribe` 函数接收三个参数：`callback` 是当通知发生时需调用的函数，`target` (可选)定义了回调函数中 `this`的值，
还有一个 `event` (可选，默认为 `"change"`)是需接收通知的事件名称。

如果你需要，你也可以终止一个订阅：首先将返回至捕获为一个变量，然后调用其 `dispose` 函数，如：

```javascript
var subscription = myViewModel.personName.subscribe(function(newValue) { /* do stuff */ });
// ...then later...
subscription.dispose(); // I no longer want notifications
```

如果你希望在 Observable 将要变更前被通知到，你可以订阅其 `beforeChange` 事件，如：

```javascript
myViewModel.personName.subscribe(function(oldValue) {
    alert("The person's previous name is " + oldValue);
}, null, "beforeChange");
```

> 注：Knockout 不会确保 `beforeChange` 和 `change` 会成对出现，因为你的代码的其它部分都可能独立地触发其中任意一个事件。
如果你需要跟踪 Observable 的前一次的值，你要自行使用订阅来捕获和跟踪它。

### 强制 Observables 总是通知订阅者

当写入值到一个包含原生值(数字、字符串、布尔值或 `null`)的 Observable 时，
该 Observable 的依赖通常仅会在值确实改变时才通知其订阅者。
然而，可以使用内建的 `notify` extender 来确保其订阅者总是在写入发生时得到通知，即使值没有改变。
你可以像下面这样应用这个 extender ：

```javascript
myViewModel.personName.extend({ notify: 'always' });
```

### 延迟并/或挂起变更通知

通常，Observable 会在变更发生时立即通知其订阅者。
当如果一个 Observable 频繁地变更或者会触发较昂贵的更新，你可以通过限制或延迟 Observable 的变更通知
来取得更好的性能。这可以通过使用 `rateLimit` extender 来完成，如：

```javascript
// Ensure it notifies about changes no more than once per 50-millisecond period
myViewModel.personName.extend({ rateLimit: 50 });
```