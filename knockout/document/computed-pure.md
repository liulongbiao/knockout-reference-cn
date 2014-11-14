# 纯 computed observables

*纯* computed observables 在 Knockout 3.2.0 中引入，为大多数应用提供了比常规 computed observables 更好
的性能和内存优势。这是因为一个 *纯* computed observable 在自身没有订阅者时不维护对其依赖的订阅。
这个特性：

* 给不再被应用中引用但其依赖还存在的 computed observables **避免了内存泄露**
* 通过不重新计算值没有被观察的 computed observables 的值来 **减少计算开支**

一个 *纯* computed observables 会自动根据它是否存在订阅者在两个状态间切换。

1. 当没有订阅者时，它处于 **sleeping**。当进入 *sleeping* 状态，它会清理所有对其依赖的订阅。
在这个状态，它不会在求值函数中订阅任何访问到的 observables 
(尽管它确实进行计数，这样 `getDependenciesCount()` 总是正确的)。
如果 computed observable 的值在它 *sleeping* 时被读取，它总是会重新求值确保该值是最新的。
2. 当存在订阅者时，它处于 **listening**。当进入 *listening* 状态，它立即调用求值函数并订阅任何访问到的 observables。
这个状态下，它的操作就和常规的 computed observable 一样，如 [依赖跟踪如何运作](./computed-dependency-tracking.md) 所描述。

## Why “pure”?

我们从 [纯函数](http://en.wikipedia.org/wiki/Pure_function) 的概念中借了这个术语，
因为这个特性通常仅适用于求值器像下面一样是 *纯函数* 的 computed observables ：

1. 对该 computed observable 的求值不会导致任何副作用
2. 该 computed observable 的值不会基于求值的次数或其他"隐藏"的信息。其值应该仅基于应用中其他 observables 的值，
对应于纯函数的定义，可以视作其参数。

## 语法

定义一个 *纯* computed observable 的标准方法是使用 `ko.pureComputed` ：

```javascript
this.fullName = ko.pureComputed(function() {
    return this.firstName() + " " + this.lastName();
}, this);
```

或者，你可以使用 `ko.computed` 的 `pure` 选项：

```javascript
this.fullName = ko.computed(function() {
    return this.firstName() + " " + this.lastName();
}, this, { pure: true });
```

完整语法，请查看 [computed observable 参考](./computed-reference.md)。

## 何时使用 *纯* computed observable

你可以将 *纯* 的特性用到任何遵循 [纯函数指南](./computed-pure.md) 的 computed observable 上。
然而当它应用到设计持久化视图模型被临时的视图和视图模型所使用和共享的应用设计上时更有用。
在持久化视图模型中使用 *纯* computed observable 提供了计算性能优势(但不总是这样)。
在临时视图模型中使用它们提供了内存管理优势。

在下面简单向导接口示例中， `fullName` 纯 computed 仅在最后一步被绑定到视图，因此它仅在那一步激活时被更新。

源码：视图

```html
<div class="log" data-bind="text: computedLog"></div>
<!--ko if: step() == 0-->
    <p>First name: <input data-bind="textInput: firstName" /></p>
<!--/ko-->
<!--ko if: step() == 1-->
    <p>Last name: <input data-bind="textInput: lastName" /></p>
<!--/ko-->
<!--ko if: step() == 2-->
    <div>Prefix: <select data-bind="value: prefix, options: ['Mr.', 'Ms.','Mrs.','Dr.']"></select></div>
    <h2>Hello, <span data-bind="text: fullName"> </span>!</h2>
<!--/ko-->
<p><button type="button" data-bind="click: next">Next</button></p>
```

源码： 视图模型

```javascript
function AppData() {
    this.firstName = ko.observable('John');
    this.lastName = ko.observable('Burns');
    this.prefix = ko.observable('Dr.');
    this.computedLog = ko.observable('Log: ');
    this.fullName = ko.pureComputed(function () {
        var value = this.prefix() + " " + this.firstName() + " " + this.lastName();
        // Normally, you should avoid writing to observables within a pure computed 
        // observable (avoiding side effects). But this example is meant to demonstrate 
        // its internal workings, and writing a log is a good way to do so.
        this.computedLog(this.computedLog.peek() + value + '; ');
        return value;
    }, this);
 
    this.step = ko.observable(0);
    this.next = function () {
        this.step(this.step() === 2 ? 0 : this.step()+1);
    };
};
ko.applyBindings(new AppData());
```

## 何时不使用 *纯* computed observable

### 副作用

你不应该在意为在其依赖变更时执行某个动作的 computed observable 上使用 *纯* 特性。如：

* 当使用 computed observable 来基于多个 observables 运行回调

```javascript
ko.computed(function () {
    var cleanData = ko.toJS(this);
    myDataClient.update(cleanData);
}, this);
```

* 在绑定的 `init` 函数中，使用 computed observable 来更新所绑定元素

```javascript
ko.computed({
    read: function () {
        element.title = ko.unwrap(valueAccessor());
    },
    disposeWhenNodeIsRemoved: element
});
```

你不该在求值器具有重要的副作用时使用 pure computed 的原因，简单来讲就是只要该 computed 
没有活动的订阅者(因此它处于 sleeping) 求值器将不会运行。
如果当依赖变更时求值器总是运行很重要，就要使用 [常规的 observable](./computedObservable.md)。

### 性能

某些情况下给 computed observable 使用 pure 特性反而会导致更高的计算开支。
常规的 computed observable 仅在其某个依赖有变更时重新求值。
但 *纯* computed observable 在 sleeping 状态中每次被访问到时和进入 listening 状态时都会重新求值。
因此就存在使用 pure 特性的 computed observable 比常规情况求值更频繁的可能性。