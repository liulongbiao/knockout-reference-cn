# "submit" 绑定

### 目的

`event` 绑定添加了一个处理器，这样当关联的 DOM 元素上的提交事件被触发时你所选择的 JavaScript 函数将被调用。
通常你只会在 `form` 元素上使用它。

当你在表单上使用 `submit` 绑定时， Knockout 会阻止浏览器默认的表单提交动作。
或者说，浏览器会调用你的处理器而不会将表单提交给服务器。
这是非常有用的默认行为，因为当你使用 `submit` 绑定时，通常你是将该表单用作你的视图模型的接口，
而不是作为一个常规的 HTML 表单。
如果你确实想让表单像常规的 HTML 表单一样提交，只要在你的 `submit` 处理器中返回 `true` 即可。

### 示例

```html
<form data-bind="submit: doSomething">
    ... form contents go here ...
    <button type="submit">Submit</button>
</form>
 
<script type="text/javascript">
    var viewModel = {
        doSomething : function(formElement) {
            // ... now do something
        }
    };
</script>
```

如本例所演示， KO 将表单元素作为参数传递给你的 submit 处理器函数。
如果你想，你可以忽略该参数，或者存在大量你可能使用它的方式，如：

* 从表单元素中提取额外的数据或状态
* 使用类似以下代码片段 `if ($(formElement).valid()) { /* do something */ }` 
来触发 [jQuery Validation](https://github.com/jzaefferer/jquery-validation) 这样的 UI 层验证，

### 为何不只是在提交按钮上放置一个 click 处理器

除了在表单上使用 `submit`，你也可以在提交按钮上使用 `click`。
然而，`submit` 还有一个优势就是它会捕获提交表单的其它的方式，
如在某个文本框中按下 *enter* 键。

### 参数

  * 主参数
  
  你想给元素的 `submit` 事件所绑定的函数。
  
  你可以引用任何 JavaScript 函数 - 它不需要是你视图模型上的函数。
你可以通过写 `submit: someObject.someFunction` 来引用任何对象上的函数。

  在你的视图模型上的函数有些特殊，因为你可以通过名称来引用它们，如你可以写 `submit: doSomething `
而不需要写 `submit: viewModel.doSomething` (虽然技术上也是有效的)

  * 额外参数
  
    * 没有
	
### 注

关于如何给你的 submit 处理器函数传递参数或在调用不处于你的视图模型中的函数时如何控制 `this` 句柄，
可以查看 [click 绑定](./click-binding.md) 中相关注解。
那个页面中所有的注解对 `submit` 处理器也适用。

### 依赖

没有，除了核心 Knockout 类库本身。