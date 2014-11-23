# "template" 绑定

### 目的

`template` 绑定给关联的 DOM 元素填充渲染一个模板的结果。
模板是一种简单方便地构建复杂 UI 结构的方式 - 可能带有重复或嵌套的块 - 作为你的视图模板数据的函数。

有两种使用模板的主要方式：

* *原生模板* 时 `foreach,if,with` 和其它控制流绑定底层所使用的机制。
在内部，这些控制流绑定捕获包含在你的元素中的 HTML 标签，且将它用在模板来渲染一个任意的数据项。
这种特性内建于 Knockout 且不需要任何外部类库。
* *基于字符串模板* 是一种将 Knockout 和第三方模板引擎连接到一起的方式。
Knockout 会将你的模型值传递给外部模板引擎，且将生产的标签字符串结果注入到你的文档中。
查看以下使用 jquery.tmpl 和 Underscore 模板引擎的示例。

### 参数

  * 主参数
  
    * 快捷语法： 如果你只提供了一个字符串值， KO 会将其解释为要渲染的模板的 ID。提供给模板的数据会作为其当前模型对象。
    * 需要更多控制，可以传入具有以下属性的某种组合的 JavaScript 对象：
    
      * name - 包含你希望用于渲染的元素的ID - 见 注5 中多种编程式使用方式
      * data - 提供给模板用于渲染的数据对象。如果忽略该参数， KO 会查找一个 `foreach` 参数，或者回退到使用当前的模型对象
      * if - 如果提供了该参数，模板将仅在给点表达式求值为 `true` (或逻辑真值)时被渲染。
这在避免一个 null observable 在其被填充前被绑定到模板上时很有用。
      * foreach - 指示 KO 在 "foreach" 模式下渲染模板 - 见 注2 相关细节。
      * as - 连同 `foreach` 使用，给每个被渲染项定义一个别名 - 见 注3 相关细节。
      * afterRender,afterAdd 或 afterRemove - 在渲染后的 DOM 元素上被调用的回调函数 - 见 注4

### 注1：渲染具名模板

通常，当你使用控制流绑定(foreach,with,if 等) 时并不需要给模板一个名称：
它们隐式且匿名地定义为你的 DOM 元素内部的标签。
但如果你想，你可以提取模板到一个独立地元素，然后通过名称来引用它们：

```html
<h2>Participants</h2>
Here are the participants:
<div data-bind="template: { name: 'person-template', data: buyer }"></div>
<div data-bind="template: { name: 'person-template', data: seller }"></div>
 
<script type="text/html" id="person-template">
    <h3 data-bind="text: name"></h3>
    <p>Credits: <span data-bind="text: credits"></span></p>
</script>
 
<script type="text/javascript">
     function MyViewModel() {
         this.buyer = { name: 'Franklin', credits: 250 };
         this.seller = { name: 'Mario', credits: 5800 };
     }
     ko.applyBindings(new MyViewModel());
</script>
```

这里， `person-template` 标签被用了两次：一次是 `buyer`，一次用于 `seller`。
注意模板标签被一个 `<script type="text/html">` 标签封装 - 这种哑 type 属性在确保
标签不作为 JavaScript 来执行是有必要的，且 Knockout 不会试图给这样的标签应用绑定，
除非它们被用作模板。

通常你不会经常需要使用具名模板，但有时它可以帮助你减少重复的标签。

### 注2： 给具名模板使用 "foreach" 选项

如果你想要 `foreach` 绑定的等价物，但要使用具名模板，你可以以以下自然的方式来完成：

```html
<h2>Participants</h2>
Here are the participants:
<div data-bind="template: { name: 'person-template', foreach: people }"></div>
 
<script type="text/html" id="person-template">
    <h3 data-bind="text: name"></h3>
    <p>Credits: <span data-bind="text: credits"></span></p>
</script>
 
 function MyViewModel() {
     this.people = [
         { name: 'Franklin', credits: 250 },
         { name: 'Mario', credits: 5800 }
     ]
 }
 ko.applyBindings(new MyViewModel());
```

它和你在使用 `foreach` 时直接在元素中内嵌匿名模板相同的结果，即：

```html
<div data-bind="foreach: people">
    <h3 data-bind="text: name"></h3>
    <p>Credits: <span data-bind="text: credits"></span></p>
</div>
```

