# "with" 绑定

### 目的

`with` 绑定创建一个新的 [绑定上下文](./binding-context.md)，因此后代元素被绑定特定对象的上下文上。

当然，你可以任意嵌套 `with` 绑定和其它诸如 `if` 和 `foreach` 这样的控制流绑定。

### 示例1

这是切换绑定上下文到一个子对象的基本示例。注意在 `data-bind` 属性中，用 `coords` 前缀到
`latitude` 和 `longitude` 不再是必需的，因为其绑定上下文被切换到了 `coords`。

```html
<h1 data-bind="text: city"> </h1>
<p data-bind="with: coords">
    Latitude: <span data-bind="text: latitude"> </span>,
    Longitude: <span data-bind="text: longitude"> </span>
</p>
 
<script type="text/javascript">
    ko.applyBindings({
        city: "London",
        coords: {
            latitude:  51.5001524,
            longitude: -0.1262362
        }
    });
</script>
```

### 示例2

这个交互式示例演示了：

* `with` 绑定会根据关联的值是否是 `null/undefined` 动态添加或移除后代元素
* 如果你想访问父绑定上下文中的 数据/函数，你可以使用 [像 $parent 和 $root 这样的特殊上下文属性](./binding-context.md)。

源码：视图

```html
<form data-bind="submit: getTweets">
    Twitter account:
    <input data-bind="value: twitterName" />
    <button type="submit">Get tweets</button>
</form>
 
<div data-bind="with: resultData">
    <h3>Recent tweets fetched at <span data-bind="text: retrievalDate"> </span></h3>
    <ol data-bind="foreach: topTweets">
        <li data-bind="text: text"></li>
    </ol>
 
    <button data-bind="click: $parent.clearResults">Clear tweets</button>
</div>
```

源码：视图模型

```javascript
function AppViewModel() {
    var self = this;
    self.twitterName = ko.observable('@example');
    self.resultData = ko.observable(); // No initial value
 
    self.getTweets = function() {
        var name = self.twitterName(),
            simulatedResults = [
                { text: name + ' What a nice day.' },
                { text: name + ' Building some cool apps.' },
                { text: name + ' Just saw a famous celebrity eating lard. Yum.' }
            ];
 
        self.resultData({ retrievalDate: new Date(), topTweets: simulatedResults });
    }
 
    self.clearResults = function() {
        self.resultData(undefined);
    }
}
 
ko.applyBindings(new AppViewModel());
```

### 参数

* 主参数

  你想给后代元素用作绑定上下文的对象。
  
  如果你提供的表达式求值为 `null` 或 `undefined`，后代元素将完全不会被绑定，相反是会从文档中被移除。
  
  如果你提供的表达式涉及任何 observable 值，表达式将在任何这些 observable 变更时重新求值。
然后后代元素将被清除，且 **一个新的标签的拷贝** 将被添加到你的文档且新的求值结果将被作为上下文绑定。
   
* 额外参数

   * 没有
   
### 注4：不带容器元素使用 with

就像其他项 `if` 和 `foreach` 这样的控制流元素，你可以不用任何容器元素来 `with`。
这在你需要在仅为了 `with` 绑定而引入新的容器元素却不合法的地方使用 `with` 非常有用。
详见 [if 绑定](./if-binding.md) 或 [foreach 绑定](./foreach-binding.md) 相关文档。

例如：

```html
<ul>
    <li>Header element</li>
    <!-- ko with: outboundFlight -->
        ...
    <!-- /ko -->
    <!-- ko with: inboundFlight -->
        ...
    <!-- /ko -->
</ul>
```

其中 `<!-- ko -->` 和 `<!-- /ko -->` 注释用作起止标记，在标签的内部定义了一个 "虚拟元素"。
Knockout 能理解虚拟元素语法并且可以像有个真实容器元素进行绑定。

### 依赖

没有，除了核心 Knockout 类库本身。