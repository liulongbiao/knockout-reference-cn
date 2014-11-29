# Mapping

Knockout 被设计为允许你使用任意的 JavaScript 对象作为视图模型。
只要视图模型的某些属性是 [observables](./observables.md)，
你都可以用 KO 将其绑定到 UI 上，且当该 observable 的属性变更时 UI 会自动更新。

大多数应用需要从后端服务器获取数据。
因为服务器没有任何 observable 的概念，它只会提供纯 JavaScript 对象(通常被序列化为 JSON)。
mapping 插件让你可以直接地将纯 JavaScript 对象映射为带合适的 observables 的视图模型。
这是手动将基于某些服务器获取的数据构造视图模型的替代方案。

### 下载

* [2.0 版](https://github.com/SteveSanderson/knockout.mapping/tree/master/build/output) 压缩后 8.6 kb

### 示例： 不带 `ko.mapping` 插件的手动映射

你想显示你的 web 页面上当前服务器时间和用户数量。你可以用以下视图模型表示该信息：

```javascript
var viewModel = {
    serverTime: ko.observable(),
    numUsers: ko.observable()
}
```

你可以将该视图模型绑到某些 HTML 元素上：

```html
The time on the server is: <span data-bind='text: serverTime'></span>
and <span data-bind='text: numUsers'></span> user(s) are connected.
```

因为视图模型是 observable ， KO 将在这些属性变更时自动更新 HTML 元素。

接下来，你想从服务器获取最新数据。每 5 秒钟你可能会发出 Ajax 请求
(如，使用 jQuery 的 `$.getJSON` 或 `$.ajax` 函数)：

```javascript
var data = getDataUsingAjax();          // Gets the data from the server
```

服务器可能返回类似以下的数据：

```javascript
{
    serverTime: '2010-01-07',
    numUsers: 3
}
```

最后，使用该数据更新你的视图模型(不使用 mapping 插件)，你可以写：

```javascript
// Every time data is received from the server:
viewModel.serverTime(data.serverTime);
viewModel.numUsers(data.numUsers);
```

你需要给每个想在页面上显示的变量手动更新。
若你的数据结构变得更加复杂(如，它们包含子数据或包含数组)，手动处理会更加繁琐。
mapping 插件让你从常规的 JavaScript 对象(或 JSON 结构) 创建 observable 视图模型。

### 示例： 使用 `ko.mapping`

要通过 mapping 插件创建视图模型，将上面代码中 `viewModel` 的创建替换为使用 `ko.mapping.fromJS` 函数：

```javascript
var viewModel = ko.mapping.fromJS(data);
```

它会自动该 `data` 上的每个属性创建 observable 属性。
然后，每次你从服务器接收新数据时，你可以再次调用 `ko.mapping.fromJS` 函数一步更新所有 `viewModel` 上的属性：

```javascript
// Every time data is received from the server:
ko.mapping.fromJS(data, viewModel);
```

### <a name="how-things-are-mapped"></a>对象如何被映射

* 对象的所有属性被转换为一个 observable。若一个更新会改变其值，它将更新该 observable。
* 数组被转换为 [observable 数组](./observableArrays.md)。
若一个更新会改变项的数量，它会执行合适的添加/移除动作。它也回保持和原始 JavaScript 数组相同的顺序。

### 反映射

若你想将映射后的对象转换会常规的 JS 对象，使用：

```javascript
var unmapped = ko.mapping.toJS(viewModel);
```

它将会创建一个反映射的对象，它仅包含曾是你的原始 JS 对象部分的被映射对象的属性。
因此换句话说，任何你手动添加到你的视图模型的属性或函数会被忽略。
默认，该规则仅有的例外是 `_destroy` 属性会被映射回去，因为它是当你从一个 `ko.observableArray`
中清除一个项时 Knockout 可能会生成的属性。
更多如何配置的信息查看 "高级用法" 部分。

### 使用 JSON 字符串

当你的 Ajax 调用返回一个 JSON 字符串(且还没被序列化为 JavaScript 对象)，
则你可以选择使用 `ko.mapping.fromJSON` 来创建和更新你的视图模型。
要反映射可以使用 `ko.mapping.toJSON` 。

除了它们可用于 JSON 字符串而不是 JS 对象，它和其相应的 `*JS` 函数是等价的。

### 高级用法

有时可能会需要控制映射任何被执行。它可通过使用 *mapping 选项* 来完成。
它们可在 `ko.mapping.fromJS` 时被指定。在后续调用中，你就不需要再指定它们了。

以下是某些你可能使用这些 mapping 选项的场景。

#### 使用 "keys" 唯一地标识对象

假设你有一个 JavaScript 对象：

```javascript
var data = {
    name: 'Scot',
    children: [
        { id : 1, name : 'Alicw' }
    ]
}
```

你可以没有任何问题地将其映射为视图模型：

```javascript
var viewModel = ko.mapping.fromJS(data)
```

现在，假设该数据被更新为没有错别字的：

```javascript
var data = {
    name: 'Scott',
    children: [
        { id : 1, name : 'Alice' }
    ]
}
```

这里会发生两件事情： `name` 从 `Scot` 变为 `Scott` 且 `children[0].name`
从 `Alicw` 被改为没有错别字的 `Alice`。你可以给予新数据来更新 `viewModel`：

```javascript
ko.mapping.fromJS(data, viewModel);
```

`name` 会被如愿改变。然而，在 `children` 数组中，子项 (Alicw) 将被完全移除而一个新节点 (Alice) 会被添加。
这不完全是你所希望的。相反，你可能希望仅有子项的 `name` 属性从 `Alicw` 被更新为 `Alice`，
而不是替换掉整个子项。

它的发生时因为，默认情况下， mapping 插件简单地对比数组中的两个对象。
因为 JavaScript 对象 `{ id : 1, name : 'Alicw' } ` 不等于 `{ id : 1, name : 'Alice' }`，
它任务 *整个* 子项需要被移除并替换为一个新的子项。

要解决这个，你可以指定哪个 `key` 来决定对象是新还是旧的。你可以如下设置：

```javascript
var mapping = {
    'children': {
        key: function(data) {
            return ko.utils.unwrapObservable(data.id);
        }
    }
}
var viewModel = ko.mapping.fromJS(data, mapping);
```

这时，每次 mapping 插件检查 `children` 的项时，它仅会查看 `id` 属性来决定一个对象是被完全移除还是需要更新。

#### 使用 "create" 来自定义对象构建

若你想自行处理某部分的映射，你可以提供一个 `create` 回调。
若存在该回调， mapping 插件让你可以自己处理这部分映射。

假设你有如下 JavaScript 对象：

```javascript
var data = {
    name: 'Graham',
    children: [
        { id : 1, name : 'Lisa' }
    ]
}
```

若你想自行映射 `children` ，你可以如下指定：

```javascript
var mapping = {
    'children': {
        create: function(options) {
            return new myChildModel(options.data);
        }
    }
}
var viewModel = ko.mapping.fromJS(data, mapping);
```

提供给你的 `create` 回调的 `options` 参数是包含以下数据的 JavaScript 对象：

* data ： 包含该子项数据的 JavaScript 对象
* parent : 该子项所属的父对象或数组

当然，在 `create` 回调中你可以根据需要调用另一个 `ko.mapping.fromJS`。
典型用例可能是你想给原始的 JavaScript 对象扩展额外的 [computed observables](./computedObservables.md)：

```javascript
var myChildModel = function(data) {
    ko.mapping.fromJS(data, {}, this);
     
    this.nameLength = ko.computed(function() {
        return this.name().length;
    }, this);
}
```

#### 使用 "update" 来自定义对象更新

你也可以通过指定一个 `update` 回调来定制对象如何被更新。它会接收试图更新的对象，
和一个和由 `create` 回调所使用的相同的 `options` 对象。
你应该 `return` 更新后的值。

提供给你的 `update` 回调的 `options` 参数是包含以下数据的 JavaScript 对象：

* data ： 包含该子项数据的 JavaScript 对象
* parent : 该子项所属的父对象或数组
* observable : 若该属性时一个 observable ，它会被设置为实际的 observable

以下是一个配置示例，它会在更新前给来源数据添加某些文本：

```javascript
var data = {
    name: 'Graham',
}
 
var mapping = {
    'name': {
        update: function(options) {
            return options.data + 'foo!';
        }
    }
}
var viewModel = ko.mapping.fromJS(data, mapping);
alert(viewModel.name());
```

它会弹出 `Grahamfoo!`。

#### 使用 "ignore" 来忽略特定属性

若你想 mapping 插件忽略你的 JS 对象的某些属性(即不映射它们)，
你可以指定一个需忽略的属性名的数组：

```javascript
var mapping = {
    'ignore': ["propertyToIgnore", "alsoIgnoreThis"]
}
var viewModel = ko.mapping.fromJS(data, mapping);
```

mapping 选项中指定的 `ignore` 数组被和默认 `ignore` 结合起来。你可以像这样操作默认数组：

```javascript
var oldOptions = ko.mapping.defaultOptions().ignore;
ko.mapping.defaultOptions().ignore = ["alwaysIgnoreThis"];
```

#### 使用 "include" 来包含特定属性

当将你的视图模型转换回 JS 对象时，默认 mapping 插件将仅包含曾是你的原始视图模型部分的属性，
除了它也包含 Knockout 生成 `_destroy` 属性，即使它不是元素对象的一部分。
然而，你可以选择定制该数组：

```javascript
var mapping = {
    'include': ["propertyToInclude", "alsoIncludeThis"]
}
var viewModel = ko.mapping.fromJS(data, mapping);
```

mapping 选项中指定的 `include` 数组被和默认 `include` 结合起来，它默认仅包含 `_destroy`。
你可以像这样操作默认数组：

```javascript
var oldOptions = ko.mapping.defaultOptions().include;
ko.mapping.defaultOptions().include = ["alwaysIncludeThis"];
```

#### 使用 "copy" 来拷贝特定属性

当将你的视图模型转换回 JS 对象时，默认 mapping 插件将仅会基于 [上述规则](#how-things-are-mapped) 创建 observables。
若你想强制 mapping 插件简单地拷贝属性而不是使其变为 observable，将其名称添加到 "copy" 数组：

```javascript
var mapping = {
    'copy': ["propertyToCopy"]
}
var viewModel = ko.mapping.fromJS(data, mapping);
```

mapping 选项中指定的 `copy` 数组被和默认 `copy` 结合起来，它默认是空数组。
你可以像这样操作默认数组：

```javascript
var oldOptions = ko.mapping.defaultOptions().copy;
ko.mapping.defaultOptions().copy = ["alwaysCopyThis"];
```

#### 使用 "copy" 来仅观察特定属性

若你想 mapping 插件仅给 JS 对象的某些属性创建 observables 且拷贝其它属性，
你可以指定你要观察的属性名数组：

```javascript
var mapping = {
    'observe': ["propertyToObserve"]
}
var viewModel = ko.mapping.fromJS(data, mapping);
```

mapping 选项中指定的 `observe` 数组被和默认 `observe` 结合起来，它默认是空数组。
你可以像这样操作默认数组：

```javascript
var oldOptions = ko.mapping.defaultOptions().observe;
ko.mapping.defaultOptions().observe = ["onlyObserveThis"];
```

`ignore` 和 `include` 还是像往常一样工作。数组 `copy` 可用于高效地拷贝数组或包含子数据的对象。
若数组或对象属性没在 `copy` 或 `observe` 中指定，则它会被递归地映射：

```javascript
var data = {
    a: "a",
    b: [{ b1: "v1" }, { b2: "v2" }] 
};
 
var result = ko.mapping.fromJS(data, { observe: "a" });
var result2 = ko.mapping.fromJS(data, { observe: "a", copy: "b" }); //will be faster to map.
```

`result` 和 `result2` 都将变为：

```javascript
{
    a: observable("a"),
    b: [{ b1: "v1" }, { b2: "v2" }] 
}
```

下钻的数组和对象也能工作，但 `copy` 和 `observe` 可能会冲突：

```javascript
var data = {
    a: "a",
    b: [{ b1: "v1" }, { b2: "v2" }] 
};
var result = ko.mapping.fromJS(data, { observe: "b[0].b1"});
var result2 = ko.mapping.fromJS(data, { observe: "b[0].b1", copy: "b" });
```

其 `result` 为：

```javascript
{
    a: "a",
    b: [{ b1: observable("v1") }, { b2: "v2" }] 
}
```

而 `result2` 为：

```javascript
{
    a: "a",
    b: [{ b1: "v1" }, { b2: "v2" }] 
}
```

#### 指定更新目标

如果，像上面一样，你在某个类中执行映射，你可能想有 `this` 作为映射操作的目标。
`ko.mapping.fromJS` 的第三个参数指示了该目标。如，

```javascript
ko.mapping.fromJS(data, {}, someObject); // overwrites properties on someObject
```

因此，若你想映射一个 JavaScript 对象到 `this`，你可以将 `this` 传入作为第三个参数：

```javascript
ko.mapping.fromJS(data, {}, this);
```

#### 从多个来源映射

你可以通过应用多个 `ko.mapping.fromJS` 调用将多个 JS 对象整合为一个视图模型，如：

```javascript
var viewModel = ko.mapping.fromJS(alice, aliceMappingOptions);
ko.mapping.fromJS(bob, bobMappingOptions, viewModel);
```

你在每次调用中所指定的映射选项将被聚合。

#### 映射 observable 数组

由 mapping 插件所生成的 observable 数组被扩展了一些函数，可以使用 `keys` 映射：

* mappedRemove
* mappedRemoveAll
* mappedDestroy
* mappedDestroyAll
* mappedIndexOf

它们的功能等价于常规的 `ko.observableArray` 函数，但可基于对象的键来完成事情。
例如，以下将能运行：

```javascript
var obj = [
    { id : 1 },
    { id : 2 }
]
 
var result = ko.mapping.fromJS(obj, {
    key: function(item) {
        return ko.utils.unwrapObservable(item.id);
    }
});
 
result.mappedRemove({ id : 2 });
```

该被映射数组也暴露了一个 `mappedCreate` 函数：

```javascript
var newItem = result.mappedCreate({ id : 3 });
```

它将首先检查键是否已存在且在存在时抛出一个异常。
然后它会调用 create 和 update 回调，如果有的话，来创建新的对象。
最后，它会添加该对象到数组并返回它。

### 下载

* [2.0 版](https://github.com/SteveSanderson/knockout.mapping/tree/master/build/output) 压缩后 8.6 kb