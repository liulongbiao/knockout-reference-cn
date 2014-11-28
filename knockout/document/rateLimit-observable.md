# 给 observable 通知限速

*注： 该限速 API 在 Knockout 3.1.0 中添加。对以前的版本， [throttle extender](./throttle-extender.md)
提供了类似的功能。*

通常， 有变更的 [observable](./observables.md) 会立即通知订阅者，因此任何依赖于该 observable 
的 computed observables 或绑定会同步更新。
然而 这个 `rateLimit` extender 会让 observable 在指定时间段里挂起并延迟变更通知。
被限速的 observable 因此会异步地更新依赖。

`rateLimit` extender 可以应用到任何类型的 observable 上，包括 [observable 数组](./observableArrays.md)
和 [computed observables](./computedObservables.md)。限速的主要用处有：

* 让事物在特定延迟后再响应
* 将多个变更结合为一个单个的更新

### 应用 `rateLimit` extender

`rateLimit` 支持两种参数格式：

```javascript
// Shorthand: Specify just a timeout in milliseconds
someObservableOrComputed.extend({ rateLimit: 500 });
 
// Longhand: Specify timeout and/or method
someObservableOrComputed.extend({ rateLimit: { timeout: 500, method: "notifyWhenChangesStop" } });
```

其 `method` 参数控制何时触发通知，并接收以下值：

1. `notifyAtFixedRate` - **若没有指定其他值时的默认值**。
通知在该 observable 首次变更 (不管是初始化还是在前一次通知发生后) 发生指定时段后触发。
2. `notifyWhenChangesStop` - 通知在该 observable 指定时段内没有发生变更后触发。
每次 observable 变更时，计时器会被重新设置，因此当 observable 的变更比超时周期更频繁时就不会触发通知。

### 示例1： 基础

考虑以下代码的 observables：

```javascript
var name = ko.observable('Bert');
 
var upperCaseName = ko.computed(function() {
    return name().toUpperCase();
});
```

通常，若你像下面这样改变 `name`：

```javascript
name('The New Bert');
```

... 则 `upperCaseName` 将被立即重计算，在你下一行代码执行前。
但如果你使用 `rateLimit` 来定义 `name` ，如下：

```javascript
var name = ko.observable('Bert').extend({ rateLimit: 500 });
```

... 则 `upperCaseName` 将不会在 `name` 改变时被立即重计算， -- 相反， `name` 会在
给 `upperCaseName` 通知新值前等待 500 毫秒，到那时才会重计算其值。
不管 `name` 在这 500 ms 内变更了多少次， `upperCaseName` 将仅以最新的值更新一次。

### 示例2： 当用户停止输入时做某些事

以下活动示例中，有一个 `instantaneousValue` observable 会在你敲击键时立即响应。
它随后被封装在一个 `delayedValue` computed observable 中，它使用了 `notifyWhenChangesStop` 方法，
来将其配置为仅在至少 400 ms 内变更停止时才通知。

源码： 视图

```html
<p>Type stuff here: <input data-bind='value: instantaneousValue,
    valueUpdate: ["input", "afterkeydown"]' /></p>
<p>Current delayed value: <b data-bind='text: delayedValue'> </b></p>
 
<div data-bind="visible: loggedValues().length > 0">
    <h3>Stuff you have typed:</h3>
    <ul data-bind="foreach: loggedValues">
        <li data-bind="text: $data"></li>
    </ul>
</div>
```

源码： 视图模型

```javascript
function AppViewModel() {
    this.instantaneousValue = ko.observable();
    this.delayedValue = ko.pureComputed(this.instantaneousValue)
        .extend({ rateLimit: { method: "notifyWhenChangesStop", timeout: 400 } });
 
    // Keep a log of the throttled values
    this.loggedValues = ko.observableArray([]);
    this.delayedValue.subscribe(function (val) {
        if (val !== '')
            this.loggedValues.push(val);
    }, this);
}
 
ko.applyBindings(new AppViewModel());
```

### 示例3： 避免多个 Ajax 请求

以下模型表现了你可以渲染为分页 grid 的模型：

