# "selectedOptions" 绑定

### 目的

`selectedOptions` 绑定控制在多选列表中哪些元素当前被选中。
它设计为用于和 `<select>` 元素和 options 绑定一起使用。

当用户选择或取消选择多选列表中某一项时，它会给视图模型中的数组添加或移除相应的值。
例如，假设它是视图模型上一个 observable 数组，
则当你给数组添加或移除 (如通过 push 或 splice) 项时，
UI 上相应的项也会被选中或取消选中。
这是一个双向绑定。

注：要控制在单选下拉列表中哪个元素被选中，可以使用 [value 绑定](./value-binding.md)。

### 示例

```html
<p>
    Choose some countries you'd like to visit: 
    <select data-bind="options: availableCountries, selectedOptions: chosenCountries" size="5" multiple="true"></select>
</p>
 
<script type="text/javascript">
    var viewModel = {
        availableCountries : ko.observableArray(['France', 'Germany', 'Spain']),
        chosenCountries : ko.observableArray(['Germany']) // Initially, only Germany is selected
    };
     
    // ... then later ...
    viewModel.chosenCountries.push('France'); // Now France is selected too
</script>
```

### 参数

  * 主参数
  
  应该是一个数组(或 observable 数组)。KO 会设置元素的被选中选项来匹配数组的内容。
任何已有的选中项状态将被覆盖。
  
  如果你的参数是一个 observable 数组，该绑定会在数组变更
  (通过 push、pop 或其它 [observableArray 函数](./observableArrays.md)) 时更新元素元素的选中项。
如果参数不是一个 observable ，它仅会设置元素的选中项状态一次，后续不会再更新。

  不管参数是不是 observable 数组， KO 会侦测用户在多选列表中对项的选中或取消选中，
并更新数组以匹配它。这是你可以如何读取被选中项的方式。

  * 额外参数
  
    * 没有
	
### 注：让用户从任意 JavaScript 对象选择

上例中，用户可以从字符串值的数组中选择。
你并不限于提供字符串值 - 如果你想的话，你的 `options` 数组可以保护任意的 JavaScript 对象。
见 [options 绑定](./options-binding.md) 中控制如何在列表中显示任意对象的细节。

这时，你可以使用 `selectedOptions` 来读写的是这些对象本身，而不是其文本表示。
它在多数场景下导向了更清晰和优雅的代码。
你的视图模型可以认为用户从一个任意对象的数组中进行了选择，
而不用关心这些对象如何被映射到屏幕表示上。

### 依赖

没有，除了核心 Knockout 类库本身。