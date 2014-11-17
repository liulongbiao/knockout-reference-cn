# "enable" 绑定

### 目的

`enable` 绑定导致关联的 DOM 元素仅在参数值为 `true` 时启用。
它在像 `input`、`select` 和 `textarea` 这样的表单元素中很有用。

### 示例

```html
<p>
    <input type='checkbox' data-bind="checked: hasCellphone" />
    I have a cellphone
</p>
<p>
    Your cellphone number:
    <input type='text' data-bind="value: cellphoneNumber, enable: hasCellphone" />
</p>
 
<script type="text/javascript">
    var viewModel = {
        hasCellphone : ko.observable(false),
        cellphoneNumber: ""
    };
</script>
```

这里， "Your cellphone number" 文本框默认初始化是禁用的。
它仅在用户选中了 "I have a cellphone" 标记的选择框时被启用。

### 参数

  * 主参数
  
  一个用于控制关联 DOM 元素是否应被启用的值。
  
  非布尔值被松散地解释为布尔值。如，`0` 和 `null` 被当作 `false`，而 `21` 和非 `null` 对象被当作 `true`。
  
  如果你的参数引用一个 observable 值，该绑定会在 observable 值变更时更新启用/禁用状态。
如果参数没有引用一个 observable 值，它仅会设置该状态一次，后续不会再更新。

  * 额外参数
  
    * 没有
	
### 注：使用任意 JavaScript 表达式

这里不限于引用变量 - 你可以引用任意的表达式来控制元素的启用状态。如：

```html
<button data-bind="enable: parseAreaCode(viewModel.cellphoneNumber()) != '555'">
    Do something
</button>
```

### 依赖

没有，除了核心 Knockout 类库本身。