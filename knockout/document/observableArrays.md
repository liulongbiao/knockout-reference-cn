# Observable Arrays

当你需要侦测并响应单个对象上的变更，你可以使用 [Observable](./observables.md)。
如果你要侦测并响应 *事物的集合* 上的变更，你就需要使用 `observableArray`。
它在很多情况下很有用，如你需要显示或编辑多个值且需要重复部分 UI 以在项目被添加和移除时相应地显示和隐藏。

```javascript
var myObservableArray = ko.observableArray();    // Initially an empty array
myObservableArray.push('Some value');            // Adds the value and notifies observers
```

要了解如何将 `observableArray` 绑定到 UI 上且可让用户编辑它，可以查看
[简单列表示例](http://knockoutjs.com/examples/simpleList.html)。

> 关键：`observableArray` 仅跟踪哪个对象在数组中，而不是这些对象的状态。

简单地将对象放到一个 `observableArray` 中并不会让这些对象的属性本身具有可观察性。
当然你可以根据需要自行让这些属性成为 Observable，但这是一个独立地选择。
`observableArray` 仅跟踪其所持有的对象，并在对象被添加和删除时通知监听器。

## 填充 `observableArray`

如果你不希望从一个空的可观察数组开始，而是包含了初始值，可以将这些项作为数组传递给其构造器，如：

```javascript
// This observable array initially contains three objects
var anotherObservableArray = ko.observableArray([
    { name: "Bungle", type: "Bear" },
    { name: "George", type: "Hippo" },
    { name: "Zippy", type: "Unknown" }
]);
```

## 从 `observableArray` 中读取信息

在幕后，`observableArray` 实际上就是一个值是一个数组的 observable 
(`observableArray` 添加了下述的额外的特性)。
因此，你可以通过将该 `observableArray` 以无参形式进行调用来获取底层的 JavaScript 数组，
就像任何其他的 observable 一样。
然后你就可以从底层的数组中读取信息，如：

```javascript
alert('The length of the array is ' + myObservableArray().length);
alert('The first element is ' + myObservableArray()[0]);
```

技术上，你可以使用任何原生 JavaScript 数组函数来直接操作底层的数组，但通常还有更好的选择。
KO 的 `observableArray` 本身具有等价的函数，且更加有用，因为：

1. 它们在所有目标浏览器上都可用。(如原生的 `indexOf` 在 IE8 及以前版本中不可用，但 KO 的 `indexOf` 都可用)
2. 对修改数组的函数来说，如 `push` 和 `splice`，KO 的方法会自动触发依赖跟踪机制，
这样所有的注册监听器都会收到变更通知，且你的 UI 会自动更新。
3. 其语法更加简洁。要调用 KO 的 `push` 方法，只需要写 `myObservableArray.push(...)`。
它比通过写 `myObservableArray().push(...)` 来调用底层数组的方法更好一些。

本页后面部分描述了用于读写 `observableArray` 上数组信息的函数。

### indexOf

`indexOf` 函数返回数组中第一个等于你的参数的项的索引。如 `myObservableArray.indexOf('Blah')`
将返回第一个等于 Blah 的数组项的基于零的索引，或者没有找到时返回 `-1`。

### slice

`slice` 是 `observableArray` 上原生 JavaScript `slice` 函数的等价物(即它返回从给点起止索引间的项)。
调用 `myObservableArray.slice(...)` 等价于调用底层数组的相同方法(即`myObservableArray().slice(...)`)。

## 操作 `observableArray`

`observableArray` 暴露了类似的方法集来修改数组的内容并通知监听器。

### pop, push, shift, unshift, reverse, sort, splice

所有这些函数都等价于在底层数组上运行原生 JavaScript 数组函数，并将变更通知给监听器：

* `myObservableArray.push('Some new value')` 在数组末尾添加新项
* `myObservableArray.pop()` 从数组中移除最后一个值并返回它
* `myObservableArray.unshift('Some new value')` 在数组头部添加新项
* `myObservableArray.shift()` 从数组中移除第一个值并返回它
* `myObservableArray.reverse()` 反转数组的顺序
* `myObservableArray.sort()` 排序数组内容
   * 默认排序是基于字母顺序的，但你可以传入函数来控制如何排序。该函数接收数组中的两个对象并返回一个负值
指示第一个参数更小，正值表示第二个更小，或者相等时返回零。
* `myObservableArray.splice()` 移除并返回从起始索引开始的给点数量的元素。
如 `myObservableArray.splice(1, 3)` 从索引位置 1 开始移除 3 个元素(即第 2、3、4个元素)，
并将它们作为数组返回。

更多这些 `observableArray` 函数的细节可以查看 
[标准 JavaScript 数组函数](https://developer.mozilla.org/en/JavaScript/Reference/Global_Objects/Array#Methods_2)
的相关文档。

### remove 和 removeAll

`observableArray` 添加了一些 JavaScript 数组默认没有的有用的方法：

* `myObservableArray.remove(someItem)` 移除所有等于 `someItem` 的值，并将它们作为数组返回
* `myObservableArray.remove(function(item) { return item.age < 18 })` 移除所有 `age` 小于 18 的值，并将它们作为数组返回
* `myObservableArray.removeAll(['Chad', 132, undefined])` 移除所有等于 `'chad'`，`123` 或 `undefined` 的值，并将它们作为数组返回
* `myObservableArray.removeAll()` 移除所有值，并将它们作为数组返回

### <a name="destroy-and-destroyAll"></a> destroy 和 destroyAll (仅和 Ruby on Rails 开发者相关)

`destroy` 和 `destroyAll` 函数主要是给使用 Ruby on Rails 的开发者提供便利：

* `myObservableArray.destroy(someItem)` 找到数组中所有等于 `someItem` 的值并给它们一个特殊的称为 `_destroy` 属性设置为 `true`
* `myObservableArray.destroy(function(someItem) { return someItem.age < 18 })` 找到所有 `age` 小于 18 的对象，并给它们一个特殊的称为 `_destroy` 属性设置为 `true`
* `myObservableArray.destroyAll(['Chad', 132, undefined])` 找到所有等于 `'chad'`，`123` 或 `undefined` 的值，并给它们一个特殊的称为 `_destroy` 属性设置为 `true`
* `myObservableArray.destroyAll()` 给数组中所有对象的一个特殊的称为 `_destroy` 属性设置为 `true`

因此 `_destroy` 都是些什么？它只对 Rails 开发者有用。在 Rails 中约定，当给一个 JSON 对象图传递一个动作时，
框架会自动将其转换成一个 ActiveRecord 对象图然后保存到数据库。
它知道哪个对象已存在数据库，且发出正确的 INSERT 或 UPDATE 语句。
要告诉框架 DELETE 一个记录，你仅需将它的 `_destroy` 设置为 `true`。

注意 KO 渲染 `foreach` 绑定时会自动隐藏任何标记了 `_destroy` 为 `true` 的对象。
因此你可以有某种删除按钮调用数组上的 `destroy(someItem)` 方法，它会立即导致特定项从可视化 UI 中消失。
随后，当你给 Rails 提交 JSON 对象图时，这一项会自动从数据库中删除掉
(而其他数组项将会被正常插入或更新)。

## 延迟且/或挂起变更通知

通常，`observableArray` 会在发生变更时立即通知订阅者。
但如果 `observableArray` 变更频繁或会触发昂贵的更新操作，你可能希望通过限制或延迟变更通知来获取更好的性能。
这可以通过 `rateLimit` extender 来完成，如：

```javascript
// Ensure it notifies about changes no more than once per 50-millisecond period
myViewModel.myObservableArray.extend({ rateLimit: 50 });
```