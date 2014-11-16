# "foreach" 绑定

### 目的

`foreach` 绑定给数组中的每个元素一段标签的副本，且给每个副本绑定到相应的标签上。
这在渲染列表或表格时非常有用。

假设你的数组是一个 [observable array](./observableArrays.md)，当你后续添加、移除或重排数组元素时，
该绑定会高效地更新匹配的 UI - 插入或移除标签的拷贝或重排已有的 DOM 元素，而不会影响任何其他 DOM 元素。
这比在每次数组变更时重新生成整个 `foreach` 输出要快得多。

当然，你可以任意地嵌套 `foreach` 绑定及其他像 `if` 和 `with` 这样的控制流绑定。

### 示例1：迭代数组

该例使用 `foreach` 生成一个只读的表格，其中每个数组元素对应表格中的一行。

```html
<table>
    <thead>
        <tr><th>First name</th><th>Last name</th></tr>
    </thead>
    <tbody data-bind="foreach: people">
        <tr>
            <td data-bind="text: firstName"></td>
            <td data-bind="text: lastName"></td>
        </tr>
    </tbody>
</table>

<script type="text/javascript">
    ko.applyBindings({
        people: [
            { firstName: 'Bert', lastName: 'Bertington' },
            { firstName: 'Charles', lastName: 'Charlesforth' },
            { firstName: 'Denise', lastName: 'Dentiste' }
        ]
    });
</script>
```

### 示例2：动态添加/移除示例

以下示例显示，如果你的数组是一个 observable ，则 UI 会和数组的变更保持同步。

源码：视图

```html
<h4>People</h4>
<ul data-bind="foreach: people">
    <li>
        Name at position <span data-bind="text: $index"> </span>:
        <span data-bind="text: name"> </span>
        <a href="#" data-bind="click: $parent.removePerson">Remove</a>
    </li>
</ul>
<button data-bind="click: addPerson">Add</button>
```

源码：视图模型

```javascript
function AppViewModel() {
    var self = this;
 
    self.people = ko.observableArray([
        { name: 'Bert' },
        { name: 'Charles' },
        { name: 'Denise' }
    ]);
 
    self.addPerson = function() {
        self.people.push({ name: "New at " + new Date() });
    };
 
    self.removePerson = function() {
        self.people.remove(this);
    }
}
 
ko.applyBindings(new AppViewModel());
```

### 参数

* 主参数

  传入你想进行迭代的数组。该绑定会给每个元素输出一段标签。
  
  或者，传入一个 JavaScript 对象字面量，其中带有一个名为 `data` 的属性，它持有进行迭代的数组。
该对象字面量也可能有其他属性，如 `afterAdd` 或 `includeDestroyed` - 详见以下额外选项及其用法示例。

  如果你提供的数组是一个 observable，`foreach` 绑定会响应后续数组内容变更，
以在 DOM 中添加或移除相应标签段。
   
* 额外参数

   * 没有
   
### 注1：使用 $data 来引用每个数组元素

如上例所示，在 `foreach` 绑定中可以引用数组元素上的属性。如，示例1 引用了每个数组元素上的 `firstName` 和 `lastName` 属性。

但如果你想引用数组元素本身(而不只是其属性)呢？
这时候你就需要使用 [特殊的上下文属性](./binding-context.md) `$data`。
在每个 `foreach` 块中，它意味着 "当前项"。如：

```html
<ul data-bind="foreach: months">
    <li>
        The current item is: <b data-bind="text: $data"></b>
    </li>
</ul>
 
<script type="text/javascript">
    ko.applyBindings({
        months: [ 'Jan', 'Feb', 'Mar', 'etc' ]
    });
</script>
```

如果你想的话，你也可以使用 `$data` 作为引用每个元素上的属性时的前缀。如示例1 可以重写为：

```html
<td data-bind="text: $data.firstName"></td>
```

但这没有必要，因为 `firstName` 默认会在 `$data` 上下文中被求值。

### 注2：使用 $index、$parent 和其它上下文属性

如示例2中可见，可以使用 `$index` 来引用当前数组元素的基于 0 的索引值。
`$index` 时一个 observable 且在每次项的索引改变时(如项被添加或从数组中移除)更新。

类似的，你可以使用 `$parent` 来引用 `foreach` 外部的数据，如：

```html
<h1 data-bind="text: blogPostTitle"></h1>
<ul data-bind="foreach: likes">
    <li>
        <b data-bind="text: name"></b> likes the blog post <b data-bind="text: $parent.blogPostTitle"></b>
    </li>
</ul>
```

