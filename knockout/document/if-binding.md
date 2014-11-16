# "if" 绑定

### 目的

`if` 绑定让文档中的标签段仅在指定表达式求值为 `true` 
(或像非 `null` 对象或非空数组这样的逻辑真值)时显示出来(并且应用 `data-bind` 属性)。

`if` 的角色类似于 [visible 绑定](./visible-binding.md)。
差别是 `visible` 绑定其包含的标签总是保留在 DOM 中，且总是应用其 `data-bind` 属性 - `visible` 绑定
只是使用 CSS 来切换容器的可见性。
而 `if` 绑定会物理地在 DOM 中添加或移除包含的标签，且仅在表达式为 `true` 时将绑定应用到其后代上。

### 示例1

本例显示 `if` 绑定可以在 observable 值变更时动态添加和移除标签段。

源码： 视图

```html
<label><input type="checkbox" data-bind="checked: displayMessage" /> Display message</label>
 
<div data-bind="if: displayMessage">Here is a message. Astonishing.</div>
```

源码： 视图模型

```javascript
ko.applyBindings({
    displayMessage: ko.observable(false)
});
```

### 示例2

下例中，`<div>` 元素对 "Mercury" 将是空，但对 "Earth" 将会填充。这是因为 Earth 有一个非 `capital` 属性，
而 "Mercury" 中该属性为 `null`。

```html
<ul data-bind="foreach: planets">
    <li>
        Planet: <b data-bind="text: name"> </b>
        <div data-bind="if: capital">
            Capital: <b data-bind="text: capital.cityName"> </b>
        </div>
    </li>
</ul>
 
 
<script>
    ko.applyBindings({
        planets: [
            { name: 'Mercury', capital: null }, 
            { name: 'Earth', capital: { cityName: 'Barnsley' } }        
        ]
    });
</script>
```

重要的是理解 `if` 绑定真的对代码的正常工作很重要。
没有它，当在 "Mercury" 上下文中试图求值 `capital.cityName` 将抛出一个错误，因为 `capital` 时 `null`。
在 JavaScript 中，你不允许求值 `null` 或 `undefined` 值的子属性。

### 参数

* 主参数

  你想求值的表达式。如果它求值为 `true` (或逻辑真值)，其包含的标签将被呈现在文档中，
且其中的任何 `data-bind` 属性都会被应用。如果表达式求值为 `false`，
其包含的标签将从文档中移除，而不会先给其中应用任何绑定。

  如果你的表达式涉及任何 observable 值，表达式将在任何变更时被重新求值。
相应的，`if` 块中的标签将根据表达式变更动态地添加或移除。
`data-bind` 属性将在其重新被添加时重新应用到 **一个新的包含的标签的拷贝**。
   
* 额外参数

   * 没有
   
### 注4：不带容器元素使用 if

有时你可能想控制标签段的存在性而不需要任何容器元素来持有 `if` 绑定。
例如，你可能想控制是否在总是出现的兄弟节点边出现特定的 `<li>` 元素：

```html
<ul>
    <li>This item always appears</li>
    <li>I want to make this item present/absent dynamically</li>
</ul>
```

这里，你不能在 `<ul>` 上放置 `if` (因为它也不应该影响第一个 `<li>`)，
而且你也不能将其放到第二个 `<li>` 外(因为在 `<ul>` 中不允许额外的容器)。

要处理这个，你可以使用 *无容器控制流语法*，它基于注释标签。例如，

```html
<ul>
    <li>This item always appears</li>
    <!-- ko if: someExpressionGoesHere -->
        <li>I want to make this item present/absent dynamically</li>
    <!-- /ko -->
</ul>
```

其中 `<!-- ko -->` 和 `<!-- /ko -->` 注释用作起止标记，在标签的内部定义了一个 "虚拟元素"。
Knockout 能理解虚拟元素语法并且可以像有个真实容器元素进行绑定。

### 依赖

没有，除了核心 Knockout 类库本身。