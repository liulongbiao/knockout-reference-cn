# 使用 extenders 来扩展 observables

Knockout observables 提供了为支持读/写值和当值变更时通知订阅者所需的基本特性。
尽管某些情况下，你可能想给 observable 添加额外的功能。
它可能包含给 observable 添加额外的属性或者通过在该 observable 前放置一个可写的 computed observable
来拦截写入。 Knockout extenders 提供了一种容易而灵活的方式来给一个 observable 添加这种类型的扩展。

### 如何创建 extender

创建 extender 涉及给 `ko.extenders` 对象添加一个函数。
该函数接收该 observable 本身作为第一个参数，和任何选项作为第二个参数。
它可以返回该 observable 或者提供某些像 computed observable 的东西以某些方式来使用原始的 observable。

下面简单地 `logChange` extender 给该 observable 订阅该 observable ，且使用控制台
来在变更时输出一个配置的消息。

```javascript
ko.extenders.logChange = function(target, option) {
    target.subscribe(function(newValue) {
       console.log(option + ": " + newValue);
    });
    return target;
};
```

你可以通过调用某个 observable 上的 `extend` 函数来使用该 extender，
并传入一个包含一个 `logChange` 属性的对象。

```javascript
this.firstName = ko.observable("Bob").extend({logChange: "first name"});
```

若 `firstName` observable 的值被改为 `Ted`，则控制台会显示 `first name: Ted`。

### 活动示例1： 强制输入为数字

该示例创建了一个 extender 强制对 observable 的写入为一个舍人到配置精度的数字。
这里，该 extender 将返回一个新的可写 computed observable ，
它在真实 observable 之前去解释其写入。

源码： 视图

```html
<p><input data-bind="value: myNumberOne" /> (round to whole number)</p>
<p><input data-bind="value: myNumberTwo" /> (round to two decimals)</p>
```

源码： 视图模型

```javascript
ko.extenders.numeric = function(target, precision) {
    //create a writable computed observable to intercept writes to our observable
    var result = ko.pureComputed({
        read: target,  //always return the original observables value
        write: function(newValue) {
            var current = target(),
                roundingMultiplier = Math.pow(10, precision),
                newValueAsNum = isNaN(newValue) ? 0 : parseFloat(+newValue),
                valueToWrite = Math.round(newValueAsNum * roundingMultiplier) / roundingMultiplier;

            //only write if it changed
            if (valueToWrite !== current) {
                target(valueToWrite);
            } else {
                //if the rounded value is the same, but a different value was written, force a notification for the current field
                if (newValue !== current) {
                    target.notifySubscribers(valueToWrite);
                }
            }
        }
    }).extend({ notify: 'always' });

    //initialize with current value to make sure it is rounded appropriately
    result(target());

    //return the new computed observable
    return result;
};

function AppViewModel(one, two) {
    this.myNumberOne = ko.observable(one).extend({ numeric: 0 });
    this.myNumberTwo = ko.observable(two).extend({ numeric: 2 });
}

ko.applyBindings(new AppViewModel(221.2234, 123.4525));
```

注意这里为它能自动从 UI 中檫除被拒绝的值，在 computed observable 上使用 `.extend({ notify: 'always' })`
是必要的。
没有它，可能用户会输入一个无效的 `newValue` 则当舍人后给出一个未改变的 `valueToWrite`。
则当视图的值将不会改变，这时将没有通知来更新 UI 上的文本框。
使用 `{ notify: 'always' }` 能导致文本框的刷新(檫除被拒绝的值)，即使 computed 属性具有非变更的值。

### 活动示例2： 给 observable 添加验证

该例创建了一个 extender 允许 observable 被标记为必需的。
取之返回一个新对象，该 extender 简单地给已有的 observable 添加额外的子 observables。
因为 observables 是函数，它们实际上具有自己的属性。
然而当视图模型被转换成 JSON 时，其中的子 observables 会被丢弃而我们只会留有我们实际 observable 的值。
这是给仅和 UI 相关而不需要发送回服务器添加额外的功能的优雅的方式。

源码： 视图

```html
<p data-bind="css: { error: firstName.hasError }">
    <input data-bind='value: firstName, valueUpdate: "afterkeydown"' />
    <span data-bind='visible: firstName.hasError, text: firstName.validationMessage'> </span>
</p>
<p data-bind="css: { error: lastName.hasError }">
    <input data-bind='value: lastName, valueUpdate: "afterkeydown"' />
    <span data-bind='visible: lastName.hasError, text: lastName.validationMessage'> </span>
</p>
```

源码： 视图模型

```javascript
ko.extenders.required = function(target, overrideMessage) {
    //add some sub-observables to our observable
    target.hasError = ko.observable();
    target.validationMessage = ko.observable();

    //define a function to do validation
    function validate(newValue) {
       target.hasError(newValue ? false : true);
       target.validationMessage(newValue ? "" : overrideMessage || "This field is required");
    }

    //initial validation
    validate(target());

    //validate whenever the value changes
    target.subscribe(validate);

    //return the original observable
    return target;
};

function AppViewModel(first, last) {
    this.firstName = ko.observable(first).extend({ required: "Please enter a first name" });
    this.lastName = ko.observable(last).extend({ required: "" });
}

ko.applyBindings(new AppViewModel("Bob","Smith"));
```

### 应用多个 extenders

可以在某个 observable 的单个 `.extend` 方法调用中应用多个 extenders 。

```javascript
this.firstName = ko.observable(first).extend({ required: "Please enter a first name", logChange: "first name" });
```

这里， `required` 和 `logChange` 两个 extenders 都会在我们的 observable 上被执行。