# "html" 绑定

### 目的

`html` 绑定让关联的 DOM 元素可以显示你的参数所指定的 HTML。

通常这在你的视图模型中的值实际上是你所希望渲染的 HTML 标记的字符串时非常有用。

示例

```html
<div data-bind="html: details"></div>
 
<script type="text/javascript">
    var viewModel = {
        details: ko.observable() // Initially blank
    };
    viewModel.details("<em>For further details, view the report <a href='report.html'>here</a>.</em>"); // HTML content appears
</script>
```

### 参数

* 主参数

  KO 清除以前的的内容并使用 jQuery 的 `html` 函数或在 jQuery 不可用时通过将字符串解析为 HTML 节点并将每个节点作为
该元素的子节点附着来将元素的内容设置为你的参数值。

  如果该参数是一个 observable 值，该绑定会在值变更时更新元素的内容。若参数不是 observable，
它仅会设置元素的内容一次且再也不会更新。

  如果你提供了数字或字符串以为的东西(如传入了一个对象或数组)，其 `innerHTML` 将等价于 `yourParameter.toString()`。

* 额外参数

   * 没有
   
### 注：关于 HTML 转码

鉴于该绑定使用 `innerHTML` 来设置你的元素的内容，你应该小心不在不受信的模型值里使用它，
因为这样可能开启脚本注入攻击的可能性。
如果你不能保证内容显示的安全性(如它基于存储于你的数据库中一个不同用户的输入)，
则你可以使用 [text 绑定](./text-binding.md)，它将转而使用 `innerText` 或 `textContent` 来设置元素的文本值。

### 依赖

没有，除了核心 Knockout 类库本身。