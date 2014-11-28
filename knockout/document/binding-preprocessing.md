# 使用预处理来扩展 Knockout 的绑定语法

*注：这是一个高级的技术，通常仅在创建可重用绑定类库时会用到。在你使用 Knockout 构建应用时通常不需要它。*

从 Knockout 3.0 开始，开发者可以通过提供回调在绑定过程中重写 DOM 节点和绑定字符串以定义自定义的语法。

### 预处理绑定语法

你可以通过给某个绑定处理器(如`click`, `visible` 或任何 [自定义绑定](./custom-bindings.md))
提供一个 *绑定预处理器* 来挂到 Knockout 的解释 `data-bind` 属性的逻辑上。

要这么做，给该绑定处理器添加一个 `preprocess` 函数：

```javascript
ko.bindingHandlers.yourBindingHandler.preprocess = function(stringFromMarkup) {
    // Return stringFromMarkup if you don't want to change anything, or return
    // some other string if you want Knockout to behave as if that was the
    // syntax provided in the original HTML
}
```

见本页后面的 API 参考

#### 示例1： 给绑定设置默认值

如果你漏掉了一个绑定的值，它默认会被绑定到 `undefined`。
若你想有一个不同的默认值，你可以用一个预处理器来完成。
例如，你可以让 `uniqueName` 没有绑定值时默认绑定值 `true`：

```javascript
ko.bindingHandlers.uniqueName.preprocess = function(val) {
    return val || 'true';
}
```

现在你可以像这样绑定：

```html
<input data-bind="value: someModelProperty, uniqueName" />
```

#### 示例2： 给事件绑定表达式

如果你想给 `click` 事件绑定表达式(而不是像 Knockout 所期待的函数引用)，
你可以给 `click` 处理器设置预处理器来支持该语法：

```javascript
ko.bindingHandlers.click.preprocess = function(val) {
    return 'function($data,$event){ ' + val + ' }';
}
```

现在你可以像这样绑定 `click`：

```html
<button type="button" data-bind="click: myCount(myCount()+1)">Increment</button>
```

#### 绑定预处理器参考

* `ko.bindingHandlers.<name>.preprocess(value, name, addBindingCallback)`

  如果被定义，该函数将在每次绑定被求值前调用。
  
  **参数**：
  
  * `value`: 在 Knockout 试图解析前该绑定所关联的语法(如，对 `yourBinding: 1 + 1`，关联的值是作为一个字符串的 "1 + 1" )
  * `name`: 绑定名称(如，对 `yourBinding: 1 + 1`，名称是作为一个字符串的 "yourBinding")
  * `addBinding`: 一个可选择在当前元素上插入另一个绑定的回调函数。
它需要两个参数 `name` 和 `value`。如，在你的 `preprocess` 函数中，调用 `addBinding('visible', 'acceptsTerms()'); `
来让 Knockout 的行为就像元素上具有一个 `visible: acceptsTerms()` 绑定一样。

  **返回值**：
  
  你的 `preprocess` 函数必需返回新的字符串值以被解析并传递给该绑定，或者返回 `undefined` 来移除该绑定。
  
  例如，若你返回字符串 `'value + ".toUpperCase()"'`， 则你的绑定 `yourBinding: "Bert"`
将被解释为像标签包含了 `yourBinding: "Bert".toUpperCase()` 一样。
Knockout 会想通常一样解析返回的值，因此它必需是一个合法的 JavaScript 表达式。

  不要返回非字符串值。那样没有意义，因为标签总是一个字符串。
  
### 预处理 DOM 节点

你可以通过提供一个 *节点预处理器* 切入 Knockout 用于遍历 DOM 的逻辑。
这是 Knockout 会对每个它游走时所遇到的节点调用一次，不管是 UI 第一次被绑定还是
当随后任何新的 DOM 子树被注入时(如，通过 [foreach 绑定](./foreach-binding.md))。

要这么做，在你的绑定提供者上定义一个 `preprocessNode` 函数：

```javascript
ko.bindingProvider.instance.preprocessNode = function(node) {
    // Use DOM APIs such as setAttribute to modify 'node' if you wish.
    // If you want to leave 'node' in the DOM, return null or have no 'return' statement.
    // If you want to replace 'node' with some other set of nodes,
    //    - Use DOM APIs such as insertChild to inject the new nodes
    //      immediately before 'node'
    //    - Use DOM APIs such as removeChild to remove 'node' if required
    //    - Return an array of any new nodes that you've just inserted
    //      so that Knockout can apply any bindings to them
}
```

见本页后面的 API 参考。

#### 示例3： 虚拟模板元素

如果你常使用虚拟元素来引入模板，常规的语法看起来有些繁琐。
使用预处理器，你可以添加一个新的模板格式，其中只使用了单个注释：

```javascript
ko.bindingProvider.instance.preprocessNode = function(node) {
    // Only react if this is a comment node of the form <!-- template: ... -->
    if (node.nodeType == 8) {
        var match = node.nodeValue.match(/^\s*(template\s*:[\s\S]+)/);
        if (match) {
            // Create a pair of comments to replace the single comment
            var c1 = document.createComment("ko " + match[1]),
                c2 = document.createComment("/ko");
            node.parentNode.insertBefore(c1, node);
            node.parentNode.replaceChild(c2, node);
 
            // Tell Knockout about the new nodes so that it can apply bindings to them
            return [c1, c2];
        }
    }
}
```

现在你可以再你的视图中像这样包含一个模板：

```html
<!-- template: 'some-template' -->
```

#### 预处理参考

* `ko.bindingProvider.instance.preprocessNode(node)`

  如果被定义，该函数将在每次 DOM 节点被绑定前调用。
该函数可以修改、移除或替换 `node`。任何新的节点必需立即在 `node` 前被插入，
且若任何节点被添加或 `node` 被移除，该函数必需返回一个新的节点的数组，
它会替换文档中 `node` 所在的位置。