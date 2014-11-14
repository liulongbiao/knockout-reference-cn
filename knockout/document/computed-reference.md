# Computed Observable 参考

以下文档描述了如何构建和使用 computed observables。

## 构建 computed observable

computed observable 可以通过以下形式之一来构建：

1. `ko.computed( evaluator [, targetObject, options] )` -- 这种形式支持大多数创建 computed observable 的情形
   * `evaluator` -- 一个用于求该 computed observable 当前值的函数
   * `targetObject` -- 如果给定的话，它会定义当 KO 调用你的回调函数时的 `this` 的值。
   * `options` -- 一个带有更多关于 computed observable 的信息的对象。见以下完整列表。
2. `ko.computed( options )` -- 创建 computed observable 的单参数形式，接收一个带有任何以下属性的 JavaScript 对象。
   * `read` -- 必需。一个用于求该 computed observable 当前值的函数
   * `write` -- 可选。如果给定，让该 computed observable 可写。该函数接收的值是其他代码试图给你的 computed observable 写入的值。
然后你可以自行提供自定义逻辑来处理输入值，通常是将值写到其它底层的 observable(s)。
   * `owner` -- 可选。如果给定，它定义了当 KO 调用你的 `read` 或 `write` 回调时 `this` 的值。
   * `pure` -- 可选。如果该选项是 `true`，该 computed observable 将被设置为 [纯 computed observable](./computed-pure.md)。
该选项是 `ko.pureComputed` 构造器的替代形式。
   * `deferEvaluation` -- 可选。如果该选项 `true`，则该 computed observable 的求值将被延迟直到某些东西试图
访问它的值或手动地订阅它。默认情况下， computed observable 的值会在创建时被立即确定。
   * `disposeWhen` -- 可选。如果给点，每次重求值时该函数都会被执行以决定该 computed observable 是否应该被清理。
逻辑真值的结果将会触发该 computed observable 的清理。
   * `disposeWhenNodeIsRemoved` -- 可选。如果给点，当指定 DOM 节点被 KO 所移除时将会触发该 computed observable 的清理。
该特性被用于当节点被 `template` 和控制流绑定所移除时在绑定中清理 computed observable。
3. `ko.pureComputed( evaluator [, targetObject] )` -- 使用给定的求值函数和可选的用作 `this` 的对象来构造 [纯 computed observable](./computed-pure.md)。
不像 `ko.computed`，该方法不接收一个 `options` 参数。
4. `ko.pureComputed( options )` -- 使用一个 `options` 参数来构造一个纯 computed observable。它接收上述的 `read`、`write` 和 `owner` 选项。

## 使用 computed observable

computed observable 提供了以下函数：

* `dispose()` -- 收到清理该 computed observable，清理对所有依赖的订阅。该函数在你想停止某个 computed observable 的更新
或希望给一个依赖于不会被清除的 observables 的 computed observable 清理内存时非常有用。
* `extend(extenders)` -- 给该 computed observable 应用指定的 [extenders](./extenders.md)
* `getDependenciesCount()` -- 返回该 computed observable 当前的依赖的数量
* `getSubscriptionsCount()` -- 返回该 computed observable 当前的订阅的数量(不管是来自其它 computed observables 还是手动订阅)
* `isActive()` -- 返回该 computed observable 未来是否会更新。如果 computed observable 没有依赖的话它就是非激活的。
* `peek()` -- 返回该 computed observable 的当前值而不会创建一个依赖
* `subscribe( callback [,callbackTarget, event] )` -- 注册一个手动订阅以在该 computed observable 变更时收到通知。

## 使用 computed context

在某个 computed observable 的求值函数的执行期间，你可以访问 `ko.computedContext` 来获取关于当前 computed 相关属性的信息。
它提供了以下函数：

* `isInitial()` -- 当当前 computed observable 处于首次求值期间返回 `true`，否则返回 `false`。
对 *纯* computed observable 而言，`isInitial()` 总是返回 `undefined`。
* `getDependenciesCount()` -- 返回该 computed observable 在当前求值中已经侦测到的依赖数量
   * 注：`ko.computedContext.getDependenciesCount()` 等价于在当前 computed observable 本身调用 `getDependenciesCount()`。
在 `ko.computedContext` 也存在该方法的原因是在首次求值期间提供一种给依赖计数的方式，那时该 computed observable 还没有完成构造。

示例：

```javascript
var myComputed = ko.computed(function() {
    // ... Omitted: read some data that might be observable ...
 
    // Now let's inspect ko.computedContext
    var isFirstEvaluation = ko.computedContext.isInitial(),
        dependencyCount = ko.computedContext.getDependenciesCount(),
    console.log("Evaluating " + (isFirstEvaluation ? "for the first time" : "again"));
    console.log("By now, this computed has " + dependencyCount + " dependencies");
 
    // ... Omitted: return the result ...
});
```

这些机制通常仅在高级场景中有用，如当你的 computed observable 的主要目的是在其求值期间触发某些副作用，
其你希望仅在首次运行或仅在其至少存在一个依赖(因此会在未来重新求值)时执行某些设置逻辑。
大多数 computed 属性不需要关心它们是否之前被求值过或它们有多少个依赖。