### 注3：使用 "as" 给 "foreach" 项一个别名

当嵌套 `foreach` 模板时，通常在层级的更高层引用项非常有用。
一种方式是在你的绑定中引用 `$parent` 或其它 [绑定上下文](./binding-context.md) 变量。

然而一种更简单或更优雅的选项是使用 `as` 来给你的迭代变量中声明一个名称。如：

```html
<ul data-bind="template: { name: 'employeeTemplate',
                                  foreach: employees,
                                  as: 'employee' }"></ul>
```

注意字符串值 `'employee'` 关联了 `as`。现在在 `foreach` 循环内部的任何地方，
你的子模板中的绑定都能引用 `employee` 来访问被渲染的雇员对象。

这主要在你有多个嵌套的 `foreach` 块时非常有用，因为它给你一种清晰的方式来引用任何在层级总更高层
所定义的具名项。
下面是一个完整示例，显示了在渲染 `month` 时 `season` 可以如何被引用到：

```html
<ul data-bind="template: { name: 'seasonTemplate', foreach: seasons, as: 'season' }"></ul>
 
<script type="text/html" id="seasonTemplate">
    <li>
        <strong data-bind="text: name"></strong>
        <ul data-bind="template: { name: 'monthTemplate', foreach: months, as: 'month' }"></ul>
    </li>
</script>
 
<script type="text/html" id="monthTemplate">
    <li>
        <span data-bind="text: month"></span>
        is in
        <span data-bind="text: season.name"></span>
    </li>
</script>
 
<script>
    var viewModel = {
        seasons: ko.observableArray([
            { name: 'Spring', months: [ 'March', 'April', 'May' ] },
            { name: 'Summer', months: [ 'June', 'July', 'August' ] },
            { name: 'Autumn', months: [ 'September', 'October', 'November' ] },
            { name: 'Winter', months: [ 'December', 'January', 'February' ] }
        ])
    };
    ko.applyBindings(viewModel);
</script>
```

> 提示：记得给 `as` 传入 *字符串字面量值* (如： `as: 'season'` 而不是 `as: season`)，因为
> 你是给一个新的变量一个名称，而不是读取一个已有变量的值。

### 注4： 使用 "afterRender", "afterAdd" 和 "afterRemove"

有时你可能希望在由你的模板生成的 DOM 元素上运行自定义的后处理逻辑。
例如，如果你使用像 jquery UI 这样的 JavaScript widgets 类库时，
你可能想解释你的模板输出这样你可以再它上面运行 jQuery UI 命令
以将其某些渲染元素转换成 date picker、sliders 或其它东西。

通常，最好的在 DOM 元素上执行这种后处理的方式是写一个 [自定义绑定](./custom-bindings.md)，
但如果你已经想访问由模板生成的原始 DOM 元素了，你可以使用 `afterRender`。

传入一个函数引用(不管是给函数字面量或视图模型上给定函数的名称)，
且 Knockout 将在渲染或重渲染你的模板时立即调用它。
如果你使用 `foreach`， Knockout 将在每个元素被添加到你的 observable 数组时调用你的 `afterRender` 回调。
如：

```html
<div data-bind='template: { name: "personTemplate",
                            data: myData,
                            afterRender: myPostProcessingLogic }'> </div>
```

 ... 且在你的数据模型上定义一个相应的函数(即，包含 `myData` 的对象)：

```javascript
viewModel.myPostProcessingLogic = function(elements) {
    // "elements" is an array of DOM nodes just rendered by the template
    // You can add custom post-processing logic here
}
```

如果你在使用 `foreach` 且只想在元素特别被添加或移除时被通知，你可以使用 `afterAdd` 和 `afterRemove`。
详细信息可查看 [foreach 绑定](./foreach-bindings.md)。

### 注5：动态选择使用哪个模板

如果你有多个具名模板，你可以给 `name` 选项传入一个 observable。
当 observable 的值被更新时，元素的内容将使用合适的模板被重新渲染。
或者，你可以传入一个回调函数来决定要使用哪个模板。
如果你在使用 `foreach` 模板模式， Knockout 将给数组中每个元素求值该函数，传入该元素的值作为仅有的参数。
否则，该函数将被给定 `data` 选项的值，或者回退为提供你的整个当前模型对象。

例如：

