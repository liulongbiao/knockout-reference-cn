# 创建支持虚拟元素的

*注：这是一个高级的技术，通常仅在创建可重用绑定类库时会用到。在你使用 Knockout 构建应用时通常不需要它。*

Knockout 的 *控制流绑定* (如 [if](./if-binding.md) 和 [foreach](./foreach-binding.md))
不仅可以应用到常规的 DOM 元素上，也能应用到由特殊的基于注释语法所定义的 "虚拟" DOM 元素上。
例如：

```html
<ul>
    <li class="heading">My heading</li>
    <!-- ko foreach: items -->
        <li data-bind="text: $data"></li>
    <!-- /ko -->
</ul>
```

自定义绑定也可以用于虚拟元素，但要启用它，你需要通过使用 `ko.virtualElements.allowedBindings` API 
明确告诉 Knockout 你的绑定能理解虚拟元素。

### 示例

下例中自定义绑定会随机打乱 DOM 节点的顺序：

```javascript
ko.bindingHandlers.randomOrder = {
    init: function(elem, valueAccessor) {
        // Pull out each of the child elements into an array
        var childElems = [];
        while(elem.firstChild)
            childElems.push(elem.removeChild(elem.firstChild));
 
        // Put them back in a random order
        while(childElems.length) {
            var randomIndex = Math.floor(Math.random() * childElems.length),
                chosenChild = childElems.splice(randomIndex, 1);
            elem.appendChild(chosenChild[0]);
        }
    }
};
```

它在常规的 DOM 元素上能很好地工作。下例元素将被以乱序洗牌：

```html
<div data-bind="randomOrder: true">
    <div>First</div>
    <div>Second</div>
    <div>Third</div>
</div>
```

但它在虚拟元素上 *不* 能运行。如果你尝试：

```html
<!-- ko randomOrder: true -->
    <div>First</div>
    <div>Second</div>
    <div>Third</div>
<!-- /ko -->
```

... 则你会得到一个错误 `The binding 'randomOrder' cannot be used with virtual elements`。
让我们来处理它，通过先告诉 Knockout 来允许它来让 `randomOrder` 在虚拟元素上可用。
添加以下代码：

```javascript
ko.virtualElements.allowedBindings.randomOrder = true;
```

现在不会出现错误了。但是，它还是不能正确地运行，因为我们的 `randomOrder` 绑定
使用了常规的 DOM API 调用(`firstChild`,`appendChild`等)，
它们不能理解虚拟元素。
这也是为何 KO 需要你明确地声明虚拟元素支持的原因：
除非你的自定义绑定使用了虚拟元素 API ，否则它不会正常工作！

这次让我们用 KO 的虚拟元素 API 来更新 `randomOrder` 的代码：

```javascript
ko.bindingHandlers.randomOrder = {
    init: function(elem, valueAccessor) {
        // Build an array of child elements
        var child = ko.virtualElements.firstChild(elem),
            childElems = [];
        while (child) {
            childElems.push(child);
            child = ko.virtualElements.nextSibling(child);
        }
 
        // Remove them all, then put them back in a random order
        ko.virtualElements.emptyNode(elem);
        while(childElems.length) {
            var randomIndex = Math.floor(Math.random() * childElems.length),
                chosenChild = childElems.splice(randomIndex, 1);
            ko.virtualElements.prepend(elem, chosenChild[0]);
        }
    }
};
```

注意我们是如何通过使用 `ko.virtualElements.firstChild(domOrVirtualElement)` 等 API
来替换 `domElement.firstChild` 等 API 的使用的。
`randomOrder` 绑定现在能够在虚拟元素上正确地运行了，如
`<!-- ko randomOrder: true -->...<!-- /ko -->`。

同样，`randomOrder` 在常规的 DOM 元素上依旧能运行，
因为所有的 `ko.virtualElements` API 都后向兼容常规的 DOM 元素。

### 虚拟元素 APIs

Knockout 提供了以下用于操作虚拟元素的函数。

* ko.virtualElements.allowedBindings

  一个对象，其键决定了哪个绑定在虚拟元素上可用。
设置 `ko.virtualElements.allowedBindings.mySuperBinding = true` 来允许 `mySuperBinding` 可用于虚拟元素。

* ko.virtualElements.emptyNode(containerElem)

  从真实或虚拟元素 `containerElem` 上移除所有子节点(也会清除任何它们所关联的数据以避免内存泄露)
  
* ko.virtualElements.firstChild(containerElem)

  返回真实或虚拟元素 `containerElem` 上的第一个子节点。如果没有子节点就返回 `null`。
  
* ko.virtualElements.insertAfter(containerElem, nodeToInsert, insertAfter)

  将 `nodeToInsert` 作为真实或虚拟元素 `containerElem` 的子节点插入，
位置就紧跟 `insertAfter` 后面(其中 `insertAfter` 必需时 `containerElem` 的子节点)

* ko.virtualElements.nextSibling(node)

  返回在真实或虚拟父元素中 `node` 节点后跟兄弟节点，或者没有后继兄弟节点时返回 `null`。
  
* ko.virtualElements.prepend(containerElem, nodeToPrepend)

  将 `nodeToPrepend` 作为真实或虚拟元素 `containerElem` 的第一个子节点插入。
  
* ko.virtualElements.setDomNodeChildren(containerElem, arrayOfNodes)

  从真实或虚拟元素 `containerElem` 上移除所有子节点(在这个过程中清除任何它们所关联的数据以避免内存泄露)，
然后插入所有 `arrayOfNodes` 中的节点作为其新的子节点。

注意它不是常规 DOM API 的一个完整地替换。
Knockout 只提供了让在实现控制流绑定时所需的这些类型的转换成为可能的最小集合的虚拟元素 APIs。