# Underscore 源码阅读

JavaScript 的函数是一级对象，可以以值的形式进行传递(函数对象可以作为参数，也可以作为返回值)。
对 JavaScript 的函数的理解是学习 JavaScript 中的一个重要的步骤。

Underscore 是一个小巧的工具类库，提供了很多有用的函数式编程工具。
对 Underscore 源码进行阅读可以更好地掌握对这个类库的使用，也能增进自己对 JavaScript 中函数式编程的理解。

本文基于 Underscore [1.7.0](./underscore-1.7.0.js) 版本。
在官网 http://underscore.org 上还有一份带注解的源码文档，感兴趣的可以参考。

## 通用初始化块

> ### 即时调用函数
> 
> 在类库的书写中，类库作者需要尽量避免变量泄露到全局变量中，以防和其它引用的 Js 代码产生冲突。
> 我们知道，在 JavaScript 只有函数的执行才能产生一个作用域，所以大量类库都采用了即时调用函数来生成作用域，
> 所有需要使用到的变量都会限制在这个作用域中。

```javascript
(function() {
  // put your code here...
})();
```

Underscore 的目标执行环境是浏览器和 NodeJs。Underscore 并不支持 AMD 模块规范。
在第 12 行，定义了根对象，在浏览器中就是 `window` 对象，在服务器端就是 `exports` 对象。

```javascript
var root = this;
```

类库会给全局对象暴露一个变量名，所以通常类库都会提供方法，给调用者提供处理命名冲突的方法。流程类似于：

```javascript
var previousUnderscore = root._;
var _ = myImpl;
root._ = _;
_.noConflict = function() {
  root._ = previousUnderscore;
  return this;
};
```

这样在加载完类库后，调用者可以调用 `var someVar = _.noConflict()` 得到这个类库对象，而原来同名的 `_` 可以继续使用。

在 18~33 行，给一些常用的原型对象和函数定义别名，便于使用。

第 36 行定义了 `_` ，它是一个构造函数，对传入的参数做了一个封装。
后续会给 `_` 构造函数的原型对象进行扩展，这样就添加了很多有用的方法，便于链式调用。

```javascript
var _ = function(obj) {
  if (obj instanceof _) return obj;
  if (!(this instanceof _)) return new _(obj);
  this._wrapped = obj;
};
```

第 45 行对 NodeJs 模块定义导出对象，对浏览器直接挂到全局对象上：

```javascript
  if (typeof exports !== 'undefined') {
    if (typeof module !== 'undefined' && module.exports) {
      exports = module.exports = _;
    }
    exports._ = _;
  } else {
    root._ = _;
  }
```

第 60 行是一个内部使用的函数，根据参数个数返回一个回调函数，这个方法在内部各种迭代器中会使用到。

```javascript
  var createCallback = function(func, context, argCount) {
    if (context === void 0) return func;
    switch (argCount == null ? 3 : argCount) {
      case 1: return function(value) {
        return func.call(context, value);
      };
      case 2: return function(value, other) {
        return func.call(context, value, other);
      };
      case 3: return function(value, index, collection) {
        return func.call(context, value, index, collection);
      };
      case 4: return function(accumulator, value, index, collection) {
        return func.call(context, accumulator, value, index, collection);
      };
    }
    return function() {
      return func.apply(context, arguments);
    };
  };
```

其实际执行效果其实就是给函数绑定一个 context 上下文对象给 `this`，而且其实直接返回最后一个函数效果是一样。
但在大量测试结果中，当前浏览器引擎直接引用上述 4 个版本执行速度更快，所以使用了这个函数进行了优化。

> ### `call`、`apply`、`bind` 和 `this`
>
> 函数调用时，`this` 会动态地指向一个上下文对象。如果是对象方法调用，那么 `this` 会指向这个对象。
> 如 `a.doSomething()` 中 `this` 会指向对象 `a`。而如果函数直接调用，则 `this` 会指向全局对象。
> 如 `doSomething()` 中 `this` 会指向全局对象。
> 通过函数的 `call` 和 `apply` 方法来调用函数，我们可以把函数执行中的 `this` 指向其第一个参数。
> 在 ES5 规范中给 Function.prototype 添加了 `bind`，可用于将函数的 `this` 上下文永久地绑定到某个对象上。

第 84 行，创建了一个内部使用的用于创建迭代器的函数：

```javascript
  _.iteratee = function(value, context, argCount) {
    if (value == null) return _.identity;
    if (_.isFunction(value)) return createCallback(value, context, argCount);
    if (_.isObject(value)) return _.matches(value);
    return _.property(value);
  };
```

