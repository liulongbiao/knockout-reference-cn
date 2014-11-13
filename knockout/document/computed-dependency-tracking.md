# 依赖跟踪如何运作

*初学者不需要知道它，但更高阶的开发者会希望知道为什么我们能声明所有这些关于 KO 能自动跟踪依赖
并更新 UI 中正确的部分等等*

它实际上非常简单而可爱。跟踪算法类似于：

1. 当你声明一个 computed observable 时， KO 立即调用其求值函数获取其初始值。
2. 当求值函数运行时，KO 建立了对求值函数中所读取的所有 observables (包括其他 computed observables) 的订阅。
订阅回调的设置导致求值器再次运行，循环整个过程回到步骤 1 (清楚任何不再应用的旧的订阅)。
3. KO 通知任何订阅者有关 computed observable 的新的值。

因此，Knockout 不只是在求值器第一个运行时侦测依赖 - 它每次都重侦测它们。
这意味着，比如说，依赖可以是非常动态化的： 依赖 A 可以决定该 computed observable 是否也依赖于 B 或 C。
然后它将仅在 A 或者你当前的选择 B 或 C 变更时被重求值。
你不需要声明依赖：它们在代码执行时在运行时被决定。

另一个简单的伎俩是声明式绑定被简单地实现为 computed observables。
因此如果某个绑定读取一个 observable 的值，该绑定就会依赖于这个 observable，
这会导致这个 observable 变更时该绑定被重新求值。

*纯* computed observables 的运作方式有些不同。
更多细节，查看 [纯 computed observables](./computed-pure.md) 相关文档。

## 使用 peek 控制依赖

Knockout 的自动依赖跟踪通常做的就是你所需要的。
但有时候你可能需要控制哪些 observables 将更新你的 computed observable，特别是当这个 computed observable
执行某些动作，如发送 Ajax 请求时。
`peek` 函数让你可以访问一个 observable 或 computed observable 而不会创建一个依赖。

下例中，一个 computed observable 被用于使用 Ajax 和其它两个 observable 属性的数据来
重新加载名为 `currentPageData` 的 observable 。
这个 computed observable 会在每次 `pageIndex` 变化时更新，但却忽略 `selectedItem` 上的变更，
因为它是用 `peek` 来访问的。
这里，用户可能只是想把 `selectedItem` 的当前值用于当新的集合的数据被加载时的跟踪目的。

```javascript
ko.computed(function() {
    var params = {
        page: this.pageIndex(),
        selected: this.selectedItem.peek()
    };
    $.getJSON('/Some/Json/Service', params, this.currentPageData);
}, this);
```

> 注：如果你希望避免 computed observable 更新过于频繁，参见 [rateLimit extender](./rateLimit-observable.md)。

## 注： 为何循环依赖没有意义

computed observables 被设计为将一组 observables 输入映射为单个 observable 输出。
因此，在依赖链中包含环没有意义。环并不类比于递归；它们类比于两个 Excel 单元格分别是彼此的函数计算值。
这会导致无限循环。

因此，如果你在依赖图中包含一个环时 Knockout 怎么做？
它通过强制以下规则来避免无限循环： **Knockout 在其求值过程中不会重新开启一个 computed 的求值**。
这一般不会影响你的代码。
它和两种场景相关：当两个 computed observables 彼此依赖时(仅在某个或两个都使用了 `deferEvaluation` 选项有可能)，
或者当某个 computed observable 写给另一个它具有依赖的 observable (直接或通过依赖链)。
如果你需要使用某种模式且希望完整地避免循环依赖，你可以使用上面所描述的 `peek` 函数。