更多关于 `$index` 和 `$parent` 这样上下文属性信息，请查看 [绑定上下文属性](./binding-context.md)。

### 注3：用 "as" 给 "foreach" 项定义别名

如注1 所描述，你可以使用 `$data` 上下文变量来引用每个数组元素。
然而在某些情况下，使用 `as` 选项来给当前项一个更具有描述性的名称会更有用，如：

```html
<ul data-bind="foreach: { data: people, as: 'person' }"></ul>
```

现在这个 `foreach` 循环中任意地方，绑定都可以从 `people` 数组中引用 `person` 来访问被渲染的当前数组项。
这在你需要嵌套 `foreach` 块且需要在层级中引用更高层定义的一个项时特别有用。如：

```html
<ul data-bind="foreach: { data: categories, as: 'category' }">
    <li>
        <ul data-bind="foreach: { data: items, as: 'item' }">
            <li>
                <span data-bind="text: category.name"></span>:
                <span data-bind="text: item"></span>
            </li>
        </ul>
    </li>
</ul>
 
<script>
    var viewModel = {
        categories: ko.observableArray([
            { name: 'Fruit', items: [ 'Apple', 'Orange', 'Banana' ] },
            { name: 'Vegetables', items: [ 'Celery', 'Corn', 'Spinach' ] }
        ])
    };
    ko.applyBindings(viewModel);
</script>
```

提示：记得给 `as` 传递一个 *字符串字面量* (如，`as: 'category'` ，而不是 `as: category`)，
因为你是要给新变量一个名称，而不是读取某个已存在的变量。

### 注4：不带容器元素使用 foreach

某些情况下，你可以想重复某段标签，但却没有任何容器元素来放置 `foreach` 绑定。
如，你可能想生成以下片段：

```html
<ul>
    <li class="header">Header item</li>
    <!-- The following are generated dynamically from an array -->
    <li>Item A</li>
    <li>Item B</li>
    <li>Item C</li>
</ul>
```

本例中，没有任何地方可以放置一个常规的 `foreach` 绑定。你不能把它放在 `<ul>` 上，
(因为这样你会重复头部项)，你也不能放在 `<ul>` 中的某个元素 (因为在 `<ul>` 中仅允许 `<li>` 元素)。

要处理这个，你可以使用 *无容器控制流语法*，它基于注释标签。如：

```html
<ul>
    <li class="header">Header item</li>
    <!-- ko foreach: myItems -->
        <li>Item <span data-bind="text: $data"></span></li>
    <!-- /ko -->
</ul>
 
<script type="text/javascript">
    ko.applyBindings({
        myItems: [ 'A', 'B', 'C' ]
    });
</script>
```

其中 `<!-- ko -->` 和 `<!-- /ko -->` 注释用作起止标记，在标签的内部定义了一个 "虚拟元素"。
Knockout 能理解虚拟元素语法并且可以像有个真实容器元素进行绑定。

### 注5：数组变更如何被侦测和处理

当你修改模型数组的内容时(通过添加、移动或删除元素)， `foreach` 绑定使用了一种高效的比较算法
来指出什么被变更了，因此它可以相应地更新 DOM。
这意味着它可以处理任意变更的组合。

* 当你 **添加** 数组元素时， `foreach` 会渲染模板的新拷贝并将它们插入到已有的 DOM 中。
* 当你 **删除** 数组元素时， `foreach` 会简单地移除相应的 DOM 元素。
* 当你 **重排** 数组元素时(保留相同的对象实例)，`foreach` 通常只是将相应的 DOM 元素移动到其新的位置。

注意重排侦测不是确保的：为保证算法快速完成，它对侦测小量数组元素的 "简单" 移动做了优化。
如果算法侦测到很多同时的重排连同非相关的插入和删除，则为速度因素它会选择
将重排看做 "删除" 外加 "添加" 而不是简单地 "移动"，这时相应的 DOM 元素会被移除并重建。
多数开发者不会遇到这种边界情况，而且即使遇到了，终端用户体验常常是一样的。

### 注6：destoryed 元素默认是隐藏的

有时你可能想将数组元素标记为被删除，而不用实际移除该记录。
这称为 *非破坏性删除*。其详细情况可查看 [observableArray](./observableArrays.md) 的 `destroy` 函数。

默认，`foreach` 绑定会跳过(即隐藏)被标记为已 destoryed 的任何数组元素。
如果你想显示 destoryed 元素，可以使用 `includeDestroyed` 选项，如：

