# 自定义元素

自定义元素提供了方便地将 [组件](./component-overview.md) 注入到你的视图的方式。

* [简介](#introduction)
* [示例](#example)
* [传递参数](#passing-parameters)
  * [在父子组件间通信](#communication-between-parent-and-child-components)
  * [传入 observable 表达式](#passing-observable-expressions)
* [控制自定义元素标签名称](#controlling-custom-element-tag-names)
* [注册自定义元素](#registering-custom-elements)
* [注：组合自定义元素和常规绑定](#note-combining-custom-elements-with-regular-bindings)
* [注：自定义元素不能是自闭合的](#note-custom-elements-cannot-be-self-closing)
* [注：自定义元素和 IE6-8](#note-custom-elements-and-internet-explorer-6-to-8)
* [高阶：访问 $raw 参数](#advanced-accessing-raw-parameters)

### <a name="introduction"></a>简介

自定义元素时 [component 绑定](./component-binding.md) 的语法替代方案
(且实际上，自定义元素在幕后使用了 component 绑定)。

例如，除了写这个：

```html
<div data-bind='component: { name: "flight-deals", params: { from: "lhr", to: "sfo" } }'></div>
```

... 你可以写：

```html
<flight-deals params='from: "lhr", to: "sfo"'></flight-deals>
```

它允许以现代，类似 [WebCompoents](http://www.w3.org/TR/components-intro/) 的方式来组织你的代码，
但即使对旧浏览器 (见 [自定义元素和 IE6-8](#note-custom-elements-and-internet-explorer-6-to-8))
也保持支持。

### <a name="example"></a>示例

本例声明了一个组件，且给视图注入了两个实例。见以下源码。

源码： 视图

```html
<h4>First instance, without parameters</h4>
<message-editor></message-editor>
 
<h4>Second instance, passing parameters</h4>
<message-editor params='initialText: "Hello, world!"'></message-editor>
```

源码： 视图模型

```javascript
ko.components.register('message-editor', {
    viewModel: function(params) {
        this.text = ko.observable(params.initialText || '');
    },
    template: 'Message: <input data-bind="value: text" /> '
            + '(length: <span data-bind="text: text().length"></span>)'
});
 
ko.applyBindings();
```

注： 在更真实的场景中，你通常会从外部文件中加载组件视图模型和模板，
而不是在注册时硬编码它们。
详见 [一个示例](./component-overview.md#example-loading-the-likedislike-widget-from-external-files-on-demand)
和 [注册文档](./component-registration.md)。

### <a name="passing-parameters"></a>传递参数

如上例所见，你可以使用 `params` 属性给组件视图模型提供参数。
`params` 属性的内容会像 JavaScript 对象字面量来解释(就像 `data-bind` 属性)，
这样你可以传入任何类型的任意的值。如：

```html
<unrealistic-component
    params='stringValue: "hello",
            numericValue: 123,
            boolValue: true,
            objectValue: { a: 1, b: 2 },
            dateValue: new Date(),
            someModelProperty: myModelValue,
            observableSubproperty: someObservable().subprop'>
</unrealistic-component>
```

#### <a name="communication-between-parent-and-child-components"></a>在父子组件间通信

若你在 `params` 属性上引用模型属性，则你当然引用的是在组件外部的视图模型('父' 或 '宿主' 视图模型)上的属性，
因为组件本身还没有被实例化。
在上例中， `myModelValue` 将是父视图模型上的属性，且将被子组件视图模型的构造器作为
`params.someModelProperty` 所接收。

这就是你从父视图模型给子组件传递属性的方式。
若属性本身是 observable，则父视图模型将能够观察并响应
由子组件给它们插入的任何新值。

#### <a name="passing-observable-expressions"></a>传入 observable 表达式

在以下示例中，

```html
<some-component
    params='simpleExpression: 1 + 1,
            simpleObservable: myObservable,
            observableExpression: myObservable() + 1'>
</some-component>
```

... 该组件视图模型的 `params` 参数将包含三个值：

* simpleExpression

  * 它是数字值 2。它不是一个 observable 或 computed 值，因此这里不涉及任何 observables。
  
    通常，若参数的求值不涉及求值一个 observable (这里，这个值根本不涉及 observable)，
则其值会被直接传入。若该值是一个对象，则子组件可以修改它，但因为它不是 observable ，
所以父组件不会知道子组件的修改。

* simpleObservable
  
  * 它将是声明在父视图模型中为 `myObservable` 的 [ko.observable](./observables.md) 实例。
它不是一个封装 - 它和由父组件所引用的是相同的实例。
因此若子视图模型写入该 observable，父视图模型将能接收该变更。

    通常，若参数的求值不涉及求值一个 observable (这里，该 observable 被简单地传入而不求值它)，
则其值会被直接传入。

* observableExpression

  * 这个有些棘手。该表达式本身，当被求值时，会被读取为一个 observable。
该 observable 的值会随着时间变化，因此表达式会随着时间变化。

    为确保子组件可以响应表达式中值的变更， Knockout **会自动升级该参数为一个 computed 属性**。
因此，子组件将能够以 `params.observableExpression()` 来读取其当前值，
或者也可以使用 `params.observableExpression.subscribe(...)` 等等。

    通常，对自定义元素，若参数的求值涉及求值一个 observable，
则 Knockout 会自动给表达式的结果构造一个 `ko.computed` 值，且将其提供给该组件。

总结而言，其通用规则是：

1. 若参数的求值 **不涉及** 求值一个 observable/computed ，它会被直接传入
2. 若参数的求值 **涉及** 求值一或多个 observable/computed，
它会被作为一个 computed 属性传递，这样你可以响应参数值的变更。

### <a name="controlling-custom-element-tag-names"></a>控制自定义元素标签名称

默认， Knockout 假设你的自定义元素标签的名称恰好对应于使用 `ko.components.register` 所注册的组件的名称。
这种约定优于配置策略对多数应用非常理想。

若你想让有不同的自定义元素标签名，你可以覆盖 `getComponentNameForNode` 来控制它。如：

```javascript
ko.components.getComponentNameForNode = function(node) {
    var tagNameLower = node.tagName && node.tagName.toLowerCase();
 
    if (ko.components.isRegistered(tagNameLower)) {
        // If the element's name exactly matches a preregistered
        // component, use that component
        return tagNameLower;
    } else if (tagNameLower === "special-element") {
        // For the element <special-element>, use the component
        // "MySpecialComponent" (whether or not it was preregistered)
        return "MySpecialComponent";
    } else {
        // Treat anything else as not representing a component
        return null;
    }
}
```

比如说，若你想控制注册组件的哪些子集可用作自定义元素，可以使用该技术，

### <a name="registering-custom-elements"></a>注册自定义元素

若你在使用默认组件加载器，且以后使用 `ko.components.register` 来注册你的组件，
则你就不需要做额外的事情了。以这种方式注册的组件对用作自定义元素也立即可用。

若你已经实现了一个 [自定义组件加载器](./component-loaders.md)，且没有使用 `ko.components.register`，
则你需要告诉 Knockout 你希望用作自定义元素的任何元素名称。
要这么做，简单地调用 `ko.components.register` - 你不需要指点任何配置，
因为你的自定义组件加载器不能以任何方式使用该配置。如：

```javascript
ko.components.register('my-custom-element', { /* No config needed */ });
```

或者，你可以 [重载 getComponentNameForNode](#controlling-custom-element-tag-names)
来动态控制哪些元素映射到哪些组件名称，和预注册独立的。

### <a name="note-combining-custom-elements-with-regular-bindings"></a>注：组合自定义元素和常规绑定

如果需要，一个自定义元素可以有个常规的 `data-bind` 属性 (除了任何 `params` 属性外)。如：

```html
<products-list params='category: chosenCategory'
               data-bind='visible: shouldShowProducts'>
</products-list>
```

然而，使用想 [text](./text-binding.md) 或 [template](./template-binding.md) 
这样能够修改元素内容的绑定没有任何意义，因为它们重写由你的组件所注入的模板。

Knockout 将避免使用任何使用了 [controlsDescendantBindings](./custom-bindings-controlling-descendant-bindings.md)
的绑定，因为它会和试图给注入的模板绑定其视图模型的组件相冲突。
因此若你想使用像 `if` 或 `foreach` 这样的控制流绑定时，你需要在你的自定义元素外封装它，
而不是直接在自定义元素上使用它，如：

```html
<!-- ko if: someCondition -->
    <products-list></products-list>
<!-- /ko -->
```

或：

```html
<ul data-bind='foreach: allProducts'>
    <product-details params='product: $data'></product-details>
</ul>
```

### <a name="note-custom-elements-cannot-be-self-closing"></a>注：自定义元素不能是自闭合的

你必须写 `<my-custom-element></my-custom-element>` ，而 **不是** `<my-custom-element />`。
否则，你的自定义元素是非闭合的，且后续元素将被解析为子元素。

这是 HTML 标准的限制且属于 Knockout 所能控制的范围之外。
HTML 解析器，遵循 HTML 标准， [会忽略任何自闭合斜杠](http://dev.w3.org/html5/spec-author-view/syntax.html#syntax-start-tag)
(除了少量特殊 "异质元素"，它们被硬编码进解析器)。
HTML 和 XML 不相同。

### <a name="note-custom-elements-and-internet-explorer-6-to-8"></a>注：自定义元素和 IE6-8

Knockout 努力省去开发者处理跨浏览器兼容性问题的痛苦，特别是那些和旧浏览器相关的问题！
即使自定义元素提供了很现代的 web 开发风格，它们在常见浏览器中也能工作：

* HTML5 时代的浏览器，包含 IE9 及更新，自动允许自定义元素而没有困难
* IE6-8 也支持自定义元素， *但仅在它们在 HTML 解析器遇到任何这些元素前注册*

IE6-8 的 HTML 解析器将抛弃任何未知的元素。为确保它不丢弃你的自定义元素，你必需做以下一种：

* 确保在 HTML 解析器遇见任何 `<your-component>` 元素前调用 `ko.components.register('your-component')`
* 或者，至少在 HTML 解析器遇见任何 `<your-component>` 元素前调用 `document.createElement('your-component')`。
你可以忽略 `createElement` 调用的结果 - 所重要的是你必须调用它

例如，若你像这样组织你的页面结构，则所有事情都很好：

```html
<!DOCTYPE html>
<html>
    <body>
        <script src='some-script-that-registers-components.js'></script>
 
        <my-custom-element></my-custom-element>
    </body>
</html>
```

若你使用 AMD，则你可能喜欢如下结构：

```html
<!DOCTYPE html>
<html>
    <body>
        <script>
            // Since the components aren't registered until the AMD module
            // loads, which is asynchronous, the following prevents IE6-8's
            // parser from discarding the custom element
            document.createElement('my-custom-element');
        </script>
 
        <script src='require.js' data-main='app/startup'></script>
 
        <my-custom-element></my-custom-element>
    </body>
</html>
```

或者如果你不喜欢 `document.createElement` 调用这样的 hackiness，
则你可以使用一个 [component 绑定](./component-binding.md) 作为你的顶级组件而不是自定义元素。
只要所有其它组件在你的 `ko.applyBindings` 调用前被注册，
它们就可以在 IE6-8 上被用作自定义元素而不会有任何问题：

```html
<!DOCTYPE html>
<html>
    <body>
        <!-- The startup module registers all other KO components before calling
             ko.applyBindings(), so they are OK as custom elements on IE6-8 -->
        <script src='require.js' data-main='app/startup'></script>
 
        <div data-bind='component: "my-custom-element"'></div>
    </body>
</html>
```

### <a name="advanced-accessing-raw-parameters"></a>高阶：访问 $raw 参数

考虑以下非常见情况，这里 `useObservable1`, `observable1`, 和 `observable2` 都是 observable：

```html
<some-component
    params='myExpr: useObservable1() ? observable1 : observable2'>
</some-component>
```

因为求值 `myExpr` 会涉及读取一个 observable (`useObservable1`)， KO 会将该参数作为
一个 computed 属性提供给组件。

然而，该 computed 属性的值本身是一个 observable。
它会导致一个怪异的场景，当读取其当前值时会涉及双解封 (即 `params.myExpr()()`)，
其中第一个括号给出了表达式的值，而第二个给出结果 observable 实例的值。

这种双解封会很难看、不方便且不直观，因此 Knockout 会自动设置生成的 computed 属性(`params.myExpr`)
为你解封它的值。即该组件可以读取 `params.myExpr()` 来获取
任何所选定的 observable (`observable1` 或 `observable2`) 的值，而不需要双解封。

在某些情况下你 *不想自动解封*，因为你想直接访问 `observable1/observable2` 实例，
你可以从 `params.$raw` 读取值。如：

```javascript
function MyComponentViewModel(params) {
    var currentObservableInstance = params.$raw.myExpr();
     
    // Now currentObservableInstance is either observable1 or observable2
    // and you would read its value with "currentObservableInstance()"
}
```

这可能是非常规场景，因此通常你并不需要使用 `$raw`。