```javascript
function GridViewModel() {
    this.pageSize = ko.observable(20);
    this.pageIndex = ko.observable(1);
    this.currentPageData = ko.observableArray();
 
    // Query /Some/Json/Service whenever pageIndex or pageSize changes,
    // and use the results to update currentPageData
    ko.computed(function() {
        var params = { page: this.pageIndex(), size: this.pageSize() };
        $.getJSON('/Some/Json/Service', params, this.currentPageData);
    }, this);
}
```

因为该 computed observable 求值了 `pageIndex` 和 `pageSize`，它会依赖于它们。
因此，这里的代码将使用 jQuery 的 [$.getJSON 函数](http://api.jquery.com/jQuery.getJSON/)
来在 `GridViewModel` 被首次初始化和当 `pageIndex` 和 `pageSize` 随后有变化时重新加载 `currentPageData`。

它很简单和优雅(且添加更多同样能触发在其变更时能自动刷新的 observable 查询参数也很简单)，
但它有一个潜在的效率问题。
假设你给 `GridViewModel` 添加以下能同时变更 `pageIndex` 和 `pageSize` 的函数：

```javascript
this.setPageSize = function(newPageSize) {
    // Whenever you change the page size, we always reset the page index to 1
    this.pageSize(newPageSize);
    this.pageIndex(1);
}
```

这个问题会导致 *两个* Ajax 请求：第一个在你更新 `pageSize` 时，而第二个参数
就在你随后立即启动更新 `pageIndex` 时。
它是对带宽和服务器资源的浪费，也是不可预期竞争条件的来源。

当应用到 computed observable 时， `rateLimit` extender 也能避免昂贵的 computed 函数重求值。
使用一个较短的限速超时时间(如 0 毫秒)确保了任何对依赖的同步变更将只会给你的 computed observable
触发一次重求值。如：

```javascript
ko.computed(function() {
    // This evaluation logic is exactly the same as before
    var params = { page: this.pageIndex(), size: this.pageSize() };
    $.getJSON('/Some/Json/Service', params, this.currentPageData);
}, this).extend({ rateLimit: 0 });
```

现在你可以随意次数地更新 `pageIndex` 和 `pageSize`，
而该 Ajax 调用只会在你释放线程回 JavaScript 运行时后发生一次。

### computed observables 的特殊考量

对一个 computed observable ，限速计时器将在该 computed observable 的某个依赖
而不是其值变更时被触发。
该 computed observable 仅在实际需要时才被重求值 - 在变更通知应该发生一个超时时段后，
或者当该 computed observable 的值被直接访问时。
若你需要访问该 computed 的最新求值的值时，你可以用 `peek` 方法来做。

### 强制限速 observable 总是能通知订阅者

当任何 observable 的值时原生的时，(数字、字符串、布尔值或 null)，该 observable 的依赖默认仅在
当它的值确实和以前所不同时才被通知。
因此原生值的限速的 observable 仅在当它们的值在超时时段末尾有改变时才通知。
或者说，若原生值的限速的 observable 被改变为一个新的值，而随后在超时时段末尾又被改回原来的值时，
则不会触发任何通知。

若你想确保更新总是被通知给订阅者，即使值时相同的，你可以在 `rateLimit` 之外使用 `notify` extender：

```javascript
myViewModel.fullName = ko.computed(function() {
    return myViewModel.firstName() + " " + myViewModel.lastName();
}).extend({ notify: 'always', rateLimit: 500 });
```

### 和 throttle extender 的比较

如果你想迁移使用了被废弃的 `throttle` extender 的代码，你应该注意
`rateLimit` extender 和 `throttle` extender 的不同点。

当使用 `rateLimit` 时：

1. 对 observable 的 *写入* 不会被延迟； observable 的值会被立即更新。对可写的 computed observable，
它意味着写入函数总是会立即写入。
2. 所有 `change` 通知都会被延迟，包括当手动调用 `valueHasMutated` 时。
它意味着你不能使用 `valueHasMutated` 来强制一个限速 observable 来通知一个未改变的值。
3. 默认的限速方法和 `throttle` 算法不同。要匹配 `throttle` 的行为，请使用 `notifyWhenChangesStop` 方法。
4. 对一个限速 computed observable 的求值不是限速的；当你读取其值时将被重求值。