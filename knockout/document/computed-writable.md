# 可写的 computed observables

*初学者可能想跳过本节- 可写的 computed observables 比较高阶且大多数情形下不是必需的*

通常， computed observables 具有一个从其他 observables 计算而来的值，因此是 *只读* 的。
这看起来有些令人惊讶，我们可以让 computed observables 变得 *可写* 。
你只需要提供自己的回调函数，对写入的值做些有意义的更新即可。

你可以像常规的 observable 一样来使用一个可写的 computed observable，其中带有你自己的定制逻辑来
解释所有的读和写。就像 observable 一样，你可以使用 *链式语法* 给一个模型对象上的
多个 observable 或 computed observable 属性写值。
如， `myViewModel.fullName('Joe Smith').age(50)`。

可写的 computed observables 是一种强大的特性，可有大量可能的用处。

## 示例 1 ： 分解用户输入

回到经典的 "first name + last name = full name" 示例，你可以将事物转换回去：
让 `fullName` computed observable 可写，这样用户可以直接编辑全名，且其提供的值将被解析并映射回
底层的 `firstName` 和 `lastName` observables。
本例中， `write` 回调通过将文本分割成 “firstName” 和 “lastName” 组件，并分别写回给底层的 observables 来处理输入的的值。

源码： 视图

```html
<div>First name: <span data-bind="text: firstName"></span></div>
<div>Last name: <span data-bind="text: lastName"></span></div>
<div class="heading">Hello, <input data-bind="textInput: fullName"/></div>
```

源码： 视图模型

```javascript
function MyViewModel() {
    this.firstName = ko.observable('Planet');
    this.lastName = ko.observable('Earth');
 
    this.fullName = ko.pureComputed({
        read: function () {
            return this.firstName() + " " + this.lastName();
        },
        write: function (value) {
            var lastSpacePos = value.lastIndexOf(" ");
            if (lastSpacePos > 0) { // Ignore values with no space character
                this.firstName(value.substring(0, lastSpacePos)); // Update "firstName"
                this.lastName(value.substring(lastSpacePos + 1)); // Update "lastName"
            }
        },
        owner: this
    });
}
 
ko.applyBindings(new MyViewModel());
```

这里恰好和 [Hello World](http://knockoutjs.com/examples/helloWorld.html) 例子相反，
这里 firstName 和 lastName 是不可编辑的，但组合后的 fullName 是可编辑的。

前面的视图模型代码演示了用于初始化 computed observables 的 *单参数语法* 。
查看 [computed observable 参考](./computed-reference.md) 中更多可用选项。

## 示例 2 ： 选择/反选所有项

当给用户呈现一个可选项的列表时，包含一个方法来选择或取消选择所有项常会很有用。
这可以直觉地通过一个表示是否所有项都被选中的布尔值来表示。
当设置为 `true` 时它会选择所有项，而当设置为 `false` 时会取消选择所有项。

源码： 视图

```html
<div class="heading">
    <input type="checkbox" data-bind="checked: selectedAllProduce" title="Select all/none"/> Produce
</div>
<div data-bind="foreach: produce">
    <label>
        <input type="checkbox" data-bind="checkedValue: $data, checked: $parent.selectedProduce"/>
        <span data-bind="text: $data"></span>
    </label>
</div>
```

源码： 视图模型

```javascript
function MyViewModel() {
    this.produce = [ 'Apple', 'Banana', 'Celery', 'Corn', 'Orange', 'Spinach' ];
    this.selectedProduce = ko.observableArray([ 'Corn', 'Orange' ]);
    this.selectedAllProduce = ko.pureComputed({
        read: function () {
            // Comparing length is quick and is accurate if only items from the
            // main array are added to the selected array.
            return this.selectedProduce().length === this.produce.length;
        },
        write: function (value) {
            this.selectedProduce(value ? this.produce.slice(0) : []);
        },
        owner: this
    });
}
ko.applyBindings(new MyViewModel());
```

## 示例 3 ： 值转换器

有时你可能希望以不同于底层存数的格式在屏幕上呈现数据。
如，你可能想将单价存储为原始浮点值，但让用户可以带一个货币符合固定小数位数来编辑它。
你可以使用可写的 computed observable 来表示格式化的单价，将输入的值映射回底层浮点值：

源码： 视图

```html
<div>Enter bid price: <input data-bind="textInput: formattedPrice"/></div>
<div>(Raw value: <span data-bind="text: price"></span>)</div>
```

源码： 视图模型

```javascript
function MyViewModel() {
    this.price = ko.observable(25.99);
 
    this.formattedPrice = ko.pureComputed({
        read: function () {
            return '$' + this.price().toFixed(2);
        },
        write: function (value) {
            // Strip out unwanted characters, parse as float, then write the 
            // raw data back to the underlying "price" observable
            value = parseFloat(value.replace(/[^\.\d]/g, ""));
            this.price(isNaN(value) ? 0 : value); // Write to underlying storage
        },
        owner: this
    });
}
 
ko.applyBindings(new MyViewModel());
```

现在，当用户输入新的价格时，文本框会立即以带货币符的两位小数的格式来更新，而不管他们输入的格式是什么。
这具有良好的用户体验，因为用户可以看到软件如何理解他们输入的作为价格的数据。
他们知道不能输入超过两个小数位的数，因为他们这么尝试时额外的小数位会被立即移除。
类似地，他们不能输入负数，因为 `write` 回调会去除任何负号。

## 示例 4 ： 过滤和验证用户输入

示例 1 显示了如何可写的 computed observable 可通过选择
在不满足某些条件是不将特定的值写回底层的 observables 来有效地 *过滤* 输入数据。
它会忽略不包含空格的全名的值。

更进一步，你可以根据最新的输入是否满足来切换一个 `isValid` 标记，并相应地在 UI 中显示消息。
虽然存在更容易的验证方式(下面会解释)，但首先考虑下下面的示例，它演示了这种机制：

源码：视图

```html
<div>Enter a numeric value: <input data-bind="textInput: attemptedValue"/></div>
<div class="error" data-bind="visible: !lastInputWasValid()">That's not a number!</div>
<div>(Accepted value: <span data-bind="text: acceptedNumericValue"></span>)</div>
```

源码： 视图模型

```javascript
function MyViewModel() {
    this.acceptedNumericValue = ko.observable(123);
    this.lastInputWasValid = ko.observable(true);
 
    this.attemptedValue = ko.pureComputed({
        read: this.acceptedNumericValue,
        write: function (value) {
            if (isNaN(value))
                this.lastInputWasValid(false);
            else {
                this.lastInputWasValid(true);
                this.acceptedNumericValue(value); // Write to underlying storage
            }
        },
        owner: this
    });
}
 
ko.applyBindings(new MyViewModel());
```

现在，`acceptedNumericValue` 将仅包含数字值，且任何其他输入的值将触发验证消息的出现，而不是更新 `acceptedNumericValue`。

> 注：对这样简单地验证输入是数字的需求，这种技术有些大材小用了。
> 直接在 `<input>` 上使用 jQuery Validation 和其 `number` 类会更简单一些。
> Knockout 和 jQuery Validation 可以很好地一起工作，
> 如 [grid 示例](http://knockoutjs.com/examples/gridEditor.html) 中演示的那样。
> 然而，前面的示例演示了一种更通用的过滤和验证机制，带有自定义逻辑来控制出现哪种用户反馈，
> 这样你可以在比 jQuery Validation 可原生处理的更复杂些的场景中使用。