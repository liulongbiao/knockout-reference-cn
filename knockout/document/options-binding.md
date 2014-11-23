# "options" 绑定

### 目的

`options` 绑定控制在下拉列表(即 `<select>` 元素) 或多选列表 (如 `<select size='6'>`)中应该出现什么选项。
该绑定不能用在除 `<select>` 元素以外的元素。

你所赋予的值应该为一个数组(或 observable 数组)。`<select>` 元素将给数组中每个元素显示一项。

注：对多选列表，要设置或读取哪些选项被选中，请使用 [selectedOptions 绑定](./selectedOptions-binding.md)。
对单选列表，你也可以使用 [value 绑定](./value-binding.md) 来读写被选中项。

### 示例1：下拉列表

```html
<p>
    Destination country:
    <select data-bind="options: availableCountries"></select>
</p>
 
<script type="text/javascript">
    var viewModel = {
        // These are the initial options
        availableCountries: ko.observableArray(['France', 'Germany', 'Spain'])
    };
 
    // ... then later ...
    viewModel.availableCountries.push('China'); // Adds another option
</script>
```

### 示例2：多选列表

```html
<p>
    Choose some countries you would like to visit:
    <select data-bind="options: availableCountries" size="5" multiple="true"></select>
</p>
 
<script type="text/javascript">
    var viewModel = {
        availableCountries: ko.observableArray(['France', 'Germany', 'Spain'])
    };
</script>
```

### 示例3：展示任意 JavaScript 对象而不仅是字符串的下拉列表

```html
<p>
    Your country:
    <select data-bind="options: availableCountries,
                       optionsText: 'countryName',
                       value: selectedCountry,
                       optionsCaption: 'Choose...'"></select>
</p>
 
<div data-bind="visible: selectedCountry"> <!-- Appears when you select something -->
    You have chosen a country with population
    <span data-bind="text: selectedCountry() ? selectedCountry().countryPopulation : 'unknown'"></span>.
</div>
 
<script type="text/javascript">
    // Constructor for an object with two properties
    var Country = function(name, population) {
        this.countryName = name;
        this.countryPopulation = population;
    };
 
    var viewModel = {
        availableCountries : ko.observableArray([
            new Country("UK", 65000000),
            new Country("USA", 320000000),
            new Country("Sweden", 29000000)
        ]),
        selectedCountry : ko.observable() // Nothing selected by default
    };
</script>
```

### 示例4：展示任意 JavaScript 对象，带有从所展示项的函数计算而来的显示文本的下拉框列表

```html
<!-- Same as example 3, except the <select> box expressed as follows: -->
<select data-bind="options: availableCountries,
                   optionsText: function(item) {
                       return item.countryName + ' (pop: ' + item.countryPopulation + ')'
                   },
                   value: selectedCountry,
                   optionsCaption: 'Choose...'"></select>
```

注意 示例3 和 示例4 的差别只是 `optionsText` 值。

### 参数

  * 主参数
  
  你应该提供一个数组(或 observable 数组)。对其中的没一项， KO 将会给关联的 `<select>` 节点添加一个 `<option>`。
任何已有的选项将被移除。

  如果参数值时字符串的数组，你就不需要给定其它参数了。`<select>` 元素会给每个字符串值显示一个选项。
然而如果你想让用户从任意 JavaScript 对象(而不仅是字符串)的数组中选择，
则可以查看下面的 `optionsText` 和 `optionsValue` 参数。

  如果该参数是个 observable 值，该绑定会在值变更时更新可用的选项。
如果参数不是 observable ，它仅会设置元素可以选项一次，且后续不会再更新。

  * 额外参数
  
    * optionsCaption
    
      有时可能不想默认选中任意特定选项。但单选下拉列表通常由某些选中项开始，
因此你该如何避免预选中呢？常见的方案是在可选项列表前前缀一个特殊的空选项，
其文本类似于 "Select an item" 或 "Please choose an option" ，
然后默认选中这一项即可。

      它很容易做：只要添加一个名为 `optionsCaption` 的额外属性，其值为要显示的字符串。
      
      KO 会在选项列表的前面前缀一个显示文本 "Select an item…" 而值为 `undefined` 的选项。
因此如果 `myChosenValue` 持有值 `undefined` (它也是 observables 的默认值)，
则空选项将被选中。如果 `optionsCaption` 参数是一个 observable ，
则初始项的文本将会在值变更时被更新。

    * optionsText
    
      上面的示例3中可看到你可以绑定 `options` 到任意 JavaScript 对象(不只是字符串)的数组。
