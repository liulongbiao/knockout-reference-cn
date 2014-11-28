# 使用非唐突的事件处理器

多数情况下， data-bind 属性提供了清晰足够的方式来绑定视图模型。
然而，事件处理是一个常会导致啰嗦的 data-bind 属性的区域，
因为匿名函数通常是推荐的传参的技术。如：

```html
<a href="#" data-bind="click: function() { viewModel.items.remove($data); }">
    remove
</a>
```

作为替代方案， Knockout 提供了两个帮助函数，让你可以取得和某个 DOM 元素上相关的数据。

* `ko.dataFor(element)` - 返回该元素的绑定可用的数据对象
* `ko.contextFor(element)` - 返回该 DOM 元素上可用的整个 [绑定上下文](./binding-context.md)

这些帮助函数可用于非唐突地添加使用了像 jQuery 的 `bind` 或 `click` 这样的事件处理器上。
以上函数添加到每个具有 `remove` 类的链接上，如：

```javascript
$(".remove").click(function () {
    viewModel.items.remove(ko.dataFor(this));
});
```

更进一步，该技术可用于支持事件委托。 jQuery 的 `live/delegate/on` 函数是简单地可行方式：

```javascript
$(".container").on("click", ".remove", function() {
    viewModel.items.remove(ko.dataFor(this));
});
```

现在，单个事件处理器被添加到更高层级，且可以处理任何具有 `remove` 类的链接。
这种方法还有一个好处是能够自动处理被动态添加到文档上的额外的链接。
(比如作为被添加到一个 observableArray 的项的结果)

### 活动示例： 嵌套子节点

本例显示了通过单个处理器给每个链接类型非唐突地添加在多个层级的父子节点间的 "添加" 和 "删除"。

源码： 视图

```html
<ul id="people" data-bind='template: { name: "personTmpl", foreach: people }'>
</ul>
 
<script id="personTmpl" type="text/html">
    <li>
        <a class="remove" href="#"> x </a>
        <span data-bind='text: name'></span>
        <a class="add" href="#"> add child </a>
        <ul data-bind='template: { name: "personTmpl", foreach: children }'></ul>
    </li>
</script>
```

源码： 视图模型

```javascript
var Person = function(name, children) {
    this.name = ko.observable(name);
    this.children = ko.observableArray(children || []);
};
 
var PeopleModel = function() {
    this.people = ko.observableArray([
        new Person("Bob", [
            new Person("Jan"),
            new Person("Don", [
                new Person("Ted"),
                new Person("Ben", [
                    new Person("Joe", [
                        new Person("Ali"),
                        new Person("Ken")
                    ])
                ]),
                new Person("Doug")
            ])
        ]),
        new Person("Ann", [
            new Person("Eve"),
            new Person("Hal")
        ])
    ]);
 
    this.addChild = function(name, parentArray) {
        parentArray.push(new Person(name));
    };
};
 
ko.applyBindings(new PeopleModel());
 
//attach event handlers
$("#people").on("click", ".remove", function() {
    //retrieve the context
    var context = ko.contextFor(this),
        parentArray = context.$parent.people || context.$parent.children;
 
    //remove the data (context.$data) from the appropriate array on its parent (context.$parent)
    parentArray.remove(context.$data);
 
    return false;
});
 
$("#people").on("click", ".add", function() {
    //retrieve the context
    var context = ko.contextFor(this),
        childName = context.$data.name() + " child",
        parentArray = context.$data.people || context.$data.children;
 
    //add a child to the appropriate parent, calling a method off of the main view model (context.$root)
    context.$root.addChild(childName, parentArray);
 
    return false;
});
```

不管嵌套的链接变成什么样，处理器总是能够表示并操作适当的数据。
使用这种技术，我们可以避免给每个独立链接添加处理器的开支并保持标记清晰整洁。