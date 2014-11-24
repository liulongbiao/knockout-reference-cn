# 组件的注册

为 Knockout 能够加载和实例化你的组件，你必需使用 `ko.components.register` 来注册它们，
提供像这里所描述的一个配置。

> 注：作为替代，可以实现一个 [自定义组件加载器](./component-loaders.md) 来通过
> 自己的约定而不是明确地配置来获取组件。

* [以 视图模型/模板 对来注册组件](#registering-components-as-a-viewmodeltemplate-pair)
  * [指定一个视图模型](#specifying-a-viewmodel)
    * [一个构造器函数](#a-constructor-function)
    * [一个共享对象实例](#a-shared-object-instance)
    * [一个 createViewModel 工厂函数](#a-createviewmodel-factory-function)
    * [一个 AMD 模块其值描述了一个视图模型](#an-amd-module-whose-value-describes-a-viewmodel)
  * [指定一个模板](#specifying-a-template)
    * [一个已有的元素ID](#an-existing-element-id)
    * [一个已有的元素实例](#an-existing-element-instance)
    * [一个标签的字符串](#a-string-of-markup)
    * [一个 DOM 节点的数组](#an-array-of-dom-nodes)
    * [一个文档片段](#a-document-fragment)
    * [一个 AMD 模块其值描述了一个模板](#an-amd-module-whose-value-describes-a-template)
* [Knockout 如何通过 AMD 加载组件](#how-knockout-loads-components-via-amd)
  * [按需加载 AMD 模块](#amd-modules-are-loaded-only-on-demand)
* [将单个 AMD 模块注册为组件](#registering-components-as-a-single-amd-module)
  * [推荐的 AMD 模块模式](#a-recommended-amd-module-pattern)
  
### <a name="registering-components-as-a-viewmodeltemplate-pair"></a>以 视图模型/模板 对来注册组件

你可以像下面这样注册组件：

```javascript
ko.components.register('some-component-name', {
    viewModel: <see below>,
    template: <see below>
});
```

* 组件 **名称** 可以是任何非空字符串。推荐，但不是必须的，使用中划线分隔的小写字符串(如 `your-component-name`)
这样组件名称在用作 [自定义元素](./component-custom-element.md) (如 `<your-component-name>`)时也是有效的。
* `viewModel` 是可选的，但可以是任何 [以下描述的 viewModel 格式](#specifying-a-viewmodel)
* `template` 是必需的，但可以是任何 [以下描述的 template 格式](#specifying-a-template)

若没有给定视图模型，组件被当作简单地 HTML 块可被绑定到任何传给组件的参数上。

### <a name="specifying-a-viewmodel"></a>指定一个视图模型

视图模型可以以下述任何形式来指定：

#### <a name="a-constructor-function"></a>一个构造器函数

```javascript
function SomeComponentViewModel(params) {
    // 'params' is an object whose key/value pairs are the parameters
    // passed from the component binding or custom element.
    this.someProperty = params.something;
}
 
SomeComponentViewModel.prototype.doSomething = function() { ... };
 
ko.components.register('my-component', {
    viewModel: SomeComponentViewModel,
    template: ...
});
```

Knockout 将给该组件的每个实例调用你的构造器一次，给每个都生成独立地视图模型对象。
结果对象上或其原型链上的属性(如上例中的 `someProperty` 和 `doSomething`)
在组件视图的绑定中可用。

#### <a name="a-shared-object-instance"></a>一个共享对象实例

若你想你所有组件的实例都共享相同的视图模型对象实例(它通常不是令人满意的)：

```javascript
var sharedViewModelInstance = { ... };
 
ko.components.register('my-component', {
    viewModel: { instance: sharedViewModelInstance },
    template: ...
});
```

注意指定 `viewModel: { instance: object }` 而不是 `viewModel: object` 时必要的。
它和下面其它场景是不同的。

#### <a name="a-createviewmodel-factory-function"></a>一个 createViewModel 工厂函数

若你想对关联的元素在其被绑定到视图模型前运行任何设置逻辑，
或使用任意逻辑来决定实例化哪个视图模型：

```javascript
ko.components.register('my-component', {
    viewModel: {
        createViewModel: function(params, componentInfo) {
            // - 'params' is an object whose key/value pairs are the parameters
            //   passed from the component binding or custom element
            // - 'componentInfo.element' is the element the component is being
            //   injected into. When createViewModel is called, the template has
            //   already been injected into this element, but isn't yet bound.
 
            // Return the desired view model instance, e.g.:
            return new MyViewModel(params);
        }
    },
    template: ...
});
```

注意，通常最好仅通过 [自定义绑定](./custom-binding.md) 而不是通过
在 `createViewModel` 中操作 `componentInfo.element` 来执行直接 DOM 操作。
这能导致更模块化，可重用的代码。

#### <a name="an-amd-module-whose-value-describes-a-viewmodel"></a>一个 AMD 模块其值描述了一个视图模型

若你的页面中已有 AMD 加载器(如 require.js) ，则你可以用它来获取一个视图模型。
对更多它如何工作的细节，查看下面 [Knockout 如何通过 AMD 加载模块](#how-knockout-loads-components-via-amd)。
如：

```javascript
ko.components.register('my-component', {
    viewModel: { require: 'some/module/name' },
    template: ...
});
```

返回的 AMD 模块对象可以使以下视图模型允许的形式。因此它可以是一个构造器函数，如：

```javascript
// AMD module whose value is a component viewmodel constructor
define(['knockout'], function(ko) {
    function MyViewModel() {
        // ...
    }
 
    return MyViewModel;
});
```

... 或一个共享对象实例，如：

```javascript
// AMD module whose value is a shared component viewmodel instance
define(['knockout'], function(ko) {
    function MyViewModel() {
        // ...
    }
 
    return { instance: new MyViewModel() };
});
```

... 或一个 `createViewModel` 函数，如：

```javascript
// AMD module whose value is a 'createViewModel' function
define(['knockout'], function(ko) {
    function myViewModelFactory(params, componentInfo) {
        // return something
    }
     
    return { createViewModel: myViewModelFactory };
});
```

... 或者甚至，尽管你可能不会这么做，一个对 AMD 模块的引用，如：

```javascript
// AMD module whose value is a reference to a different AMD module,
// which in turn can be in any of these formats
define(['knockout'], function(ko) {
    return { module: 'some/other/module' };
});
```

### <a name="specifying-a-template"></a>指定一个模板

模板可以以下形式来指定。最通用的是 [已有的元素ID](#an-existing-element-id)
和 [AMD 模块](#an-amd-module-whose-value-describes-a-template)。

#### <a name="an-existing-element-id"></a>一个已有的元素ID

例如，以下元素：

```html
<template id='my-component-template'>
    <h1 data-bind='text: title'></h1>
    <button data-bind='click: doSomething'>Click me right now</button>
</template>
```

... 可通过指定它的 ID 来给组件用作模板：

```javascript
ko.components.register('my-component', {
    template: { element: 'my-component-template' },
    viewModel: ...
});
```

注意仅有指定元素 *内部* 的节点会被克隆到每个组件的示例。
其容器元素(这里是 `<template>` 元素)，将 *不* 被当作组件模板的一部分。

你不限于使用 `<template>` 元素，但它们很方便(在支持它们的浏览器上)，
因为它们自己不会被渲染。任何其它元素类型也能工作。

#### <a name="an-existing-element-instance"></a>一个已有的元素实例

若你的代码中有对某个 DOM 元素的引用，你可以将其用作模板标签的容器：

```javascript
var elemInstance = document.getElementById('my-component-template');
 
ko.components.register('my-component', {
    template: { element: elemInstance },
    viewModel: ...
});
```

同样，仅有指定元素 *内部* 的节点会被用作组件的模板。

#### <a name="a-string-of-markup"></a>一个标签的字符串

```javascript
ko.components.register('my-component', {
    template: '<h1 data-bind="text: title"></h1>\
               <button data-bind="click: doSomething">Clickety</button>',
    viewModel: ...
});
```

它当你从其它地方编程式地获取标签时非常有用(如，如下 AMD)，
或将构建系统输出打包组件用于发布，
因为手动编辑作为 JavaScript 字符串字面量的 HTML 不是非常方便。

#### <a name="an-array-of-dom-nodes"></a>一个 DOM 节点的数组

若你编程式地构建配置且你有一个 DOM 节点的数组，你可以将它们用作组件模板：

```javascript
var myNodes = [
    document.getElementById('first-node'),
    document.getElementById('second-node'),
    document.getElementById('third-node')
];
 
ko.components.register('my-component', {
    template: myNodes,
    viewModel: ...
});
```

这里，所有指定的节点(及其后代节点)都将被克隆并连接到每个被示例化的组件的拷贝。

#### <a name="a-document-fragment"></a>一个文档片段

若你编程式地构建配置且你有一个 `DocumentFragment` 对象，你可以将其用作组件模板：

```javascript
ko.components.register('my-component', {
    template: someDocumentFragmentInstance,
    viewModel: ...
});
```

鉴于文档片段可以有多个顶层节点，其 *整个* 文档片段(不只是顶层节点的后代)
都会被当作组件的模板。

#### <a name="an-amd-module-whose-value-describes-a-template"></a>一个 AMD 模块其值描述了一个模板

若你页面中已有一个 AMD 加载器(如 require.js)，则你可以用它获取模板。

对更多它如何工作的细节，查看下面 [Knockout 如何通过 AMD 加载模块](#how-knockout-loads-components-via-amd)。
如：

``javascript
ko.components.register('my-component', {
    template: { require: 'some/template' },
    viewModel: ...
});
```

返回的 AMD 模块对象可以是任何试图模型所允许的形式。
因此它可以是标签的字符串，如使用 [require.js 的 text 插件](http://requirejs.org/docs/api.html#text)：

```javascript
ko.components.register('my-component', {
    template: { require: 'text!path/my-html-file.html' },
    viewModel: ...
});
```

... 或任何这里描述的其它形式，尽管其它形式通过 AMD 获取模板可能不常用。

### <a name="how-knockout-loads-components-via-amd"></a>Knockout 如何通过 AMD 加载组件

当我们通过 `require` 声明来加载视图模型或模板，如：

```javascript
ko.components.register('my-component', {
    viewModel: { require: 'some/module/name' },
    template: { require: 'text!some-template.html' }
});
```

... 所有 Knockout 所做的是调用 `require(['some/module/name'], callback)`
和 `require(['text!some-template.html'], callback)`，
并将异步返回的对象用作视图模型和模板定义。因此，

* **它不会严格依赖于 [require.js](http://requirejs.org/) 或其它特定的模块加载器**。
任何提供 AMD 风格的 `require` API 的模块加载器都可以。
若你想集成一个 API 不同的模块加载器，可以实现一个 [自定义组件加载器](./component-loaders.md)
* **Knockout 不会以任何方式解析模块名称** - 它只是将其传递给 `require()`。
因此当然 Knockout 不会知道或关心你的模块文件是从哪里加载的。
它根据的时你 AMD 加载器和你如何配置它。
* **Knockout 不知道也不关心你的 AMD 模块是否是匿名的**。
通常将组件定义为匿名模块很方便，但这种关心完全和 KO 是独立地。

#### <a name="amd-modules-are-loaded-only-on-demand"></a>按需加载 AMD 模块

Knockout 不会调用 `require([moduleName], ...)` 直到你的组件被实例化时。
这是组件如何被按需加载，而不是靠前。

例如，若你的组件在某个 [if 绑定](./if-binding.md) 中(或其它控制流绑定)，
则它不会导致 AMD 模块被加载直到 `if` 条件为真。
当然，若 AMD 模块已经被加载了(如在预加载捆中)，
则 `require` 调用不会触发任何额外 HTTP 请求，因此你可以控制哪些被预加载，哪些被按需加载。

### <a name="registering-components-as-a-single-amd-module"></a>将单个 AMD 模块注册为组件

对更好的封装，你可以将组件打包为单个自描述的 AMD 模块。
然后你可以简单地将其作为组件来引用：

```javascript
ko.components.register('my-component', { require: 'some/module' });
```

注意它没有指定 视图模型/模板 对。 AMD 模块本身可以提供 视图模型/模板 对，
使用任何上面列的定义格式。例如，文件 `some/module.js` 可声明为：

```javascript
// AMD module 'some/module.js' encapsulating the configuration for a component
define(['knockout'], function(ko) {
    function MyComponentViewModel(params) {
        this.personName = ko.observable(params.name);
    }
 
    return {
        viewModel: MyComponentViewModel,
        template: 'The name is <strong data-bind="text: personName"></strong>'
    };
});
```

#### <a name="a-recommended-amd-module-pattern"></a>推荐的 AMD 模块模式

实践中最有用的时创建具有内联视图模型类的 AMD 模块，
且明确接收外部模板文件的 AMD 依赖。

例如，若下面是文件 `path/my-component.js`，

```javascript
// Recommended AMD module pattern for a Knockout component that:
//  - Can be referenced with just a single 'require' declaration
//  - Can be included in a bundle using the r.js optimizer
define(['knockout', 'text!./my-component.html'], function(ko, htmlString) {
    function MyComponentViewModel(params) {
        // Set up properties, etc.
    }
 
    // Use prototype to declare any public methods
    MyComponentViewModel.prototype.doSomething = function() { ... };
 
    // Return component definition
    return { viewModel: MyComponentViewModel, template: htmlString };
});
```

... 且模板标签在文件 `path/my-component.html`，你可以有以下好处：

* 应用可以很容易地引用它，即 `ko.components.register('my-component', { require: 'path/my-component' })`
* 对一个组件你仅需要两个文件 - 一个视图模型(`path/my-component.js`) 和一个模板(`path/my-component.html`) - 它在开发中是很自然的安排。
* 因为对模板的依赖明确地放在 `define` 调用中，
它能自动在 [r.js 优化器](http://requirejs.org/docs/optimization.html) 或类似打包工具中工作。
整个组件 - 视图模型加模板 - 因此可很容易在构建步骤中包含到打包文件中。

  * 注：鉴于 r.js 优化器非常灵活，它有大量的选项且需要时间来设置。
你可能想从已通过 r.js 优化过的组件的已有示例开始，
这可以看 [Yeoman](http://yeoman.io/) 及其 [generator-ko](https://www.npmjs.org/package/generator-ko) 生成器。
随后将会有博客细说。