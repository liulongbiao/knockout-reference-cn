# 加载和保存 JSON 数据

Knockout 允许你实现复杂的客户端交互，但几乎所有 web 应用也需要和服务器交换数据，
或者至少需要给本地存储序列化数据。最方便地交换或存储数据的方式是 [JSON 格式](http://json.org/) - 它也是
主流 Ajax 应用所使用的格式。

### 加载或保存数据

Knockout 不强制你使用任何特定的技术来加载或保存数据。
你可以使用任何适合你所选择服务端技术的机制。
最常使用的机制是 jQuery Ajax 帮助方法，如 [getJSON](http://api.jquery.com/jQuery.getJSON/)、
[post](http://api.jquery.com/jQuery.post/) 和 [ajax](http://api.jquery.com/jQuery.ajax/)。
你可以从服务器获取数据：

```javascript
$.getJSON("/some/url", function(data) { 
    // Now use this data to update your view models, 
    // and Knockout will update your UI automatically 
})
```

... 或者你可以发送数据给服务器：

```javascript
var data = /* Your data in JSON format - see below */;
$.post("/some/url", data, function(returnedData) {
    // This callback is executed if the post was successful     
})
```

或者，如果你想使用 jQuery，你可以使用任何其它机制来加载或保存 JSON 数据。
因此，所有 Knockout 需要帮你做的是：

* 对保存，将你的视图模型数据获取为简单地 JSON 格式，这样你可以使用上面技术中的一个
* 对加载，使用上述技术所接收的数据来更新你的视图模型

### 将视图模型数据转换为纯 JSON

你的视图模型是 JavaScript 对象，因此你可以使用任何标准 JSON 序列化器，
如 [JSON.stringify](https://developer.mozilla.org/en-US/docs/JavaScript/Reference/Global_Objects/JSON/stringify (现代浏览器中的原生函数)
或 [json2.js](https://github.com/douglascrockford/JSON-js/blob/master/json2.js) 类库来将它们序列化为 JSON。
然而，你的视图模型可能包含 observables、 computed observables 和 observable 数组，
它们被实现为 JavaScript 函数且因此如果没有你的操作不总是能清晰地序列化。

要很容易的序列化视图模型数据，包含 observables 等， Knockout 包含了两个帮助函数：

* ko.toJS - 它会克隆你的视图模型对象图，为每个 observable 替换为该 observable 的当前值，
这样你可以获取一个仅包含你的数据而没有 Knockout 相关事物的纯拷贝。
* ko.toJSON - 它生成你的视图模型对象的 JSON 字符串表示。
在内部，它简单地在你的视图模型上调用 ko.toJS ，然后对其结果使用浏览器元素的 JSON 进行序列化。
注：要让它在不包含元素 JSON 序列化器的旧浏览器上工作(如 IE7 或更早)，
你必需也引用 json2.js 类库。

例如，定义了如下视图模型：

```javascript
var viewModel = {
    firstName : ko.observable("Bert"),
    lastName : ko.observable("Smith"),
    pets : ko.observableArray(["Cat", "Dog", "Fish"]),
    type : "Customer"
};
viewModel.hasALotOfPets = ko.computed(function() {
    return this.pets().length > 2
}, viewModel)
```

它包含了 observables、 computed observables 和 observable 数组和纯值的混合。
你可以如下使用 ko.toJSON 将其转换成适合发送给服务器的 JSON 字符串：

```javascript
var jsonData = ko.toJSON(viewModel);
 
// Result: jsonData is now a string equal to the following value
// '{"firstName":"Bert","lastName":"Smith","pets":["Cat","Dog","Fish"],"type":"Customer","hasALotOfPets":true}'
```

或者，如果你想在序列化前得到纯 JavaScript 对象图，如下使用 ko.toJS：

```javascript
var plainJs = ko.toJS(viewModel);
 
// Result: plainJS is now a plain JavaScript object in which nothing is observable. It's just data.
// The object is equivalent to the following:
//   {
//      firstName: "Bert",
//      lastName: "Smith",
//      pets: ["Cat","Dog","Fish"],
//      type: "Customer",
//      hasALotOfPets: true
//   }
```

注意 `ko.toJSON` 接收同 [JSON.stringify](https://developer.mozilla.org/en-US/docs/JavaScript/Reference/Global_Objects/JSON/stringify)
相同的参数。例如，在调试 Knockout 应用时能够有视图模型数据的 "活" 的表示非常有用。
要为该目的生成一个良好的格式化的显示，你可以给 ko.toJSON 传入 *空格* 参数，
并像下面这样绑定到你的视图模型：

```html
<pre data-bind="text: ko.toJSON($root, null, 2)"></pre>
```

### 使用 JSON 来更新视图模型数据

若你从服务器加载某些数据且希望用其来更新你的视图模型，最直接的方式是自行完成。如：

```javascript
// Load and parse the JSON
var someJSON = /* Omitted: fetch it from the server however you want */;
var parsed = JSON.parse(someJSON);
 
// Update view model properties
viewModel.firstName(parsed.firstName);
viewModel.pets(parsed.pets);
```

很多场景中，最直接的方式也是最简单和灵活的解决方案。
当然，当你更新视图模型上的属性， Knockout 也会处理更新可视的 UI 来匹配它。

然而，很多开发人员倾向于使用更基于约定的方式使用得到的数据来更新你的视图模型，
而不需要手动给每个需要更新的属性写代码。
若你的视图模型有很多属性或深度嵌套数据结构时非常有好处，
因为它极大地减少了大量你需要写的手动映射代码。
更多该技术的细节，参考 [knockout.mapping 插件](./plugins-mapping.md) 。