```html
<div data-bind='foreach: { data: myArray, includeDestroyed: true }'>
    ...
</div>
```

### 注7：给生成的 DOM 元素进行后加工或添加动画

如果你需要给生成的 DOM 元素运行其他自定义逻辑，你可以使用任何下述的
`afterRender/afterAdd/beforeRemove/beforeMove/afterMove` 回调。

> 注：这些回调 *仅* 用于在列表相关的变更触发动画效果。
> 如果你的目的实际上是给新的 DOM 元素在其被添加时附加其它的行为，
> (如事件处理器或激活第三方 UI 控制)，则你将新的行为实现为 [自定义绑定](./custom-bindings.md) 会更简单，
> 因为这样你可以再任何地方使用该行为，而和 `foreach` 绑定无关。

这里有个简单示例，它使用了 `afterAdd` 来给新加项应用经典的 "黄色淡出" 效果。
它需要 [jQuery Color 插件](https://github.com/jquery/jquery-color) 来启用背景色的动画效果。

```html
<ul data-bind="foreach: { data: myItems, afterAdd: yellowFadeIn }">
    <li data-bind="text: $data"></li>
</ul>
 
<button data-bind="click: addItem">Add</button>
 
<script type="text/javascript">
    ko.applyBindings({
        myItems: ko.observableArray([ 'A', 'B', 'C' ]),
        yellowFadeIn: function(element, index, data) {
            $(element).filter("li")
                      .animate({ backgroundColor: 'yellow' }, 200)
                      .animate({ backgroundColor: 'white' }, 800);
        },
        addItem: function() { this.myItems.push('New item'); }
    });
</script>
```

完整细节：

* `afterRender` -- 在每次 `foreach` 代码块被复制并插入文档时被调用，不管是 `foreach` 首次初始化，
还是新的元素被后续添加到关联的数组上。Knockout 将给你的回调提供以下参数：

  1. 一个被插入 DOM 元素的数组
  2. 它们所被绑定的数据项
  
* `afterAdd` -- 类似于 `afterRender`,除了它仅在新的元素被添加到你的数组时调用。
(而当 `foreach` 首次迭代数组初始内容时不调用) 常见的 `afterAdd` 用法是调用像 jQuery 的 `$(domNode).fadeIn()`
这样的方法，这样你在项被添加时能得到动画转换效果。Knockout 会给你的回调提供以下参数：

  1. 一个被添加到文档的 DOM 节点
  2. 被添加元素的索引
  3. 被添加的数组元素

* `beforeRemove` -- 在数组被移除时但在相应的 DOM 节点被移除前调用。如果你指定了 `beforeRemove` 回调，
则 *你需要负责移除 DOM 节点*。这里场景用法是调用 jQuery 的 `$(domNode).fadeOut()` 来动态地移除相应的 DOM 节点 -- 这时，
Knockout 不知道它应该多快才可以物理地移除 DOM 节点(谁知道你的动画需要多长时间？)，
因此你需要自行移除它们。 Knockout 会给你的回调提供以下参数：

  1. 一个应被移除的 DOM 节点
  2. 被移除元素的索引
  3. 被移除的数组元素
  
* `beforeMove` -- 当数组中的元素的位置有变更但相应的 DOM 节点还没有被移动时被调用。
注意 `beforeMove` 应用到所有索引值有变更的数组元素上，因此你在数组头部插入一个项，则
该回调(如果指定了的话)会在所有其它元素上都会触发，因为它们的索引值都被加了 1 。
你可以使用 `beforeMove` 来保存被影响元素的原始屏幕坐标，这样你可以再 `afterMove` 中给其移动添加动画。
Knockout 会给你的回调提供以下参数：

  1. 一个应被移动的 DOM 节点
  2. 被移动元素的索引
  3. 被移动的数组元素
  
* `afterMove` -- 当数组中的元素的位置有变更且 `foreach` 绑定更新了相应的 DOM 以后。
注意 `beforeMove` 应用到所有索引值有变更的数组元素上，因此你在数组头部插入一个项，则
该回调(如果指定了的话)会在所有其它元素上都会触发，因为它们的索引值都被加了 1 。
Knockout 会给你的回调提供以下参数：

  1. 一个应被移动的 DOM 节点
  2. 被移动元素的索引
  3. 被移动的数组元素
  
`afterAdd` 和 `beforeRemove` 的示例可查看 [动画转换](http://knockoutjs.com/examples/animatedTransitions.html)。

### 依赖

没有，除了核心 Knockout 类库本身。