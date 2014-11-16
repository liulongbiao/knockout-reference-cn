# "component" 绑定

`component` 绑定给元素注入特定的 [组件](./component-overview.md)，且可选择传入参数。

* [在线示例](#live-example)
* [API](#api)
* [组件生命周期](#component-lifecycle)
* 注：仅有模板的组件
* 注：不带容器元素使用 component
* [清理和内存管理](#disposal-and-memory-management)

### <a name="live-example"></a>在线示例 

源码：视图

```html
<h4>First instance, without parameters</h4>
<div data-bind='component: "message-editor"'></div>
 
<h4>Second instance, passing parameters</h4>
<div data-bind='component: {
    name: "message-editor",
    params: { initialText: "Hello, world!" }
}'></div>
```

源码：视图模型

```javascript
ko.components.register('message-editor', {
    viewModel: function(params) {
        this.text = ko.observable(params && params.initialText || '');
    },
    template: 'Message: <input data-bind="value: text" /> '
            + '(length: <span data-bind="text: text().length"></span>)'
});
 
ko.applyBindings();
```

注：更真实的场景中，你通常会从外部文件中加载组件的视图模型和模板，而不是将它们硬编码到注册中。
查看一个 [示例和注册文档](./component-overview.md)。

### API

有两种方式来使用 `component` 绑定：

* 速记语法

  如果你传入的只是一个字符串，它被解释为组件名。具名组件被注入而不用提供任何参数。如：
  
```html
<div data-bind='component: "my-component"'></div>
```

  速记值可以是一个 observable 值。这时，如果它变更时， `component` 绑定将清除旧的组件实例，
且注入新引用的组件，例如：

```html
<div data-bind='component: observableWhoseValueIsAComponentName'></div>
```

* 完整语法

  要给组件提供参数，可以传入带以下属性的对象：

    * `name` - 要注入的组件的名称。再次说明，它可以是 observable。
    * `params` - 将传给组件的对象。通常这是一个包含多个参数地键值对对象，且通常由组件的视图模型构造器所接收。

  示例：

```html
<div data-bind='component: {
    name: "shopping-cart",
    params: { mode: "detailed-list", items: productsList }
}'></div>
```

注意当组件被移除时(不管是因为 `name` observable 变更还是因为包含它的控制流绑定移除了整个元素)，
移除的组件会被清理。

### <a name="component-lifecycle"></a>组件生命周期

当一个 `component` 绑定注入一个组件时，

1. **你的组件加载器被要求提供视图模型工厂和模板**

  这是一个 *异步* 过程(它可能涉及服务器请求)，因此组件总是异步地注入的。
  
   * 多个模块加载器被请求，指定第一个能识别该组件名称并提供一个 视图模型/模板。
这个过程对 **每个组件类型仅发生一次**，因为 Knockout 会在内存中缓存其结果的定义。
   * 默认的组件加载器基于 [你已经注册的组件](./component-registration.md) 提供默认的 视图模型/模板。
如果可用，这就是它从你的 AMD 加载器请求任何指定 AMD 模块的阶段。

2. **组件模板被克隆并注入到容器元素中**

  任何已有的内容会被移除并废弃
  
3. **如果组件具有一个视图模型，它会被实例化**

  如果视图模型给定位构造器函数，它意味着 Knockout 会调用 `new YourViewModel(params)`
  
  如果视图模型给点为一个 `createViewModel` 工厂函数，Knockout 会调用 `createViewModel(params, componentInfo)`，
其中 `componentInfo.element` 时已经注入未绑定模板的元素。

  这个阶段总是同步地完成的(构造器和工厂函数不允许是异步的)，因为它会在 *每次组件实例化时* 发生，
且如果它涉及等待网络请求的话性能将是不可接受的。

4. **视图模型被绑定到视图上**

  或者，若组件没有视图模型，则视图被绑定到任何你提供给 `component` 绑定的 `params` 上。
  
5. **组件被激活**

  现在组件在运行中，且可以根据需要留存在屏幕上。
  
  如果任何传给组件的参数是 observable，则当然该组件可以观察任何变更，或者甚至回写被修改的值。
这是它可以清晰地和父上下文交互的方式，而不会紧耦合组件代码到使用它的任何父上下文上。

6. **组件被拆除，视图模型也被清理**

  如果 `component` 绑定的 `name` 值是变化的 observable，或者当一个包含它的控制流绑定导致容器元素被移除，
则视图模型上的任何 `dispose` 都会在容器从 DOM 上被移除前调用。
查看以下 [清理和内存管理](#disposal-and-memory-management) 部分。

  注：若用户导航到一个完全不同的 web 页面，浏览器会直接做而不会请求页面中任何运行的代码来做清理工作。
这时没有 `dispose` 函数会被调用。但这时 OK 的，因为浏览器会自动释放所有曾被使用的对象的内存。

### 注：仅有模板的组件

组件通常具有视图模型，但这不是必须的。组件可以仅指定一个模板。

这时，对象被绑定到你给 `component` 绑定所传入的 `params` 对象。如：

```javascript
ko.components.register('special-offer', {
    template: '<div class="offer-box" data-bind="text: productName"></div>'
});
```

可以注入为：

```html
<div data-bind='component: {
     name: "special-offer-callout",
     params: { productName: someProduct.name }
}'></div>
```

或者更方便地，作为 [自定义元素](./component-custom-elements.md)：

```html
<special-offer params='productName: someProduct.name'></special-offer>
```

### 注：不带容器元素使用 component

有时你想不使用额外的容器元素来给视图注入一个组件。
你可以使用 *无容器控制流语法* 来完成，它基于注释标签。如：

```html
<!-- ko component: "message-editor" -->
<!-- /ko -->
```

或传入参数：

```html
<!-- ko component: {
    name: "message-editor",
    params: { initialText: "Hello, world!", otherParam: 123 }
} -->
<!-- /ko -->
```

其中 `<!-- ko -->` 和 `<!-- /ko -->` 注释用作起止标记，在标签的内部定义了一个 "虚拟元素"。
Knockout 能理解虚拟元素语法并且可以像有个真实容器元素进行绑定。

### <a name="disposal-and-memory-management"></a>清理和内存管理

可选的，你的视图模型类可能具有一个 `dispose` 函数。如果实现了的话， Knockout 会在组件被拆除
以及从 DOM 中移除(如因为相应于 `foreach` 的项被移除或 `if` 绑定变为 `false`)时调用它。

你必需使用 `dispose` 来释放任何不能继承地被垃圾回收的资源。如：

* `setInterval` 回调将持续触发直到明确地清除。
   * 使用 `clearInterval(handle)` 来停止它们，否则你的视图模型会被保持在内存中。
* `ko.computed` 属性会持续从它们的依赖接收通知直到被明确地清理
   * 若依赖是外部对象，则确保使用该 computed 属性的 `.dispose()`，否则它(可能也是你的视图模型)会被保持在内存中。
或者考虑使用 [纯 computed](./computed-pure.md) 来避免手动清理的需要。
* 对 observables 的 **订阅** 持续触发直到明确地清理
   * 若你有对外部 observable 的订阅，确保使用该订阅的 `.dispose()`，否则其回调(可能也是你的视图模型)会被保持在内存中。
* 手动在外部 DOM 元素上创建的 **事件处理器**，如果创建在 `createViewModel` 函数内部
(或甚至在常规的模块视图模型内部，尽管按照 MVVM 模式你不应该这么做) 必需被移除
   * 当然，你不需要担心视图中由标准的 Knockout 绑定所创建的任何事件处理器的释放，
因为 KO 会在元素被移除时自动注销它们。

例如：

```javascript
var someExternalObservable = ko.observable(123);
 
function SomeComponentViewModel() {
    this.myComputed = ko.computed(function() {
        return someExternalObservable() + 1;
    }, this);
 
    this.myPureComputed = ko.pureComputed(function() {
        return someExternalObservable() + 2;
    }, this);
 
    this.mySubscription = someExternalObservable.subscribe(function(val) {
        console.log('The external observable changed to ' + val);
    }, this);
 
    this.myIntervalHandle = window.setInterval(function() {
        console.log('Another second passed, and the component is still alive.');
    }, 1000);
}
 
SomeComponentViewModel.prototype.dispose = function() {
    this.myComputed.dispose();
    this.mySubscription.dispose();
    window.clearInterval(this.myIntervalHandle);
    // this.myPureComputed doesn't need to be manually disposed.
}
 
ko.components.register('your-component-name', {
    viewModel: SomeComponentViewModel,
    template: 'some template'
});
```

仅依赖于相同视图模型对象上的属性的 computeds 和订阅不是严格必要进行清理的，
因为这仅创建了一个循环引用，JavaScript 的垃圾回收知道如何释放它。
然而，为避免对哪些事物需要清理的需要，你可能倾向于尽可能地使用 `pureComputed`，
且明确地清理所有其它的 computed/subscriptions 不管技术上是否需要。