```html
<ul data-bind='template: { name: displayMode,
                           foreach: employees }'> </ul>
 
<script>
    var viewModel = {
        employees: ko.observableArray([
            { name: "Kari", active: ko.observable(true) },
            { name: "Brynn", active: ko.observable(false) },
            { name: "Nora", active: ko.observable(false) }
        ]),
        displayMode: function(employee) {
            // Initially "Kari" uses the "active" template, while the others use "inactive"
            return employee.active() ? "active" : "inactive";
        }
    };
 
    // ... then later ...
    viewModel.employees()[1].active(true); // Now "Brynn" is also rendered using the "active" template.
</script>
```

若你的函数引用 observable 值，则该绑定将在任何这些值变更时更新。这也会导致数据使用相应的模板重渲染。

若你的函数接收第二个参数，则它会接收整个 [绑定上下文](./binding-context.md)。
然后你可以在动态选择一个模板时访问 `$parent` 或任何其他绑定上下文变量。
例如，你可以将前面的代码模板修正为以下代码：

```javascript
displayMode: function(employee, bindingContext) {
    // Now return a template name string based on properties of employee or bindingContext
}
```

### 注6： 使用 jQuery.tmpl ，一个外部基于字符串的模板引擎

在大量主要场景中， Knockout 的原生模板和 `foreach,if,with`和其它控制流绑定将是所有你构造一个任意复杂 UI 的需要。
但有时你希望集成一个外部模板引擎，如 [Underscore 模板引擎](http://documentcloud.github.com/underscore/#template)
或 [jquery.tmpl](http://api.jquery.com/jquery.tmpl/)， Knockout 提供了一种这么做的方式。

默认， Knockout 带有对 jquery.tmpl 的支持。要使用它，你需要引用以下类库，按照这样的顺序：

```html
<!-- First jQuery -->     <script src="http://code.jquery.com/jquery-1.7.1.min.js"></script>
<!-- Then jQuery.tmpl --> <script src="jquery.tmpl.js"></script>
<!-- Then Knockout -->    <script src="knockout-x.y.z.js"></script>
```

然后你可以在你的模板中使用 jQuery.tmpl 语法。如：

```html
<h1>People</h1>
<div data-bind="template: 'peopleList'"></div>
 
<script type="text/html" id="peopleList">
    {{each people}}
        <p>
            <b>${name}</b> is ${age} years old
        </p>
    {{/each}}
</script>
 
<script type="text/javascript">
    var viewModel = {
        people: ko.observableArray([
            { name: 'Rod', age: 123 },
            { name: 'Jane', age: 125 },
        ])
    }
    ko.applyBindings(viewModel);
</script>
```

它能工作，因为 `{{each ...}}` 和 `${ ... }` 是 jQuery.tmpl 语法。
再说，嵌套模板很简单：你可以在模板内部使用 data-bind 属性，
你可以在模板内部使用 `data-bind="template: ..."` 来渲染内嵌的模板。

请注意，在 2011 年 12 月， jQuery.tmpl 就不再开发了。
我们推荐使用 Knockout 的原生基于 DOM 的模板(即 `foreach, if, with` 等绑定)
而不是 jQuery.tmpl 或其它基于字符串的模板引擎。

### 注7： 使用 Underscore.js 模板引擎

[Underscore 模板引擎](http://documentcloud.github.com/underscore/#template) 默认使用 ERB 风格分隔符 (`<%= ... %>`)。
以下是上例在 Underscore 中的代码：

```html
<script type="text/html" id="peopleList">
    <% _.each(people(), function(person) { %>
        <li>
            <b><%= person.name %></b> is <%= person.age %> years old
        </li>
    <% }) %>
</script>
```

以下是 [在 Knockout 中集成 Underscore 模板的简单实现](http://jsfiddle.net/rniemeyer/NW5Vn/)。
集成代码只有 16 行，但它足够支持 Knockout 的 `data-bind` 属性(以及嵌套模板)
及 Knockout [绑定上下文](./binding-context.md) 变量(`$parent, $root` 等)

如果你不喜欢 `<%= ... %>` 分隔符，你可以配置 Underscore 模板引擎以使用任何你选择的分隔符。

### 依赖

* **原生模板** 不需要除 Knockout 本身以外的类库
* **基于字符串模板** 仅在你引用合适的模板引擎时运行，如 jQuery.tmpl 或 Underscore 模板引擎