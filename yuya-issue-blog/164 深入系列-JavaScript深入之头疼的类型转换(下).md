## 前言

举个例子：

```js
console.log(1 + '1')
```

在 JavaScript 中，这是完全可以运行的，不过你有没有好奇，为什么 1 和 '1' 分属不同的数据类型，为什么就可以进行运算呢？

这其实是因为 JavaScript 自动的将数据类型进行了转换，我们通常称为隐式类型转换。但是我们都知道，`+`运算符既可以用于数字加法，也能用于字符串拼接，那在这个例子中，是将数字 `1` 转成字符串 `'1'`，进行拼接运算？还是将字符串 `'1'` 转成数字 `1`，进行加法运算呢？

先卖个关子，虽然估计你也知道答案。今天，我们就常见的隐式类型转化的场景进行介绍。

## 一元操作符 + 

```js
console.log(+'1');
```

当 + 运算符作为一元操作符的时候，查看 [ES5规范1.4.6](http://es5.github.io/#x11.4.6)，会调用 `ToNumber` 处理该值，相当于 `Number('1')`，最终结果返回数字 `1`。

那么下面的这些结果呢？

```js
console.log(+[]);
console.log(+['1']);
console.log(+['1', '2', '3']);
console.log(+{});
```

既然是调用 `ToNumber` 方法，回想[《JavaScript 深入之头疼的类型转换(上)》](https://github.com/mqyqingfeng/Blog/issues/159)中的内容，当输入的值是对象的时候，先调用 `ToPrimitive(input,  Number)` 方法，执行的步骤是：

1. 如果 `obj` 为基本类型，直接返回
2. 否则，调用 `valueOf` 方法，如果返回一个原始值，则 `JavaScript` 将其返回。
2. 否则，调用 `toString` 方法，如果返回一个原始值，则`JavaScript` 将其返回。
3. 否则，`JavaScript` 抛出一个类型错误异常。

以 `+[]` 为例，`[]` 调用 `valueOf` 方法，返回一个空数组，因为不是原始值，调用 `toString` 方法，返回 `""`。

得到返回值后，然后再调用 `ToNumber` 方法，`""` 对应的返回值是 `0`，所以最终返回 `0`。

剩下的例子以此类推。结果是：

```js
console.log(+['1']); // 1
console.log(+['1', '2', '3']); // NaN
console.log(+{}); // NaN
```

## 二元操作符 +

### 规范

现在 `+` 运算符又变成了二元操作符，毕竟它也是加减乘除中的加号

`1 + '1'` 我们知道答案是 '11'，那 `null + 1`、`[] + []`、`[] + {}`、`{} + {}` 呢？

如果要了解这些运算的结果，不可避免的我们要从规范下手。

规范地址：[http://es5.github.io/#x11.6.1](http://es5.github.io/#x11.6.1)

不过这次就不直接大段大段的引用规范了，直接给大家讲简化后的内容。

到底当执行 `+` 运算的时候，会执行怎样的步骤呢？让我们根据规范`11.6.1` 来捋一捋：

当计算 value1 + value2时：

1.  lprim = ToPrimitive(value1)
2.  rprim = ToPrimitive(value2)
3.  如果 lprim 是字符串或者 rprim 是字符串，那么返回 ToString(lprim) 和 ToString(rprim)的拼接结果
4.  返回 ToNumber(lprim) 和 ToNumber(rprim)的运算结果

规范的内容就这样结束了。没有什么新的内容，`ToString`、`ToNumber`、`ToPrimitive`都是在[《JavaScript 深入之头疼的类型转换(上)》](https://github.com/mqyqingfeng/Blog/issues/159)中讲到过的内容，所以我们直接进分析阶段：

让我们来举几个例子：

### 1.Null 与数字

```js
console.log(null + 1);
```

按照规范的步骤进行分析：

1.  lprim = ToPrimitive(null) 因为null是基本类型，直接返回，所以 lprim = null
2.  rprim = ToPrimitive(1) 因为 1 是基本类型，直接返回，所以 rprim = null
3.  lprim 和 rprim 都不是字符串
4.  返回 ToNumber(null) 和 ToNumber(1) 的运算结果

接下来：

`ToNumber(null)` 的结果为0，(回想上篇 Number(null))，`ToNumber(1)` 的结果为 1

所以，`null + 1` 相当于 `0 + 1`，最终的结果为数字 `1`。

这个还算简单，看些稍微复杂的：

### 2.数组与数组

```js
console.log([] + []);
```

依然按照规范：

1.  lprim = ToPrimitive([])，[]是数组，相当于ToPrimitive([], Number)，先调用valueOf方法，返回对象本身，因为不是原始值，调用toString方法，返回空字符串""
2.  rprim类似。
3.  lprim和rprim都是字符串，执行拼接操作

所以，`[] + []`相当于 `"" + ""`，最终的结果是空字符串`""`。

看个更复杂的：

### 3.数组与对象

```js
// 两者结果一致
console.log([] + {});
console.log({} + []);
```

按照规范：

1.  lprim = ToPrimitive([])，lprim = ""
2.  rprim = ToPrimitive({})，相当于调用 ToPrimitive({}, Number)，先调用 valueOf 方法，返回对象本身，因为不是原始值，调用 toString 方法，返回 "[object Object]"
3.  lprim 和 rprim 都是字符串，执行拼接操作

所以，`[] + {}` 相当于 `"" + "[object Object]"`，最终的结果是 "[object Object]"。

下面的例子，可以按照示例类推出结果：

```js
console.log(1 + true);
console.log({} + {});
console.log(new Date(2017, 04, 21) + 1) // 这个知道是数字还是字符串类型就行
```

结果是：

```js
console.log(1 + true); // 2
console.log({} + {}); // "[object Object][object Object]"
console.log(new Date(2017, 04, 21) + 1) // "Sun May 21 2017 00:00:00 GMT+0800 (CST)1"
```

## 注意

以上的运算都是在 `console.log` 中进行，如果你直接在 `Chrome` 或者 `Firebug` 开发工具中的命令行直接输入，你也许会惊讶的看到一些结果的不同，比如：

![type1](https://gw.alicdn.com/tfs/TB1W7pIAXY7gK0jSZKzXXaikpXa-1022-82.jpg)

我们刚才才说过 `{} + []` 的结果是 `"[object Object]"` 呐，这怎么变成了 `0` 了？

不急，我们尝试着加一个括号：

![type2](https://gw.alicdn.com/tfs/TB1jM8KAhn1gK0jSZKPXXXvUXXa-1124-76.jpg)

结果又变成了正确的值，这是为什么呢？

其实，在不加括号的时候，`{}` 被当成了一个独立的空代码块，所以 `{} + []` 变成了 `+[]`，结果就变成了 0

同样的问题还出现在 `{} + {}` 上，而且火狐和谷歌的结果还不一样：

```js
> {} + {}
// 火狐： NaN
// 谷歌： "[object Object][object Object]"
```

如果 `{}` 被当成一个独立的代码块，那么这句话相当于 `+{}`，相当于 `Number({})`，结果自然是 `NaN`，可是 `Chrome` 却在这里返回了正确的值。

那为什么这里就返回了正确的值呢？我也不知道，欢迎解答~

## == 相等

### 规范

`"=="` 用于比较两个值是否相等，当要比较的两个值类型不一样的时候，就会发生类型的转换。

关于使用"=="进行比较的时候，具体步骤可以查看[规范11.9.5](http://es5.github.io/#x11.9.3)：

当执行x == y 时：

1. 如果x与y是同一类型：
    1. x是Undefined，返回true
    2. x是Null，返回true
    3. x是数字：
        1. x是NaN，返回false
        2. y是NaN，返回false
        3. x与y相等，返回true
        4. x是+0，y是-0，返回true
        5. x是-0，y是+0，返回true
        6. 返回false
    4. x是字符串，完全相等返回true,否则返回false
    5. x是布尔值，x和y都是true或者false，返回true，否则返回false
    6. x和y指向同一个对象，返回true，否则返回false

2. x是null并且y是undefined，返回true

3. x是undefined并且y是null，返回true

4. x是数字，y是字符串，判断x == ToNumber(y)

5. x是字符串，y是数字，判断ToNumber(x) == y

6. x是布尔值，判断ToNumber(x) == y

7. y是布尔值，判断x ==ToNumber(y)

8. x不是字符串或者数字，y是对象，判断x == ToPrimitive(y)

9. x是对象，y不是字符串或者数字，判断ToPrimitive(x) == y

10. 返回false

觉得看规范判断太复杂？我们来分几种情况来看：

### 1. null和undefined

```js
console.log(null == undefined);
```

看规范第2、3步：

>2. x是null并且y是undefined，返回true

>3. x是undefined并且y是null，返回true

所以例子的结果自然为 `true`。

这时候，我们可以回想在[《JavaScript专题之类型判断(上)》](https://github.com/mqyqingfeng/Blog/issues/28)中见过的一段 `demo`，就是编写判断对象的类型 `type` 函数时，如果输入值是 `undefined`，就返回字符串 `undefined`，如果是 `null`，就返回字符串 `null`。

如果是你，你会怎么写呢？

下面是 jQuery 的写法：

```js
function type(obj) {
    if (obj == null) {
        return obj + '';
    }
    ...
}
```

### 2. 字符串与数字

```js
console.log('1' == 1);
```

结果肯定是true，问题在于是字符串转化成了数字和数字比较还是数字转换成了字符串和字符串比较呢？

看规范第4、5步：

>4.x是数字，y是字符串，判断x == ToNumber(y)

>5.x是字符串，y是数字，判断ToNumber(x) == y

结果很明显，都是转换成数字后再进行比较

### 3. 布尔值和其他类型

```js
console.log(true == '2')
```

当要判断的一方出现 `false` 的时候，往往最容易出错，比如上面这个例子，凭直觉应该是 `true`，毕竟 `Boolean('2')` 的结果可是true，但这道题的结果却是false。

归根到底，还是要看规范，规范第6、7步：

>6.x是布尔值，判断ToNumber(x) == y

>7.y是布尔值，判断x ==ToNumber(y)

当一方出现布尔值的时候，就会对这一方的值进行ToNumber处理，也就是说true会被转化成1，

`true == '2'` 就相当于 `1 == '2'` 就相当于 `1 == 2`，结果自然是 `false`。

所以当一方是布尔值的时候，会对布尔值进行转换，因为这种特性，所以尽量少使用 `xx == true` 和 `xx == false` 的写法。

比如:

```js
// 不建议
if (a == true) {}

// 建议
if (a) {}
// 更好
if (!!a) {}
```

### 4. 对象与非对象

```js
console.log( 42 == ['42'])
```

看规范第8、9步：

>8. x不是字符串或者数字，y是对象，判断x == ToPrimitive(y)

>9. x是对象，y不是字符串或者数字，判断ToPrimitive(x) == y

以这个例子为例，会使用 `ToPrimitive` 处理 `['42']`，调用`valueOf`，返回对象本身，再调用 `toString`，返回 `'42'`，所以

`42 == ['42']` 相当于 `42 == '42'` 相当于`42 == 42`，结果为 `true`。

到此为止，我们已经看完了第2、3、4、5、6、7、8、9步，其他的一概返回 false。

### 其他

再多举几个例子进行分析：

```js
console.log(false == undefined)
```

`false == undefined` 相当于 `0 == undefined` 不符合上面的情形，执行最后一步 返回 `false`

```js
console.log(false == [])
```

`false == []` 相当于 `0 == []` 相当于 `0 == ''` 相当于 `0 == 0`，结果返回 `true`

```js
console.log([] == ![])
```

首先会执行 `![]` 操作，转换成 false，相当于 `[] == false` 相当于 `[] == 0` 相当于 `'' == 0` 相当于 `0 == 0`，结果返回 `true`

最后再举一些会让人踩坑的例子：

```js
console.log(false == "0")
console.log(false == 0)
console.log(false == "")

console.log("" == 0)
console.log("" == [])

console.log([] == 0)

console.log("" == [null])
console.log(0 == "\n")
console.log([] == 0)
```

以上均返回 true

## 其他

除了这两种情形之外，其实还有很多情形会发生隐式类型转换，比如`if`、`? :`、`&&`等情况，但相对来说，比较简单，就不再讲解。

## 深入系列

JavaScript 深入系列目录地址：https://github.com/mqyqingfeng/Blog

JavaScript 深入系列预计写十五篇左右，旨在帮大家捋顺JavaScript底层知识，重点讲解如原型、作用域、执行上下文、变量对象、this、闭包、按值传递、call、apply、bind、new、继承等难点概念。

如果有错误或者不严谨的地方，请务必给予指正，十分感谢。如果喜欢或者有所启发，欢迎 star，对作者也是一种鼓励。
