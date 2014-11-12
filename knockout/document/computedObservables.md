# Computed Observables

假设你有一个 Observable 为 `firstName`，另一个为 `lastName`，且你希望显示全名该怎么办？
这就是 Computed Observables 的用武之地 - 它们是依赖于一或多个其它 Observables 的函数，
且在它们的依赖变更时会自动更新。

例如，给点以下视图模型类：

```javascript
function AppViewModel() {
    this.firstName = ko.observable('Bob');
    this.lastName = ko.observable('Smith');
}
```

你可以添加一个 Computed Observable 来返回全名：

```javascript
function AppViewModel() {
    // ... leave firstName and lastName unchanged ...
 
    this.fullName = ko.computed(function() {
        return this.firstName() + " " + this.lastName();
    }, this);
}
```

现在，你可以像下面一样绑定 UI 元素：

```html
The name is <span data-bind="text: fullName"></span>
```

它会在任何 `firstName` 或 `lastName` 变更时进行更新(你的求值函数将会在每次其依赖发生变更时调用一次，
且任何返回的值都会传递给诸如 UI 元素或其他 Computed Observables 这样的观察者)。

## 依赖链正常运转

当然如果你想，你可以创建整个 computed observables 的链。如，你可能有：

* 一个称为 `items` 的 **observable** 表示项的集合
* 另一个称为 `selectedIndexes` 的 **observable** 存储了用户所选择的项的索引
* 一个名为 `selectedItems` 的 **computed observable** 返回对应于所选择索引在数组中的项
* 另一个基于 `selectedItems` 是否具有某些属性(如是否是新的或未保存的)而返回 `true` 或 `false` 的 **computed observable**。
某些 UI 元素，如按钮，可能基于该值被启用或禁用。

对 `items` 或 `selectedIndexes` 的变更将传播过整个 computed observables 链，这进一步会更新任何绑定在它们之上的 UI 元素。

## 管理 `this`

`ko.computed` 的第二个参数(我们在上例中传入了 `this`)定义了当求值该 computed observable 时的 `this` 的值。
如果不传入它，它就不能引用 `this.firstName()` 或 `this.lastName()`。
高级 JavaScript 开发人员会认为这很显而易见，但如果你还不太了解 JavaScript，它看起来就很奇怪。
(像 C# 和 Java 从不希望程序员来给 `this` 设值，但 JavaScript 需要，因此其函数本身默认不是任何对象的部分。)

### 常见的简化事情的约定

有个常见的约定完全避开了跟踪 `this` 的需要：如果你的视图模型的构造器将 `this` 的引用拷贝到一个不同的变量
(通常为 `self`)，你就可以在你整个视图模型中都使用 `self` 而不需要担心它会指向其它东西。如：

```javascript
function AppViewModel() {
    var self = this;
 
    self.firstName = ko.observable('Bob');
    self.lastName = ko.observable('Smith');
    self.fullName = ko.computed(function() {
        return self.firstName() + " " + self.lastName();
    });
}
```

因为 `self` 被捕获到函数的闭包中，它在所有诸如 computed observable 求值函数这样的内嵌函数中都一致且可用。
这种约定在事件处理器中更加有用，如你在很多 [在线示例](http://knockoutjs.com/examples/) 中可以看到。

## 纯 computed observables

如果你的 computed observable 只是简单地基于某些 observable 依赖进行计算并返回一个值，
那么最好将其声明为 `ko.pureComputed` 而不是 `ko.computed`，如：

```javascript
this.fullName = ko.pureComputed(function() {
    return this.firstName() + " " + this.lastName();
}, this);
```

因为这个 computed 被声明为 *纯* 的(即其求值器不直接修改其他对象或状态)，Knockout 可以更高效地管理
其重求值和内存使用。Knockout 将没有其它代码有活动的依赖于它时会自动挂起或释放它。

纯 computeds 在 Knockout 3.2.0 中引入。[更多纯 computed observable 信息](http://knockoutjs.com/documentation/computed-pure.html)

## 强制 computed observables 总是通知订阅者

当一个 computed observable 返回原生值(数字、字符串、布尔值或 `null`)，该 observable 的依赖通常
只会在其值确实改变时得到通知。然而，可以使用内建的 `notify` extender 来确保其订阅者总是在写入发生时得到通知，即使值没有改变。
你可以像下面这样应用这个 extender ：

```javascript
myViewModel.fullName = ko.pureComputed(function() {
    return myViewModel.firstName() + " " + myViewModel.lastName();
}).extend({ notify: 'always' });
```

### 延迟并/或挂起变更通知

通常，computed observable 会在变更发生时立即通知其订阅者。
当如果一个 computed observable 频繁地变更或者会触发较昂贵的更新，你可以通过限制或延迟 computed observable 的变更通知
来取得更好的性能。这可以通过使用 `rateLimit` extender 来完成，如：

```javascript
// Ensure it notifies about changes no more than once per 50-millisecond period
myViewModel.fullName.extend({ rateLimit: 50 });
```

### 判断一个属性是否是一个 computed observable

某些情况下，编程式地判断你是否正在处理一个 computed observable 可能会有用。
Knockout 提供了工具函数 `ko.isComputed` 来帮助这种场景。
例如，你可能想从需要发送回服务器的数据中排除 computed observables：

```javascript
for (var prop in myObject) {
  if (myObject.hasOwnProperty(prop) && !ko.isComputed(myObject[prop])) {
      result[prop] = myObject[prop];
  }
}
```

另外， Knockout 提供了类似的函数来操作 observables 和 computed observables:

* `ko.isObservable` 对 observables、observable arrays 和所有 computed observables 返回 `true`
* `ko.isWritableObservable` 对 observables、observable arrays 和可写的 computed observables 返回 `true`(也有别名 `ko.isWriteableObservable`)