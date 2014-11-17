# "click" 绑定

### 目的

`click` 绑定添加了一个事件处理器，这样当关联的 DOM 元素被点击时你所选择的 JavaScript 函数将被调用。
它常用在像 `button`、`input` 和 `a` 这样的元素上，但实际上可用于任何可见的 DOM 元素。

### 示例

```html
<div>
    You've clicked <span data-bind="text: numberOfClicks"></span> times
    <button data-bind="click: incrementClickCounter">Click me</button>
</div>
 
<script type="text/javascript">
    var viewModel = {
        numberOfClicks : ko.observable(0),
        incrementClickCounter : function() {
            var previousCount = this.numberOfClicks();
            this.numberOfClicks(previousCount + 1);
        }
    };
</script>
```

每次点击按钮时，它都会调用视图模型上的 `incrementClickCounter()`，然后它会更新视图模型状态，
随之导致 UI 的更新。

### 参数

  * 主参数
  
  你想绑定到元素的 `click` 事件的函数。
  
  你可以引用任何 JavaScript 函数 - 它不需要是你视图模型上的函数。
你可以通过写 `click: someObject.someFunction` 来引用任何对象上的函数。

  * 额外参数
  
    * 没有
	
### 注1：给处理器函数传入 "当前项" 作为参数

当调用处理器时， Knockout 会将当前模型的值作为其第一个参数。
这在你给集合中每个元素渲染某些 UI ，且你需要知道哪个元素的 UI 被点击时特别有用。如：

```html
<ul data-bind="foreach: places">
    <li>
        <span data-bind="text: $data"></span>
        <button data-bind="click: $parent.removePlace">Remove</button>
    </li>
</ul>
 
 <script type="text/javascript">
     function MyViewModel() {
         var self = this;
         self.places = ko.observableArray(['London', 'Paris', 'Tokyo']);
 
         // The current item will be passed as the first parameter, so we know which place to remove
         self.removePlace = function(place) {
             self.places.remove(place)
         }
     }
     ko.applyBindings(new MyViewModel());
</script>
```

本例中需要注意两点：

* 如果你在嵌套的 [绑定上下文](./binding-context.md) 内部，如处于 `foreach` 或 `with` 块中，
但你的处理器函数处于根视图模型或其它父上下文中，你需要使用一个前缀如 `$parent` 或 `$root` 来定位处理器函数。
* 在你的视图模型中，定义一个 `self`(或其它变量名称)来作为 `this` 的别名常常很有用。
这样可以避免在事件处理器或 Ajax 请求回调中 `this` 被重定义。

### 注2：访问事件对象或传入更多参数

某些场景中，你需要访问和你的点击事件相关的 DOM 事件对象。
Knockout 将会把该事件作为你的函数的第二个参数传入，如下例：

```html
<button data-bind="click: myFunction">
    Click me
</button>
 
 <script type="text/javascript">
    var viewModel = {
        myFunction: function(data, event) {
            if (event.shiftKey) {
                //do something different when user has shift key down
            } else {
                //do normal action
            }
        }
    };
    ko.applyBindings(viewModel);
</script>
```

如果你需要传入更多参数，一种方式是将你的处理器封装在一个函数字面量中来接收参数，如下例：

```html
<button data-bind="click: function(data, event) { myFunction('param1', 'param2', data, event) }">
    Click me
</button>
```

现在， KO 会给你的函数字面量传入数据和事件对象，而它们随后被传递给你的处理器。

或者，如果你倾向于在视图中避免函数字面量，你可以使用 [bind](https://developer.mozilla.org/en/JavaScript/Reference/Global_Objects/Function/bind)
函数，它可以给函数引用附加特定的参数值：

```html
<button data-bind="click: myFunction.bind($data, 'param1', 'param2')">
    Click me
</button>
```

### 注3：允许默认的点击动作

默认, Knockout 会阻止点击事件执行任何默认动作。
这意味着如果你在 `a` 标签上使用 `click` 绑定，你的浏览器只会调用你的处理器函数而不会导航到该链接的 `href` 上。
这是非常有用的默认行为，因为当你使用 `click` 绑定时，通常你是使用该链接来作为操作你的视图模型的 UI 部分，
而不是作为一个常规的到另一个 web 页面的链接。

然而，如果你确实需要执行默认的点击动作，只要从你的 `click` 处理函数中返回 `true` 即可。

### 注4： 阻止事件冒泡

默认， Knockout 将允许点击事件持续冒泡到任何更高层的事件处理器。
如，当你的元素和该元素的某个父元素都能处理 `click` 事件，
则两个 click 处理器都会被触发。如果需要，你可以通过包含一个额外的名为 `clickBubble` 的绑定，
并给它传入 `false` 来阻止事件冒泡。如下例：

```html
<div data-bind="click: myDivHandler">
    <button data-bind="click: myButtonHandler, clickBubble: false">
        Click me
    </button>
</div>
```

通常，这里的 `myButtonHandler` 将先被调用，然后点击事件会被冒泡到 `myDivHandler`。
然后我们添加了带 `false` 值的 `clickBubble`，它在事件通过 `myButtonHandler` 后阻止了它。

### 依赖

没有，除了核心 Knockout 类库本身。