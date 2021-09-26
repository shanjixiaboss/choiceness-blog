## partial

在[《 JavaScript 专题之偏函数》](https://github.com/mqyqingfeng/Blog/issues/43)中，我们写了一个 partial 函数，用来固定函数的部分参数，实现代码如下：

```js
// 这是文章中的第一版
function partial(fn) {
    var args = [].slice.call(arguments, 1);
    return function() {
        var newArgs = args.concat([].slice.call(arguments));
        return fn.apply(this, newArgs);
    };
};
```

## rest parameter

ES6 为我们提供了剩余参数（rest parameter）语法，允许我们将一个不定数量的参数表示为一个数组。

```js
function fn(a, b, ...args) {
   console.log(args); // [3, 4, 5]
}

fn(1, 2, 3, 4, 5)
```

我们可以利用这一特性简化 partial 实现的代码：

```js
function partial(fn, ...args) {
    return function(...partialArgs) {
        var newArgs = args.concat(partialArgs);
        return fn.apply(this, newArgs);
    };
};
```

写个 demo，测试一下：

```js
function add(a, b) {
    return a + b;
}

var addOne = partial(add, 1);

console.log(addOne(2)); // 3
```

## restArgs

如果不使用 ... 拓展操作符，仅用 ES5 的内容，该怎么实现呢？

我们可以写一个 restArgs 函数，传入一个函数，使用函数的最后一个参数储存剩下的函数参数，使用效果如下：

```js
var func = restArgs(function(a, b, c){
    console.log(c); // [3, 4, 5]
})

func(1, 2, 3, 4, 5)
```

我们来写一版：

```js
// 第一版
function restArgs(func) {
    return function(){
        // startIndex 表示使用哪个位置的参数用于储存剩余的参数
        var startIndex = func.length - 1;
        var length = arguments.length - startIndex;

        var rest = Array(length)
        var index = 0;

        // 使用一个数组储存剩余的参数
        // 以上面的例子为例，结果为：
        // rest [3, 4, 5]
        for (; index < length; index++) {
            rest[index] = arguments[index + startIndex]
        }

        // args [1, 2, undefined]
        var args = Array(startIndex + 1);
        for (index = 0; index < startIndex; index++) {
            args[index] = arguments[index]
        }

        // args [1, 2, [3, 4, 5]]
        args[startIndex] = rest;

        return func.apply(this, args)
    }
}
```

## 优化

我们默认使用传入的函数的最后一个参数储存剩余的参数，为了更加灵活，我们可以再增加一个参数，用来指定 startIndex，如果没有指定，就默认使用最后一个参数。

此外，注意，我们使用 Array(length) 创建数组，而 length 的计算方式是 arguments.length - startIndex，这个值有可能是负数！比如：

```js
var func = restArgs(function(a, b, c, d){
    console.log(c) // 报错
})

func(1, 2)
```

所以我们再写一版：

```js
// 第二版
function restArgs(func, startIndex) {
    startIndex = startIndex == null ? func.length - 1 : +startIndex;
    return function(){
        var length = Math.max(arguments.length - startIndex, 0);
        var rest = Array(length)
        var index = 0;
        for (; index < length; index++) {
            rest[index] = arguments[index + startIndex]
        }

        var args = Array(startIndex + 1);
        for (index = 0; index < startIndex; index++) {
            args[index] = arguments[index]
        }

        args[startIndex] = rest;
        return func.apply(this, args)
    }
}
```

## 性能优化

如果是正常写业务，可能写到这里就结束了，然而 underscore 考虑的更多，鉴于 call 的性能要高于 apply，所以 underscore 做了一个优化：

```js
// 第三版
var restArgs = function(func, startIndex) {
    startIndex = startIndex == null ? func.length - 1 : +startIndex;
    return function() {
        var length = Math.max(arguments.length - startIndex, 0),
            rest = Array(length),
            index = 0;

        for (; index < length; index++) {
            rest[index] = arguments[index + startIndex];
        }

        // 增加的部分
        switch (startIndex) {
            case 0:
                return func.call(this, rest);
            case 1:
                return func.call(this, arguments[0], rest);
            case 2:
                return func.call(this, arguments[0], arguments[1], rest);
        }

        var args = Array(startIndex + 1);
        for (index = 0; index < startIndex; index++) {
            args[index] = arguments[index];
        }

        args[startIndex] = rest;
        return func.apply(this, args);
    };
};
```

至此，restArgs 函数就完成了，underscore 很多函数比如 invoke、without、union、difference、bind、partial、bindAll、delay 都用到了 restArgs 函数。

当使用 underscore 的时候，我们可以以 _.restArgs 的形式调用该函数。

## restArgs 与 partial

最后，使用我们的写的 restArgs 函数重写下 partial 函数：

```js
var partial = restArgs(function(fn, args){
    return restArgs(function(partialArgs) {
        var newArgs = args.concat(partialArgs);
        return fn.apply(this, newArgs);
    })
})

function add(a, b, c) {
    return a + b + c;
}

var addOne = partial(add, 1);
console.log(addOne(2, 3)); // 6
```

## underscore 系列

underscore 系列目录地址：[https://github.com/mqyqingfeng/Blog](https://github.com/mqyqingfeng/Blog)。

underscore 系列预计写八篇左右，重点介绍 underscore 中的代码架构、链式调用、内部函数、模板引擎等内容，旨在帮助大家阅读源码，以及写出自己的 undercore。

如果有错误或者不严谨的地方，请务必给予指正，十分感谢。如果喜欢或者有所启发，欢迎 star，对作者也是一种鼓励。