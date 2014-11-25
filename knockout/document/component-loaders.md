# 组件加载器

当你使用 [component 绑定](./component-binding.md) 或 [自定义元素](./component-custom-elements.md)
来注入一个 [组件](./component-overview.md)， Knockout 使用一或多个 *组件加载器*
来获取组件的模板和视图模型。
组件加载器的任务是给任何特定的组件名称异步地提供 模板/视图模型 对。

* [默认组件加载器](#default-component-loader)
* [组件加载器工具函数](#component-loader-utility-functions)
* [实现自定义组件加载器](#custom-component-loader)
  * [你可以实现的函数](#functions-you-can-implement)
    * [getConfig(name, callback)](#getconfigname-callback)
    * [loadComponent(name, componentConfig, callback)](#loadcomponentname-componentconfig-callback)
    * [loadTemplate(name, templateConfig, callback)](#loadtemplatename-templateconfig-callback)
    * [loadViewModel(name, templateConfig, callback)](#loadviewmodelname-templateconfig-callback)
  * [注册自定义组件加载器](#registering-custom-component-loaders)
  * [控制优先级](#controlling-precedence)
  * [调用顺序](#sequence-of-calls)
* [示例1：设置命名约定的组件加载器](#example-1-a-component-loader-that-sets-up-naming-conventions)
* [示例2：使用自定义代码加载外部文件的组件加载器](#example-2-a-component-loader-that-loads-external-files-using-custom-code)
* [注：自定义组件加载器和自定义元素](#note-custom-component-loaders-and-custom-elements)
* [注：集成 browserify](#note-integrating-with-browserify)

### <a name="default-component-loader">The default component loader</a>默认组件加载器

内建默认组件加载器， `ko.components.defaultLoader`，基于围绕一个组件定义的中心 "注册表"。
它依赖于给每个组件在使用组件前显式地注册一个配置。

[学习更多组件默认加载器的配置和注册](./component-registration.md)

### <a name="component-loader-utility-functions"></a>组件加载器工具函数

以下函数可用于读写默认组件加载器的注册表：

* `ko.components.register(name, configuration)`
  * 注册一个组件。查看 [完整文档](./component-registration.md)
* `ko.components.isRegistered(name)`
  * 若特定名称的组件已被注册则返回 `true`，否则返回 `false`
* `ko.components.unregister(name)`
  * 从注册表中移除具名组件。或者若没有该注册的组件，则什么都不做

以下函数可跨所有被注册的组件加载器所使用(不只是默认加载器)：

* `ko.components.get(name, callback)`
  * 按顺序轮询每个注册的加载器(默认仅有默认加载器)，以找到第一个能对该具名组件提供一个 视图模型/模板 定义的加载器，
然后调用 `callback` 以返回该 视图模型/模板 声明。
若该注册的加载器不知晓该组件则调用 `callback(null)`。
* `ko.components.clearCachedDefinition(name)`
  * 通常， Knockout *给每个组件名称* 轮询加载器一次，然后它会缓存结果定义。
它确保大量的组件可以被很快地实例化。
若你想给指定组件清理缓存项，调用它，然后下次该组件被需要时会再次轮询加载器。

同样，因为 `ko.components.defaultLoader` 是一个组件加载器，它实现了以下标准的组件加载器函数。
你可以直接调用它们，如作为你的自定义加载器实现的一部分：

* `ko.components.defaultLoader.getConfig(name, callback)`
* `ko.components.defaultLoader.loadComponent(name, componentConfig, callback)`
* `ko.components.defaultLoader.loadTemplate(name, templateConfig, callback)`
* `ko.components.defaultLoader.loadViewModel(name, viewModelConfig, callback)`

对这些标准组件加载器函数的文档，查看 [实现自定义组件加载器](#custom-component-loader)

### <a name="custom-component-loader"></a>实现自定义组件加载器

如果你想使用命名约定而不是显式注册来加载组件，
或者你想使用第三方 "加载器" 类库来从外部文件获取组件视图模型或模板，
你可能想使用一个自定义组件加载器。


#### <a name="functions-you-can-implement"></a>你可以实现的函数

一个自定义组件加载器就是一个简单对象，其属性可以是以下函数的 **任意组合**：

##### <a name="getconfigname-callback"></a>`getConfig(name, callback)`

如果 *你想基于名称，如实现命名约定，来编程式地提供配置*，可以定义这个。

如果声明了， Knockout 将调用该函数来给每个待实例化的组件调用该函数来获取配置对象。

* 要提供一个配置，调用 `callback(componentConfig)`，其中 componentConfig 可以是任何可由
你的加载器或其它加载器上的 `loadComponent` 函数所理解的对象。
默认加载器简单地提供了任何使用 `ko.components.register` 所注册的对象。
* 如，一个像 `{ template: 'someElementId', viewModel: { require: 'myModule' } }` 这样的
可由默认加载器所理解和实例化的 `componentConfig`。
* 你不限于提供以任何标准格式的配置对象。你可以提供任意对象，只要你的 `loadComponent` 函数能理解它们。
* 若你不想你的加载器给该具名组件提供配置，则调用 `callback(null)`。
Knockout 随后会按顺序轮询其它注册的加载器，直到某个提供了一个非 `null` 值。

##### <a name="loadcomponentname-componentconfig-callback"></a>`loadComponent(name, componentConfig, callback)`

如果 *你想控制组件配置如何被解释，如你不想使用标准的 视图模型/模板 对格式*，可以定义这个。

如果声明了， Knockout 会调用该函数将一个 `componentConfig` 转换成一个 视图模型/模板 对。

* 要提供一个 视图模型/模板 对，调用 `callback(result)`，其中 `result` 是具有以下属性的对象：
  * template - **必需**。一个 DOM 节点的数组
  * createViewModel(params, componentInfo) - **可选**。
一个随后将被调用以给该组件的每个实例提供一个视图模型对象的函数。
* 若你不想你的加载器给指定参数提供一个 视图模型/模板 对，则调用 `callback(null)`。
Knockout 随后会按顺序轮询其它注册的加载器，直到某个提供了一个非 `null` 值。

##### <a name="loadtemplatename-templateconfig-callback"></a>`loadTemplate(name, templateConfig, callback)`

如果 *你想使用自定义逻辑给指定模板配置提供 DOM 节点(如，使用 ajax 请求通过 URL 来获取一个模板)*，可以定义这个。

默认组件加载器将在任何声明了该函数的注册加载器上调用该函数，以将组件配置的
`template` 部分转换成一个 DOM 节点的数组。
该节点随后会被缓存且给每个该组件的示例克隆一份。

其 `templateConfig` 简单地就是任何 `componentConfig` 对象的 `template` 属性。
例如，它可能包含 `"some markup"` 或 `{ element: "someId" }` 或像 `{ loadFromUrl: "someUrl.html" }` 这样的自定义格式。

* 要提供一个 DOM 节点的数组，调用 `callback(domNodeArray)`。
* 若你不想你的加载器给指定参数提供模板(如，因为它不能识别该配置格式)，调用 `callback(null)`。
Knockout 随后会按顺序轮询其它注册的加载器，直到某个提供了一个非 `null` 值。

##### <a name="loadviewmodelname-templateconfig-callback"></a>`loadViewModel(name, templateConfig, callback)`

如果 *你想使用自定义逻辑给指定视图模型配置提供一个视图模型工厂(如集成第三方模块加载器或依赖注入系统)*，可以定义这个。

默认组件加载器将在任何声明了该函数的注册加载器上调用该函数，以将组件配置的
`viewModel` 部分转换成一个 `createViewModel` 工厂函数。
该函数随后被缓存且对每个需要一个视图模型的该组件的实例调用一次。

其 `viewModelConfig ` 简单地就是任何 `componentConfig` 对象的 `viewModel ` 属性。
例如，它可能是一个构造器，或像 `{ myViewModelType: 'Something', options: {} }` 的自定义格式。

* 要提供一个 `createViewModel` 函数，调用 `callback(yourCreateViewModelFunction)`。
该 `createViewModel` 函数必需接收参数 `(params, componentInfo)`，
且必需在每次被调用时同步地返回一个新的视图模型实例。
* 你不想你的加载器给指定参数提供 `createViewModel` 函数 (如，因为它不能识别该配置格式)，调用 `callback(null)`。
Knockout 随后会按顺序轮询其它注册的加载器，直到某个提供了一个非 `null` 值。
      
#### <a name="registering-custom-component-loaders"></a>注册自定义组件加载器

Knockout 运行你同时使用多个组件加载器。这很有用，例如，你可以写实现了不同机制的插件
(如，一个可能根据命名约定从后端服务器获取模板；另一个可能使用依赖注入系统来设置视图模型)，
然后让它们一起工作。

因此 `ko.components.loaders` 是一个保护当前所有启用的加载器的数组。
默认该数组仅有一项： `ko.components.defaultLoader`，要添加额外的加载器，
简单地将它们插入到 `ko.components.loaders` 数组中。

#### <a name="controlling-precedence"></a>控制优先级

若你希望你的加载器优先于默认加载器(这样它具有先提供配置/值的机会)，则将其添加到数组的 *前面*。
若你希望默认加载器优先于你的加载器(这样你的自定义加载器仅在组件没有显示定义时被调用)，则将其添加到数组的 *后面*。

如：

```javascript
// Adds myLowPriorityLoader to the end of the loaders array.
// It runs after other loaders, only if none of them returned a value.
ko.components.loaders.push(myLowPriorityLoader);
 
// Adds myHighPriorityLoader to the beginning of the loaders array.
// It runs before other loaders, getting the first chance to return values.
ko.components.loaders.unshift(myHighPriorityLoader)
```

如果需要，你也可以从加载器数组中移除 `ko.components.defaultLoader`。

#### <a name="sequence-of-calls"></a>调用顺序

Knockout 首次需要给指定名称构造一个组件时，它：

* 按顺序调用每个注册的加载器的 `getConfig` 函数，直到某个提供了一个非 null `componentConfig`。
* 然后，对这个 `componentConfig`，按顺序调用每个加载器的 `loadComponent`，直到某个提供了一个非 null `template/createViewModel` 对。

当默认加载器的 `loadComponent` 运行时，它同时地：

* 按顺序调用每个注册的加载器的 `loadTemplate` 函数，直到某个提供了一个非 null DOM 数组。
  * 默认加载器本身具有一个 `loadComponent` 函数能够将大量模板配置格式解析成 DOM 数组
* 按顺序调用每个注册的加载器的 `loadViewModel` 函数，直到某个提供了一个非 null `createViewModel` 函数。
  * 默认加载器本身具有一个 `loadViewModel` 函数能够将大量视图模型配置格式解析成 `createViewModel` 函数

自定义加载器可插入该过程的任何部分，因此你可以控制提供配置、解释配置、提供 DOM 节点还是提供视图模型工厂函数。
通过将自定义加载器以选定的顺序放到 `ko.components.loaders` 中，你可以控制不同加载策略的优先级。

### <a name="example-1-a-component-loader-that-sets-up-naming-conventions"></a>示例1：设置命名约定的组件加载器

要实现命名约定，你的自定义组件加载器只需要实现 `getConfig`。如：

```javascript
var namingConventionLoader = {
    getConfig: function(name, callback) {
        // 1. Viewmodels are classes corresponding to the component name.
        //    e.g., my-component maps to MyApp.MyComponentViewModel
        // 2. Templates are in elements whose ID is the component name
        //    plus '-template'.    
        var viewModelConfig = MyApp[toPascalCase(name) + 'ViewModel'],
            templateConfig = { element: name + '-template' };
 
        callback({ viewModel: viewModelConfig, template: templateConfig });
    }
};
 
function toPascalCase(dasherized) {
    return dasherized.replace(/(^|-)([a-z])/g, function (g, m1, m2) { return m2.toUpperCase(); });
}
 
// Register it. Make it take priority over the default loader.
ko.components.loaders.unshift(namingConventionLoader);
```

现在它被注册了，你可以以任何名称来引用组件(而不用预注册它们)，如：

```html
<div data-bind="component: 'my-component'"></div>
 
<!-- Declare template -->
<template id='my-component-template'>Hello World!</template>
 
<script>
    // Declare viewmodel
    window.MyApp = window.MyApp || {};
    MyApp.MyComponentViewModel = function(params) {
        // ...
    }
</script>
```

### <a name="example-2-a-component-loader-that-loads-external-files-using-custom-code"></a>示例2：使用自定义代码加载外部文件的组件加载器

若你的自定义组件实现了 `loadTemplate` 和/或 `loadViewModel`，则你可以将自定义代码插入加载过程中。
你也可以用这些函数来解释自定义配置格式。

例如，你可能希望启用如下配置格式：

```javascript
ko.components.register('my-component', {
    template: { fromUrl: 'file.html', maxCacheAge: 1234 },
    viewModel: { viaLoader: '/path/myvm.js' }
});
```

... 然后你可以使用自定义加载器来使用。

以下自定义加载器将处理带一个 `fromUrl` 值的模板配置的加载：

```javascript
var templateFromUrlLoader = {
    loadTemplate: function(name, templateConfig, callback) {
        if (templateConfig.fromUrl) {
            // Uses jQuery's ajax facility to load the markup from a file
            var fullUrl = '/templates/' + templateConfig.fromUrl + '?cacheAge=' + templateConfig.maxCacheAge;
            $.get(fullUrl, function(markupString) {
                // We need an array of DOM nodes, not a string.
                // We can use the default loader to convert to the
                // required format.
                ko.components.defaultLoader.loadTemplate(name, markupString, callback);
            });
        } else {
            // Unrecognized config format. Let another loader handle it.
            callback(null);
        }
    }
};
 
// Register it
ko.components.loaders.unshift(templateFromUrlLoader);
```

... 而以下自定义加载器将处理带有一个 `viaLoader` 值的视图模型配置的加载：

```javascript
var viewModelCustomLoader = {
    loadViewModel: function(name, viewModelConfig, callback) {
        if (viewModelConfig.viaLoader) {
            // You could use arbitrary logic, e.g., a third-party
            // code loader, to asynchronously supply the constructor.
            // For this example, just use a hard-coded constructor function.
            var viewModelConstructor = function(params) {
                this.prop1 = 123;
            };
 
            // We need a createViewModel function, not a plain constructor.
            // We can use the default loader to convert to the
            // required format.
            ko.components.defaultLoader.loadViewModel(name, viewModelConstructor, callback);
        } else {
            // Unrecognized config format. Let another loader handle it.
            callback(null);
        }
    }
};
 
// Register it
ko.components.loaders.unshift(viewModelCustomLoader);
```

如果你喜欢，你可以通过将 `loadTemplate` 和 `loadViewModel` 放到同一个对象中来
整合 `templateFromUrlLoader` 和 `viewModelCustomLoader` 到一个加载器中。
然而将这些不同的关注点分离开来会更好，因为他们的实现是非常独立地。

### <a name="note-custom-component-loaders-and-custom-elements"></a>注：自定义组件加载器和自定义元素

如果你使用组件加载器根据命名约定来加载组件，且没有使用 `ko.components.register` 来注册你的组件，
则这些组件将不会自动地可作为自定义元素来使用(因为你还没有告诉 Knockout 它们的存在)。

详见： [如何启用名称没有对应已显式注册的组件的自定义元素](./component-custom-elements.md#registering-custom-elements)

### <a name="note-integrating-with-browserify"></a>注：集成 browserify

[Browserify](http://browserify.org/) 是以 Node 风格同步 `require` 语法来引用 JavaScript 类库的流行的库。
它常被作为像 require.js 这样的 AMD 加载器的替代方案。
然而 Browserify 解决的是一个不同的问题： 同步地构建时引用解决方案，
而不是像 AMD 所处理的异步实时引用解决方案。

因为 Browserify 是一个构建时工具，它实际上不需要特殊的 KO 组件集成方式，
且也不需要实现任何自定义组件加载器来使用它。
你可以简单地使用 Browserify 的 `require` 语句来获取组件视图模型的实例，然后注册它们。如：

```javascript
// Note that the following *only* works with Browserify - not with require.js,
// since it relies on require() returning synchronously.
 
ko.components.register('my-browserify-component', {
    viewModel: require('myViewModel'),
    template: require('fs').readFileSync(__dirname + '/my-template.html', 'utf8')
});
```

这里使用了 [brfs Browserify 插件](https://github.com/substack/brfs) 自动内联 .html 文件，
因此你需要使用类似以下命令行来构建脚本文件：

```bash
npm install brfs
browserify -t brfs main.js > bundle.js
```