如果迭代参数是 `null` 或 `undefined` 那么，返回的迭代器是标识函数，即返回参数对象本身的函数；
如果迭代参数是一个函数，则调用上面的 `createCallback` 创建一个绑定了上下文的回调；
如果迭代参数是一个对象，说明迭代器的每个属性都需要和这个对象中的属性相匹配；
其他情况，认为迭代参数是一个字符串，迭代器就是返回对象这个对应属性的函数。
其内部调用的函数在后面的代码中会有实现。

## 集合相关函数

> It is better to have 100 functions operate on one data structure than 10 functions on 10 data structures.

数组是有序的元素集合，对象是无序的键值对的集合。
在抽象的角度上看，它们都可以看做集合，我们可以遍历集合、将集合中每个元素映射为另一个集合中的元素、过滤元素等等。

程序就是将输入数据经过转换聚合等操作加工成新的数据输出的过程。
在函数式语言中，操作对象基本都是集合，因此最著名的函数式语言直接取 LISt Processing 而成 LISP。

对于集合的基本高阶函数有:

* `map` - 遍历集合，将每个元素转换成另一个元素，返回新元素组成的集合
* `reduce` - 遍历集合聚合出一个总的结果
* `filter` - 过滤集合，返回匹配指定条件的元素集合

旧版本的 JavaScript 中，数组直接面向实现，并没有这些函数式原语。
随着语言的发展，在 ES5 中对语言引入了一些新的特性，其中数组相关的有：

* `Array.prototype.forEach`
* `Array.prototype.map`
* `Array.prototype.reduce`
* `Array.prototype.reduceRight`
* `Array.prototype.filter`
* `Array.prototype.every`
* `Array.prototype.some`
* `Array.prototype.indexOf`
* `Array.prototype.lastIndexOf`
* `Array.isArray`

从抽象的角度看，其实这里很多方法不仅仅可以针对数组，对对象而言也应该适用。
所以 Underscore 这样的类库主要是给数组和对象提供统一的函数式原语，并且对低版本的 JavaScript 也兼容。

在早期版本的 Underscore 中，它会检测 `Array.prototype` 是否有原生的 `forEach` 或 `map` 等函数。
如果存在则直接调用原生函数，如果不存在则使用自身的实现。
但实际测试中，不同的浏览器实现的性能参差不齐，且原生的函数暂时并没有性能优势，所以现在版本的实现中，直接使用相应的方法实现。

```javascript
  _.each = _.forEach = function(obj, iteratee, context) {
    if (obj == null) return obj;
    iteratee = createCallback(iteratee, context);
    var i, length = obj.length;
    if (length === +length) {
      for (i = 0; i < length; i++) {
        iteratee(obj[i], i, obj);
      }
    } else {
      var keys = _.keys(obj);
      for (i = 0, length = keys.length; i < length; i++) {
        iteratee(obj[keys[i]], keys[i], obj);
      }
    }
    return obj;
  };
```

这里传入的 obj 应该是一个数组或对象；iteratee 是一个函数，它有三个形参，第一个是值，第二个是键，第三个是集合对象本身。
它调用了 `createCallback` 来绑定上下文。

这里使用 `length === +length` 来判断对象是否具有数字类型的 length 属性；如果有 length 属性则按照数组方式进行迭代，
否则按照对象方法迭代其键值对。这里没有用 `_.isArray` 来作判断，给迭代 *伪数组对象* 提供了空间。

```javascript
var a = {'0': 1, length: 2};
[].push.call(a, 'test');
console.log(a); // Object {0: 1, 2: "test", length: 3}
```

实际上 JavaScript 中的数组的实现比较讨巧。JavaScript 引擎本身并没有像 C 语言一样有对应硬件的数组。
JavaScript 中的数组基于对象来实现，可认为是特殊的对象。它有一个 `length` 属性来指示当前数组的长度。
除此以外，它就可以被认为是键为索引数字的对象了。
因此 `Array.prototype` 上的函数基本上可以用来操作任何带有 `length` 属性的对象。

最常见的伪数组对象是函数内部的 `arguments` 对象，它是调用函数时所有传入的参数的集合，但它并不是一个数组。
`arguments` 对象给我们实现不定数量参数函数和多态函数提供了着手点。
所以 `Function.prototype.apply` 的第二个型参其实接收伪数组对象。

最后它返回集合本身，以支持链式调用。

