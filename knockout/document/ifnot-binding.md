# "ifnot" 绑定

### 目的

`ifnot` 绑定类似于 [if 绑定](./if-binding.md)，除了它反转任何你传入的表达式的结果。
更多细节，可以查看 [if 绑定](./if-binding.md) 的文档。

### 注： "ifnot" 等同于 "if" 的反面

以下标签：

```html
<div data-bind="ifnot: someProperty">...</div>
```

等价于以下：

```html
<div data-bind="if: !someProperty()">...</div>
```

假设 `someProperty` 是 *observable* 且因此你需要作为函数来调用它以获取当前值。

使用 `ifnot` 而不是反的 `if` 仅是审美的缘故：很多开发者任何它更简洁。