这时，你需要选择哪个对象属性应被显示为下拉列表会多选列表中的文本。
示例3显示了你可以如何通过传入一个名为 `optionsText` 的额外参数来指定属性名称。

      如果你不想仅仅给下拉框中每个项显示单个属性值作为文本，你也可以给 `optionsText` 传入一个 JavaScript 函数，
并提供你自己的任意用于给被展示对象计算显示文本的逻辑。
见上述的示例4，它展示了如何通过将多个属性值连接到一起来生成显示文本。

    * optionsValue
    
      类似于 `optionsText`，你可以传入一个名为 `optionsValue` 的额外参数来指定对象的哪个属性
应作为 KO 所生成的 `<option>` 元素的 `value` 属性。
你也可以指定一个 JavaScript 函数来决定其值。该函数将接收被选中项作为其唯一参数
并且应该返回一个用作 `<option>` 元素的 `value` 属性的字符串。

      通常，你仅希望使用 `optionsValue` 作为一种确保 KO 可以在你更新可选项集时保留选中项的一种方式。
例如，当你通过 Ajax 调用反复获取汽车对象的列表且希望确保已选择的汽车被保留，
你可能需要将 `optionsValue` 设置为 `"carId"` 或其它每个汽车对象所拥有的唯一标识，
否则 KO 将不会知道以前的汽车对象对应于新的哪个汽车对象。

    * optionsIncludeDestroyed
    
      有时你可能想标记某个数组元素为已删除，但不失去记录的存在性。
这被称为非破坏性删除。如何做的详细信息，查看 [observableArray 的 destroy 函数](./observableArrays.md#destroy-and-destroyAll)。
      
      默认，options 绑定会跳过(即隐藏)任何被标记为 destroyed 的元素。
如果你想显示被 destroyed 项，可以指定额外参数，如：`<select data-bind='options: myOptions, optionsIncludeDestroyed: true'></select>`

    * optionsAfterRender
    
      如果你需要在生成的 `<option>` 元素上运行进一步的自定义逻辑，你可以使用 `optionsAfterRender`回调。见下面注2。
    
    * selectedOptions
    
      对多选列表，你可以使用 `selectedOptions` 来读写选中状态。技术上它是独立的绑定，
因此它有 [自己的文档](./selectedOptions-binding.md)。

    * valueAllowUnset
    
      如果你想让 Knockout 允许你的模型属性接收在 `<select>` 元素中没有相应项的值
(且通过将 `<select>` 元素显示为空白)，
查看 [valueAllowUnset 相关文档](./value-binding.md#using-valueallowunset-with-select-elements)。
    
### 注1：当设置、变更选项时保留被选中项

当 `options` 绑定变更 `<select>` 元素中可选项的集合时，KO 会让用户已选中项尽可能不变。
因此对单选下拉列表，已有的选中项的值将还会被选中，而对多选列表，所有已有的选中项的值
将依旧被选中(除非，当然你移除了一或多个选项)。

这时因为 `options` 绑定试图独立于 `value` 绑定(它控制了单选列表的选中项)
和 `selectedOptions` 绑定(它控制了多选列表的选中项)。

### 注2：后处理生成的选项

如果你需要在生成的 `<option>` 元素上执行进一步的自定义逻辑，
你可以使用 `optionsAfterRender` 回调。该回调函数在每次一个 `<option>` 元素被插入列表中时被调用，
它带有以下参数：

1. 被插入 `<option>` 元素
2. 它所绑定的数据项，或对提示元素而言是 `undefined`

下例中使用 `optionsAfterRender` 给每个选项添加 `disable` 绑定。

```html
<select size=3 data-bind="
    options: myItems,
    optionsText: 'name',
    optionsValue: 'id',
    optionsAfterRender: setOptionDisable">
</select>
 
<script type="text/javascript">
    var vm = {
        myItems: [
            { name: 'Item 1', id: 1, disable: ko.observable(false)},
            { name: 'Item 3', id: 3, disable: ko.observable(true)},
            { name: 'Item 4', id: 4, disable: ko.observable(false)}
        ],
        setOptionDisable: function(option, item) {
            ko.applyBindingsToNode(option, {disable: item.disable}, item);
        }
    };
    ko.applyBindings(vm);
</script>
```

### 依赖

没有，除了核心 Knockout 类库本身。