```javascript
  _.map = _.collect = function(obj, iteratee, context) {
    if (obj == null) return [];
    iteratee = _.iteratee(iteratee, context);
    var keys = obj.length !== +obj.length && _.keys(obj),
        length = (keys || obj).length,
        results = Array(length),
        currentKey;
    for (var index = 0; index < length; index++) {
      currentKey = keys ? keys[index] : index;
      results[index] = iteratee(obj[currentKey], currentKey, obj);
    }
    return results;
  };
```

`map` 的实现中，使用了 `_.iteratee` 来生成迭代器。
然后它也是判断 `length` 属性来确定返回的数组的长度和迭代方式。

```javascript
  var reduceError = 'Reduce of empty array with no initial value';

  _.reduce = _.foldl = _.inject = function(obj, iteratee, memo, context) {
    if (obj == null) obj = [];
    iteratee = createCallback(iteratee, context, 4);
    var keys = obj.length !== +obj.length && _.keys(obj),
        length = (keys || obj).length,
        index = 0, currentKey;
    if (arguments.length < 3) {
      if (!length) throw new TypeError(reduceError);
      memo = obj[keys ? keys[index++] : index++];
    }
    for (; index < length; index++) {
      currentKey = keys ? keys[index] : index;
      memo = iteratee(memo, obj[currentKey], currentKey, obj);
    }
    return memo;
  };
```

`reduce` 遍历集合聚合成一个结果。这里 `iteratee` 参数是一个函数，其第一个形参为每次迭代的中间结果，
后面的形参依次为集合元素的值、键及集合本身。
这里通过 `arguments.length < 3` 而不是 `memo == null` 来判断是否传入初始值，是因为有可能传入的初始值恰好是 null，
但 arguments.length 可以知道调用者确实传入了初始值。
迭代的过程类似于 `map` 的实现，只是返回的是单个结果。

某些特定的聚合过程和遍历集合的方向有关，或者特定实现的遍历效率需要从右往左聚合。
所以和 `reduce` 对应提供了一个 `reduceRight` 实现。

```javascript
  _.find = _.detect = function(obj, predicate, context) {
    var result;
    predicate = _.iteratee(predicate, context);
    _.some(obj, function(value, index, list) {
      if (predicate(value, index, list)) {
        result = value;
        return true;
      }
    });
    return result;
  };
```

`find` 方法找到第一个满足条件的元素。它内部调用了后面定义的 `some` 函数。`find` 应该是一种常见的使用模式而提取出来。
在函数式编程中最基础的高阶函数就是 `map`、`reduce`、`filter`，其他函数是在它们的基础上实现的。

```javascript
  _.filter = _.select = function(obj, predicate, context) {
    var results = [];
    if (obj == null) return results;
    predicate = _.iteratee(predicate, context);
    _.each(obj, function(value, index, list) {
      if (predicate(value, index, list)) results.push(value);
    });
    return results;
  };
```

在 Underscore 的实现中，`each` 给其他高阶函数提供了基础。`filter` 的实现就很简单，如果满足条件就把值放到结果集中。
`reject` 恰好和 `filter` 相反，把满足条件的值从集合中移除。

```javascript
  _.every = _.all = function(obj, predicate, context) {
    if (obj == null) return true;
    predicate = _.iteratee(predicate, context);
    var keys = obj.length !== +obj.length && _.keys(obj),
        length = (keys || obj).length,
        index, currentKey;
    for (index = 0; index < length; index++) {
      currentKey = keys ? keys[index] : index;
      if (!predicate(obj[currentKey], currentKey, obj)) return false;
    }
    return true;
  };
```

`every` 在集合中的元素都满足指定条件时返回 `true`，否则返回 `false`。所以这里在遇到第一个不满足的元素时其实就可以跳出循环。
在低版本的 Underscore 里，`every` 是调用 `each` 来迭代集合，所以以前 `each` 定义了一个 `breaker` 私有空对象来跳出循环。
在这里直接使用 for 循环来迭代集合，所以可以直接通过在循环中 `return false;` 来跳出循环，但选择迭代方式等需自行定义。
这里弃用 `each` 来迭代主要是性能因素。

`some` 方法和 `every` 相反，只有集合中有某个元素满足条件就返回 `true`，否则返回 `false`。这里它的实现和 `every` 雷同。

```javascript
  _.contains = _.include = function(obj, target) {
    if (obj == null) return false;
    if (obj.length !== +obj.length) obj = _.values(obj);
    return _.indexOf(obj, target) >= 0;
  };
```

`contains` 判断集合的值中是否包含某个元素。它通过 `===` 来判断相等性。


## 数组相关函数
## 函数相关函数
## 对象相关函数
## 工具函数
## 链式调用