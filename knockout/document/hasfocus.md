# "hasFocus" 绑定

### 目的

`hasFocus` 绑定将一个 DOM 元素的 focus 状态链到视图模型的属性上。
它是一个双向绑定，因此：

* 如果你将视图模型属性设置为 `true` 或 `false`，关联的元素将得到焦点或失去焦点
* 如果用户手动给关联元素得到或失去焦点，视图模型属性将相应地被设置为 `true` 或 `false`

这在你构建复杂的表单，其中可编辑元素动态地出现，且你希望控制用户应该在哪里开始写，
或响应回车的位置，时非常有用。

### 示例1：基础示例

这个示例简单地在文本框当前得到焦点时显示一条消息，且你可以使用按钮来编程式地触发焦点。

源码： 视图

```html
<input data-bind="hasFocus: isSelected" />
<button data-bind="click: setIsSelected">Focus programmatically</button>
<span data-bind="visible: isSelected">The textbox has focus</span>
```

```javascript
var viewModel = {
    isSelected: ko.observable(false),
    setIsSelected: function() { this.isSelected(true) }
};
ko.applyBindings(viewModel);
```

### 示例2： 点击进入编辑

因为 `hasFocus` 绑定可双向运作(设置关联的值来使元素得到或失去焦点；在元素上得到或失去焦点也会设置关联的值)，
它可以成为触发 "编辑" 模式的方便地方式。
这里， UI 根据模型的 `editing` 属性值来显示一个 `<span>` 或一个 `<input>` 元素。
`<input>` 元素失去焦点会将 `editing` 设置为 `false`，这样 UI 又会离开 "编辑" 模式。

源码：视图

```html
<p>
    Name: 
    <b data-bind="visible: !editing(), text: name, click: edit">&nbsp;</b>
    <input data-bind="visible: editing, value: name, hasFocus: editing" />
</p>
<p><em>Click the name to edit it; click elsewhere to apply changes.</em></p>
```

源码：视图模型

```javascript
function PersonViewModel(name) {
    // Data
    this.name = ko.observable(name);
    this.editing = ko.observable(false);
         
    // Behaviors
    this.edit = function() { this.editing(true) }
}
 
ko.applyBindings(new PersonViewModel("Bert Bertington"));
```

### 参数

  * 主参数
  
  传入 `true`(或其他逻辑真值)来使关联元素得到焦点。否则，关联元素将失去焦点。
  
  当用户手动让元素得到或失去焦点，你的值也会被相应地设置为 `true` 或 `false`。
  
  如果你提供的值是 observable， `hasFocus` 绑定将会在 observable 值变更时更新元素焦点状态。

  * 额外参数
  
    * 没有

### 依赖

没有，除了核心 Knockout 类库本身。