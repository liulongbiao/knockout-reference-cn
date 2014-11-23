# "checked" 绑定

### 目的

`checked` 绑定将一个可选中表单控件 - 即一个多选框 (`<input type='checkbox'>`) 
或单选框 (`<input type='radio'>`) - 链到某个视图模型的属性上。

当用户选中关联的表单控件，它会更新你的视图模型上的值。
类似地，当你更新视图模型上的值时，它也会在屏幕上选中或取消选中表单控件。

注：对文本框、下拉框和所有其它非可选中表单控件，请使用 [value 绑定](./value-binding.md)
而不是 `checked` 绑定来读写元素的值。

### 多选框示例

这个示例简单地在文本框当前得到焦点时显示一条消息，且你可以使用按钮来编程式地触发焦点。

```html
<p>Send me spam: <input type="checkbox" data-bind="checked: wantsSpam" /></p>
 
<script type="text/javascript">
    var viewModel = {
        wantsSpam: ko.observable(true) // Initially checked
    };
 
    // ... then later ...
    viewModel.wantsSpam(false); // The checkbox becomes unchecked
</script>
```

### 添加绑定到数组上的多选框示例

```html
<p>Send me spam: <input type="checkbox" data-bind="checked: wantsSpam" /></p>
<div data-bind="visible: wantsSpam">
    Preferred flavors of spam:
    <div><input type="checkbox" value="cherry" data-bind="checked: spamFlavors" /> Cherry</div>
    <div><input type="checkbox" value="almond" data-bind="checked: spamFlavors" /> Almond</div>
    <div><input type="checkbox" value="msg" data-bind="checked: spamFlavors" /> Monosodium Glutamate</div>
</div>
 
<script type="text/javascript">
    var viewModel = {
        wantsSpam: ko.observable(true),
        spamFlavors: ko.observableArray(["cherry","almond"]) // Initially checks the Cherry and Almond checkboxes
    };
 
    // ... then later ...
    viewModel.spamFlavors.push("msg"); // Now additionally checks the Monosodium Glutamate checkbox
</script>
```

### 添加单选框示例

```html
<p>Send me spam: <input type="checkbox" data-bind="checked: wantsSpam" /></p>
<div data-bind="visible: wantsSpam">
    Preferred flavor of spam:
    <div><input type="radio" name="flavorGroup" value="cherry" data-bind="checked: spamFlavor" /> Cherry</div>
    <div><input type="radio" name="flavorGroup" value="almond" data-bind="checked: spamFlavor" /> Almond</div>
    <div><input type="radio" name="flavorGroup" value="msg" data-bind="checked: spamFlavor" /> Monosodium Glutamate</div>
</div>
 
<script type="text/javascript">
    var viewModel = {
        wantsSpam: ko.observable(true),
        spamFlavor: ko.observable("almond") // Initially selects only the Almond radio button
    };
 
    // ... then later ...
    viewModel.spamFlavor("msg"); // Now only Monosodium Glutamate is checked
</script>
```

### 参数

  * 主参数
  
  KO 设置元素的 checked 状态以匹配你的参数值。任何已有的 checked 状态将被覆盖。
你的参数根据所绑定的元素类型来解释：

    * 对 **多选框**，KO 将在参数值为 `true` 时将元素设为 *checked*，而 `false` 时设置为 *unchecked*。
如果给定值实际上不是布尔值，它会被松散地解释。即非零数字和非 null 对象及非空字符串将被解释为 `true`，
而 0、null、undefined 及空字符串将被解释为 `false`。

      当用户选中或取消选中多选框时， KO 将相应地将模型属性设置为 `true` 或 `false`。
	  
	  当你的参数解析为一个 **数组** 会给出特殊考量。
这时， KO 将在其值匹配数组中某一项时设置元素为 *checked*，而数组不包含时为 *unchecked*。

      当用户选中或取消选中多选框时， KO 将相应地从数组中添加或移除该值。
	  
	* 对 **单选框**，KO 将仅在参数值等于单选框节点的 `value` 属性或由 `checkedValue` 参数所指定的值时，
将元素设为 *checked*。上例中，带有 `value="almond"` 的单选框仅在视图模型的 `spamFlavor` 属性等于 `"almond"`
时被选中。

      当用户改变所选中的单选框时，KO 会将模型属性设置为等于被选中单选框的值。
上例中，点击带 `value="cherry"` 的单选框会将 `viewModel.spamFlavor` 设置为 `"cherry"`。

      当然，最有用的是你有多个单选框绑定到单个模型属性上。
为确保任何时候仅有一个单选框可被选中，你需要将所有它们的 `name` 属性设置为任意相同的值 
(上例中为 `flavorGroup`) - 这样将它们放到一个组中可让它们仅有一个可被选中。

    如果你的参数是 observable 值，该绑定会在值变更时更新元素的 checked 状态。
而如果参数不是 observable，它仅会设置元素的 checked 状态一次，后续不会再更新。

  * 额外参数
  
    * checkedValue

	  如果你的绑定也包含了 `checkedValue`，它定义了由 `checked` 所使用的值而不是元素的 `value` 属性。
这在你希望值是非字符串的东西(如整数或对象)，或你想动态设置值时非常有用。

      下例中，当选中相应的多选框时，项对象本身(而不是它们的 `itemName` 字符串)将被包含在 `chosenItems` 数组中。
	  
	  如果你的 `checkedValue` 参数是一个 observable 值，
当值变更和元素当前被选中时，该绑定会相应地更新 `checked` 模型属性。
对多选框，它会从数组中移除旧值和添加新值。对单选框，它仅会更新模型值。

```html
<!-- ko foreach: items -->
    <input type="checkbox" data-bind="checkedValue: $data, checked: $root.chosenItems" />
    <span data-bind="text: itemName"></span>
<!-- /ko -->
 
<script type="text/javascript">
    var viewModel = {
        items: ko.observableArray([
            { itemName: 'Choice 1' },
            { itemName: 'Choice 2' }
        ]),
        chosenItems: ko.observableArray()
    };
</script> 
```

### 依赖

没有，除了核心 Knockout 类库本身。