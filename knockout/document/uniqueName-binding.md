# "uniqueName" 绑定

### 目的

`uniqueName` 绑定确保关联的 DOM 元素又给非空的 `name` 属性。
若该 DOM 元素没有 `name` 属性，该绑定会给它一个兵设置为某个唯一的字符串值。

通常你并不需要它。它在以下一些特殊场景中有用，如：

* 其它技术可能依赖于特定元素具有 name 的假设，即使名称可能在使用 KO 时是不相干的。
例如 [jQuery Validation](http://docs.jquery.com/Plugins/validation) 当前仅会验证具有 name 属性的元素。
要在 Knockout UI 中使用它，有时需要应用 `uniqueName` 绑定避免让 jQuery Validation 疑惑。
见 [在 KO 中使用 jQuery Validation 的示例](http://knockoutjs.com/examples/gridEditor.html)。
* IE6 在单选框没有 name 属性时不允许它们被选中。
多数时候这是不相干的，因为你的单选框元素将会有一个 name 属性将它们放置到互斥组中。
然而，有时因为它在你的用例中并不需要而没有添加 name 属性，
KO 将在内部对这些元素使用 uniqueName 来确保它们可被选中。

### 示例

```html
<input data-bind="value: someModelProperty, uniqueName: true" />
```

### 参数

  * 主参数
  
  传入 `true`(或求值为 true 的值)来启用 uniqueName 绑定，如上例。

  * 额外参数
  
    * 没有

### 依赖

没有，除了核心 Knockout 类库本身。