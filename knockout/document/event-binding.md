# "event" 绑定

### 目的

`event` 绑定让你可以给特定事件添加一个事件处理器，这样当关联的 DOM 元素上的事件被触发时你所选择的 JavaScript 函数将被调用。
它可用于绑定任何事件，如 `keypress`, `mouseover` 或 `mouseout`。

### 示例

```html
<div>
    <div data-bind="event: { mouseover: enableDetails, mouseout: disableDetails }">
        Mouse over me
    </div>
    <div data-bind="visible: detailsEnabled">
        Details
    </div>
</div>
 
<script type="text/javascript">
    var viewModel = {
        detailsEnabled: ko.observable(false),
        enableDetails: function() {
            this.detailsEnabled(true);
        },
        disableDetails: function() {
            this.detailsEnabled(false);
        }
    };
    ko.applyBindings(viewModel);
</script>
```

这样每次你的鼠标指向或移出第一个元素时将调用视图模型上的方法来切换 `detailsEnabled` observable 值。
第二个元素会响应 `detailsEnabled` 的值的变更以显示或隐藏它本身。

### 参数

  * 主参数
  
  你应该传入一个 JavaScript 对象，其中属性名对应于事件名，其值对应于你想给事件绑定的函数。
  
  你可以引用任何 JavaScript 函数 - 它不需要是你视图模型上的函数。
你可以通过写 `event { mouseover: someObject.someFunction }` 来引用任何对象上的函数。

  * 额外参数
  
    * 没有
	
### 注1：给处理器函数传入 "当前项" 作为参数

当调用处理器时， Knockout 会将当前模型的值作为其第一个参数。
这在你给集合中每个元素渲染某些 UI ，且你需要知道哪个元素的 UI 被点击时特别有用。如：

```html
<ul data-bind="foreach: places">
    <li data-bind="text: $data, event: { mouseover: $parent.logMouseOver }"> </li>
</ul>
<p>You seem to be interested in: <span data-bind="text: lastInterest"> </span></p>
 
 <script type="text/javascript">
     function MyViewModel() {
         var self = this;
         self.lastInterest = ko.observable();
         self.places = ko.observableArray(['London', 'Paris', 'Tokyo']);
 
         // The current item will be passed as the first parameter, so we know which place was hovered over
         self.logMouseOver = function(place) {
             self.lastInterest(place);
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
<div data-bind="event: { mouseover: myFunction }">
    Mouse over me
</div>
 
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
<div data-bind="event: { mouseover: function(data, event) { myFunction('param1', 'param2', data, event) } }">
    Mouse over me
</div>
```

现在， KO 会给你的函数字面量传入数据和事件对象，而它们随后被传递给你的处理器。

或者，如果你倾向于在视图中避免函数字面量，你可以使用 [bind](https://developer.mozilla.org/en/JavaScript/Reference/Global_Objects/Function/bind)
函数，它可以给函数引用附加特定的参数值：

```html
<button data-bind="event: { mouseover: myFunction.bind($data, 'param1', 'param2') }">
    Click me
</button>
```

### 注3：允许默认的动作

默认, Knockout 会阻止事件执行任何默认动作。
例如你使用 `event` 绑定来捕获 `input` 标签上的 `keypress` 事件，
你的浏览器只会调用你的处理器函数而不会将键对应的值添加到 `input` 元素的值上。
更常见的示例时使用 [click 绑定](./click-binding.md) ，它在内部使用了该绑定，
你的处理器函数将被调用但不会导航到该链接的 `href` 上。
这是非常有用的默认行为，因为当你使用 `click` 绑定时，通常你是使用该链接来作为操作你的视图模型的 UI 部分，
而不是作为一个常规的到另一个 web 页面的链接。

然而，如果你确实需要执行默认的点击动作，只要从你的 `event` 处理函数中返回 `true` 即可。

### 注4： 阻止事件冒泡

默认， Knockout 将允许点击事件持续冒泡到任何更高层的事件处理器。
如，当你的元素和该元素的某个父元素都能处理相同的 `mouseover` 事件，
则两个事件处理器都会被触发。如果需要，你可以通过包含一个额外的名为 `youreventBubble` 的绑定，
并给它传入 `false` 来阻止事件冒泡。如下例：

```html
<div data-bind="event: { mouseover: myDivHandler }">
    <button data-bind="event: { mouseover: myButtonHandler }, mouseoverBubble: false">
        Click me
    </button>
</div>
```

通常，这里的 `myButtonHandler` 将先被调用，然后点击事件会被冒泡到 `myDivHandler`。
然后我们添加了带 `false` 值的 `mouseoverBubble`，它在事件通过 `myButtonHandler` 后阻止了它的冒泡。

### 依赖

没有，除了核心 Knockout 类库本身。