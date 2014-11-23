# "textInput" 绑定

### 目的

`textInput` 绑定将一个文本框(`<input>`)或文本区(`<textarea>`)链到视图模型的属性上，
提供了在视图模型属性和元素值之间的双向更新。
不像 `value` 绑定， `textInput` 提供了从 DOM 上各种类型的用户输入的持续更新，
包括自动完成，拖放和剪贴板事件。

### 示例

```html
<p>Login name: <input data-bind="textInput: userName" /></p>
<p>Password: <input type="password" data-bind="textInput: userPassword" /></p>
 
ViewModel:
<pre data-bind="text: ko.toJSON($root, null, 2)"></pre>
 
<script>
    ko.applyBindings({
        userName: ko.observable(""),        // Initially blank
        userPassword: ko.observable("abc")  // Prepopulate
    });
</script>
```

### 参数

  * 主参数
  
  KO 设置元素的文本内容为你的参数的值。任何以前的值将会被覆盖。
  
  如果你的参数是一个 observable 值，该绑定会在 observable 值变更时更新元素的值。
如果参数没有引用一个 observable 值，它仅会设置元素的值一次，后续不会再更新。

  如果你提供的参数不是数字或字符串(如传入了对象或数组)，其显示文本将等价于 `yourParameter.toString()`。
(这通常不是很有用，所以最好提供字符串或数字值)

  当用户编辑关联表单控件上的值时， KO 将会更新视图模型上的属性。
KO 会在值因用户或任何 DOM 事件而被改变时总是会试图更新你的视图模型，

  * 额外参数
  
    * 没有
	
### 注1：`textInput` vs `value` 绑定

尽管 [value 绑定](./value-binding.md) 也可以执行文本框和视图模型属性间的双向绑定，
当你在需要即时动态更新时应更倾向于 `textInput`。其主要差异为：

* **即时更新**
  
  `value`，默认情况下，仅会在用户将焦点移出文本框时更新模型。
`textInput` 会在每次敲击键盘或其它文本输入机制(如剪切或粘贴文本，它们不需要触发任何焦点变更事件)
发生时立即更新模型。

* **浏览器事件怪异行为处理**

  浏览器在非常规的输入机制，如剪切、粘贴或接收自动完成建议等所触发的事件上具有高度的不一致性。
`value` 绑定即时在使用像 `valueUpdate: afterkeydown` 这样的额外选项来在特定事件发生时获取更新，
也无法覆盖所有浏览器上所有文本输入场景。

  `textInput` 绑定被特别设计为处理大量浏览器怪异行为，
以提供甚至在非常规文本输入方法下一致且即时的模型更新。

不要试图在同一个元素上连同使用 `value` 和 `textInput` 绑定，它不能得到任何有用的东西。

### 依赖

没有，除了核心 Knockout 类库本身。