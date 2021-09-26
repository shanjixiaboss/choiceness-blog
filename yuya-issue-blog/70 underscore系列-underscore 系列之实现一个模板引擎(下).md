## 前言

本篇接着上篇 [underscore 系列之实现一个模板引擎(上)](https://github.com/mqyqingfeng/Blog/issues/63)。

鉴于本篇涉及的知识点太多，我们先来介绍下会用到的知识点。

## 反斜杠的作用

```js
var txt = "We are the so-called "Vikings" from the north."
console.log(txt);
```

我们的本意是想打印带 `""` 包裹的 `Vikings` 字符串，但是在 JavaScript 中，字符串使用单引号或者双引号来表示起始或者结束，这段代码会报 `Unexpected identifier` 错误。

如果我们就是想要在字符串中使用单引号或者双引号呢？

我们可以使用反斜杠用来在文本字符串中插入省略号、换行符、引号和其他特殊字符：

```js
var txt = "We are the so-called \"Vikings\" from the north."
console.log(txt);
```

现在 JavaScript 就可以输出正确的文本字符串了。

**这种由反斜杠后接字母或数字组合构成的字符组合就叫做“转义序列”。**

值得注意的是，转义序列会被视为单个字符。

我们常见的转义序列还有 `\n` 表示换行、`\t` 表示制表符、`\r` 表示回车等等。

## 转义序列

在 JavaScript 中，字符串值是一个由零或多个 Unicode 字符（字母、数字和其他字符）组成的序列。

字符串中的每个字符均可由一个转义序列表示。比如字母 `a`，也可以用转义序列 `\u0061` 表示。

> 转义序列以反斜杠 `\` 开头，它的作用是告知 JavaScript 解释器下一个字符是特殊字符。 

> 转义序列的语法为 `\uhhhh`，其中 hhhh 是四位十六进制数。

根据这个规则，我们可以算出常见字符的转义序列，以字母 `m` 为例：

```js
// 1. 求出字符 `m` 对应的 unicode 值
var unicode = 'm'.charCodeAt(0) // 109
// 2. 转成十六进制
var result = unicode.toString(16); // "6d"
```

我们就可以使用 `\u006d` 表示 `m`，不信你可以直接在浏览器命令行中直接输入字符串 `'\u006d'`，看下打印结果。

值得注意的是: `\n` 虽然也是一种转义序列，但是也可以使用上面的方式：

```js
var unicode = '\n'.charCodeAt(0) // 10
var result = unicode.toString(16); // "a"
```

所以我们可以用 `\u000A` 来表示换行符 `\n`，比如在浏览器命令行中直接输入 `'a \n b'` 和 `'a \u000A b'` 效果是一样的。

讲了这么多，我们来看看一些常用字符的转义序列以及含义：

<table>
    <tr>
        <td>Unicode 字符值</td>
        <td>转义序列</td>
        <td>含义</td>
    </tr>
    <tr>
        <td>\u0009</td>
        <td>\t</td>
        <td>制表符</td>
    </tr>
    <tr>
        <td>\u000A</td>
        <td>\n</td>
        <td>换行</td>
    </tr>
    <tr>
        <td>\u000D</td>
        <td>\r</td>
        <td>回车</td>
    </tr>
    <tr>
        <td>\u0022</td>
        <td>\"</td>
        <td>双引号</td>
    </tr>
    <tr>
        <td>\u0027</td>
        <td>\'</td>
        <td>单引号</td>
    </tr>
    <tr>
        <td>\u005C</td>
        <td>\\</td>
        <td>反斜杠</td>
    </tr>
    <tr>
        <td>\u2028</td>
        <td></td>
        <td>行分隔符</td>
    </tr>
    <tr>
        <td>\u2029</td>
        <td></td>
        <td>段落分隔符</td>
    </tr>
</table>

## Line Terminators

Line Terminators，中文译文`行终结符`。像空白字符一样，`行终结符`可用于改善源文本的可读性。

在 ES5 中，有四个字符被认为是`行终结符`，其他的折行字符都会被视为空白。

这四个字符如下所示：

| 字符编码值 | 名称 |
| -------|:-----:|
| \u000A | 换行符  |
| \u000D | 回车符  |
| \u2028 | 行分隔符 |
| \u2029 | 段落分隔符 |

## Function

试想我们写这样一段代码，能否正确运行：

```js
var log = new Function("var a = '1\t23';console.log(a)");
log()
```

答案是可以，那下面这段呢：

```js
var log = new Function("var a = '1\n23';console.log(a)");
log()
```

答案是不可以，会报错 `Uncaught SyntaxError: Invalid or unexpected token`。

这是为什么呢？

这是因为在 Function 构造函数的实现中，首先会将函数体代码字符串进行一次 `ToString` 操作，这时候字符串变成了：

```
var a = '1
23';console.log(a)
```

然后再检测代码字符串是否符合代码规范，在 JavaScript 中，**字符串表达式中是不允许换行的**，这就导致了报错。

为了避免这个问题，我们需要将代码修改为：

```js
var log = new Function("var a = '1\\n23';console.log(a)");
log()
```

其实不止 `\n`，其他三种 `行终结符`，如果你在字符串表达式中直接使用，都会导致报错！

之所以讲这个问题，是因为在模板引擎的实现中，就是使用了 Function 构造函数，如果我们在模板字符串中使用了 `行终结符`，便有可能会出现一样的错误，所以我们必须要对这四种 `行终结符` 进行特殊的处理。

## 特殊字符

除了这四种 `行终结符` 之外，我们还要对两个字符进行处理。

一个是 `\`。

比如说我们的模板内容中使用了`\`:

```js
var log = new Function("var a = '1\23';console.log(a)");
log(); // 1
```

其实我们是想打印 '1\23'，但是因为把 `\` 当成了特殊字符的标记进行处理，所以最终打印了 1。

同样的道理，如果我们在使用模板引擎的时候，使用了 `\` 字符串，也会导致错误的处理。

第二个是 `'`。

如果我们在模板引擎中使用了 `'`，因为我们会拼接诸如 `p.push('` `')` 等字符串，因为 `'` 的原因，字符串会被错误拼接，也会导致错误。

所以总共我们需要对六种字符进行特殊处理，处理的方式，就是正则匹配出这些特殊字符，然后比如将 `\n` 替换成 `\\n`，`\` 替换成 `\\`，`'` 替换成 `\\'`，处理的代码为：

```js
var escapes = {
    "'": "'",
    '\\': '\\',
    '\r': 'r',
    '\n': 'n',
    '\u2028': 'u2028',
    '\u2029': 'u2029'
};

var escapeRegExp = /\\|'|\r|\n|\u2028|\u2029/g;

var escapeChar = function(match) {
    return '\\' + escapes[match];
};
```

我们测试一下：

```js
var str = 'console.log("I am \n Kevin");';
var newStr = str.replace(escapeRegExp, escapeChar);

eval(newStr)
// I am 
// Kevin
```

## replace

我们来讲一讲字符串的 replace 函数：

语法为：

```
str.replace(regexp|substr, newSubStr|function)
```

replace 的第一个参数，可以传一个字符串，也可以传一个正则表达式。

第二个参数，可以传一个新字符串，也可以传一个函数。

我们重点看下传入函数的情况，简单举一个例子：

```js
var str = 'hello world';
var newStr = str.replace('world', function(match){
    return match + '!'
})
console.log(newStr); // hello world!
```

match 表示匹配到的字符串，但函数的参数其实不止有 match，我们看个更复杂的例子：

```js
function replacer(match, p1, p2, p3, offset, string) {
    // match，表示匹配的子串 abc12345#$*%
    // p1，第 1 个括号匹配的字符串 abc
    // p2，第 2 个括号匹配的字符串 12345
    // p3，第 3 个括号匹配的字符串 #$*%
    // offset，匹配到的子字符串在原字符串中的偏移量 0
    // string，被匹配的原字符串 abc12345#$*%
    return [p1, p2, p3].join(' - ');
}
var newString = 'abc12345#$*%'.replace(/([^\d]*)(\d*)([^\w]*)/, replacer); // abc - 12345 - #$*%
```

另外要注意的是，如果第一个参数是正则表达式，并且其为全局匹配模式， 那么这个方法将被多次调用，每次匹配都会被调用。

举个例子，如果我们要在一段字符串中匹配出 `<%=xxx%>` 中的值：

```js
var str = '<li><a href="<%=www.baidu.com%>"><%=baidu%></a></li>'

str.replace(/<%=(.+?)%>/g, function(match, p1, offset, string){
    console.log(match);
    console.log(p1);
    console.log(offset);
    console.log(string);
})
```

传入的函数会被执行两次，第一次的打印结果为：

```
<%=www.baidu.com%>
www.baidu.com
13
<li><a href="<%=www.baidu.com%>"><%=baidu%></a></li>
```

第二次的打印结果为：

```
<%=baidu%>
'baidu'
33
<li><a href="<%=www.baidu.com%>"><%=baidu%></a></li>
```

## 正则表达式的创建

当我们要建立一个正则表达式的时候，我们可以直接创建：

```js
var reg = /ab+c/i;
```

也可以使用构造函数的方式：

```js
new RegExp('ab+c', 'i');
```

值得一提的是：每个正则表达式对象都有一个 source 属性，返回当前正则表达式对象的模式文本的字符串：

```js
var regex = /fooBar/ig;
console.log(regex.source); // "fooBar"，不包含 /.../ 和 "ig"。
```

## 正则表达式的特殊字符

正则表达式中有一些特殊字符，比如 `\d` 就表示了匹配一个数字，等价于 [0-9]。

在上节，我们使用 `/<%=(.+?)%>/g` 来匹配 `<%=xxx%>`，然而在 underscore 的实现中，用的却是 `/<%=([\s\S]+?)%>/g`。

我们知道 \s 表示匹配一个空白符，包括空格、制表符、换页符、换行符和其他 Unicode 空格，\S 
匹配一个非空白符，[\s\S]就表示匹配所有的内容，可是为什么我们不直接使用 `.` 呢？

我们可能以为 `.` 匹配任意单个字符，实际上，并不是如此， `.`匹配除`行终结符`之外的任何单个字符，不信我们做个试验：

```js
var str = '<%=hello world%>'

str.replace(/<%=(.+?)%>/g, function(match){
    console.log(match); // <%=hello world%>
})
```

但是如果我们在 hello world 之间加上一个`行终结符`，比如说 '\u2029'：

```js
var str = '<%=hello \u2029 world%>'

str.replace(/<%=(.+?)%>/g, function(match){
    console.log(match);
})
```

因为匹配不到，所以也不会执行 console.log 函数。

但是改成 `/<%=([\s\S]+?)%>/g` 就可以正常匹配：

```js
var str = '<%=hello \u2029 world%>'

str.replace(/<%=([\s\S]+?)%>/g, function(match){
    console.log(match); // <%=hello   world%>
})
```

## 惰性匹配

仔细看 `/<%=([\s\S]+?)%>/g` 这个正则表达式，我们知道 `x+` 表示匹配 `x` 1 次或多次。`x?`表示匹配 `x` 0 次或 1 次，但是 `+?` 是个什么鬼？

实际上，如果在数量词 *、+、? 或 {}, 任意一个后面紧跟该符号（?），会使数量词变为非贪婪（ non-greedy） ，即匹配次数最小化。反之，默认情况下，是贪婪的（greedy），即匹配次数最大化。

举个例子：

```
console.log("aaabc".replace(/a+/g, "d")); // dbc

console.log("aaabc".replace(/a+?/g, "d")); // dddbc
```

在这里我们应该使用非惰性匹配，举个例子：

```js
var str = '<li><a href="<%=www.baidu.com%>"><%=baidu%></a></li>'

str.replace(/<%=(.+?)%>/g, function(match){
    console.log(match);
})

// <%=www.baidu.com%>
// <%=baidu%>
```

如果我们使用惰性匹配：

```js
var str = '<li><a href="<%=www.baidu.com%>"><%=baidu%></a></li>'

str.replace(/<%=(.+)%>/g, function(match){
    console.log(match);
})

// <%=www.baidu.com%>"><%=baidu%>
```

## template

讲完需要的知识点，我们开始讲 underscore 模板引擎的实现。

与我们上篇使用数组的 push ，最后再 join 的方法不同，underscore 使用的是字符串拼接的方式。

比如下面这样一段模板字符串：

```
<%for ( var i = 0; i < users.length; i++ ) { %>
    <li>
        <a href="<%=users[i].url%>">
            <%=users[i].name%>
        </a>
    </li>
<% } %>
```

我们先将 `<%=xxx%>` 替换成 `'+ xxx +'`，再将 `<%xxx%>` 替换成 `'; xxx __p+='`:

```
';for ( var i = 0; i < users.length; i++ ) { __p+='
    <li>
        <a href="'+ users[i].url + '">
            '+ users[i].name +'
        </a>
    </li>
';  } __p+='
```

这段代码肯定会运行错误的，所以我们再添加些头尾代码，然后组成一个完整的代码字符串：

```
var __p='';
with(obj){
__p+='

';for ( var i = 0; i < users.length; i++ ) { __p+='
    <li>
        <a href="'+ users[i].url + '">
            '+ users[i].name +'
        </a>
    </li>
';  } __p+='

';
};
return __p;
```

整理下代码就是：

```js
var __p='';
with(obj){
    __p+='';
    for ( var i = 0; i < users.length; i++ ) { 
        __p+='<li><a href="'+ users[i].url + '"> '+ users[i].name +'</a></li>';
    }
    __p+='';
};
return __p
```

然后我们将 `__p` 这段代码字符串传入 Function 构造函数中：

```js
var render = new Function(data, __p)
```

我们执行这个 render 函数，传入需要的 data 数据，就可以返回一段 HTML 字符串：

```js
render(data)
```

## 第五版 - 特殊字符的处理

我们接着上篇的[第四版](https://github.com/mqyqingfeng/Blog/tree/master/demos/template/template4)进行书写，不过加入对特殊字符的转义以及使用字符串拼接的方式：

```js
// 第五版
var settings = {
    // 求值
    evaluate: /<%([\s\S]+?)%>/g,
    // 插入
    interpolate: /<%=([\s\S]+?)%>/g,
};

var escapes = {
    "'": "'",
    '\\': '\\',
    '\r': 'r',
    '\n': 'n',
    '\u2028': 'u2028',
    '\u2029': 'u2029'
};

var escapeRegExp = /\\|'|\r|\n|\u2028|\u2029/g;

var template = function(text) {

    var source = "var __p='';\n";
    source = source + "with(obj){\n"
    source = source + "__p+='";

    var main = text
    .replace(escapeRegExp, function(match) {
        return '\\' + escapes[match];
    })
    .replace(settings.interpolate, function(match, interpolate){
        return "'+\n" + interpolate + "+\n'"
    })
    .replace(settings.evaluate, function(match, evaluate){
        return "';\n " + evaluate + "\n__p+='"
    })

    source = source + main + "';\n }; \n return __p;";

    console.log(source)

    var render = new Function('obj',  source);

    return render;
};
```

完整的使用代码可以参考 [template 示例五](https://github.com/mqyqingfeng/Blog/tree/master/demos/template/template5)。

## 第六版 - 特殊值的处理

不过有一点需要注意的是：

如果数据中 `users[i].url` 不存在怎么办？此时取值的结果为 undefined，我们知道：

```js
'1' + undefined // "1undefined"
```

就相当于拼接了 undefined 字符串，这肯定不是我们想要的。我们可以在代码中加入一点判断：

```js
.replace(settings.interpolate, function(match, interpolate){
    return "'+\n" + (interpolate == null ? '' : interpolate) + "+\n'"
})
```

但是吧，我就是不喜欢写两遍 interpolate …… 嗯？那就这样吧：

```js
var source = "var __t, __p='';\n";

...

.replace(settings.interpolate, function(match, interpolate){
    return "'+\n((__t=(" + interpolate + "))==null?'':__t)+\n'"
})
```

其实就相当于：

```js
var __t;

var result = (__t = interpolate) == null ? '' : __t;
```

完整的使用代码可以参考 [template 示例六](https://github.com/mqyqingfeng/Blog/tree/master/demos/template/template6)。

## 第七版

现在我们使用的方式是将模板字符串进行多次替换，然而在 underscore 的实现中，只进行了一次替换，我们来看看 underscore 是怎么实现的：

```js
var template = function(text) {
    var matcher = RegExp([
        (settings.interpolate).source,
        (settings.evaluate).source
    ].join('|') + '|$', 'g');

    var index = 0;
    var source = "__p+='";

    text.replace(matcher, function(match, interpolate, evaluate, offset) {
        source += text.slice(index, offset).replace(escapeRegExp, function(match) {
            return '\\' + escapes[match];
        });

        index = offset + match.length;

        if (interpolate) {
            source += "'+\n((__t=(" + interpolate + "))==null?'':__t)+\n'";
        } else if (evaluate) {
            source += "';\n" + evaluate + "\n__p+='";
        }

        return match;
    });

    source += "';\n";

    source = 'with(obj||{}){\n' + source + '}\n'

    source = "var __t, __p='';" +
        source + 'return __p;\n';

    var render = new Function('obj', source);

    return render;
};
```

其实原理也很简单，就是在执行多次匹配函数的时候，不断复制字符串，处理字符串，拼接字符串，最后拼接首尾代码，得到最终的代码字符串。

不过值得一提的是：在这段代码里，matcher 的表达式最后为：`/<%=([\s\S]+?)%>|<%([\s\S]+?)%>|$/g`

问题是为什么还要加个 `|$` 呢？我们来看下 $：

```js
var str = "abc";
str.replace(/$/g, function(match, offset){
    console.log(typeof match) // 空字符串
    console.log(offset) // 3
    return match
})
```

我们之所以匹配 $，是为了获取最后一个字符串的位置，这样当我们 text.slice(index, offset)的时候，就可以截取到最后一个字符。

完整的使用代码可以参考 [template 示例七](https://github.com/mqyqingfeng/Blog/tree/master/demos/template/template7)。

## 最终版

其实代码写到这里，就已经跟 underscore 的实现很接近了，只是 underscore 加入了更多细节的处理，比如：

1. 对数据的转义功能
2. 可传入配置项
3. 对错误的处理
4. 添加 source 属性，以方便查看代码字符串
5. 添加了方便调试的 print 函数
6. ...

但是这些内容都还算简单，就不一版一版写了，最后的版本在 [template 示例八](https://github.com/mqyqingfeng/Blog/tree/master/demos/template/template8)，如果对其中有疑问，欢迎留言讨论。

## underscore 系列

underscore 系列目录地址：[https://github.com/mqyqingfeng/Blog](https://github.com/mqyqingfeng/Blog)。

underscore 系列预计写八篇左右，重点介绍 underscore 中的代码架构、链式调用、内部函数、模板引擎等内容，旨在帮助大家阅读源码，以及写出自己的 undercore。

如果有错误或者不严谨的地方，请务必给予指正，十分感谢。如果喜欢或者有所启发，欢迎 star，对作者也是一种鼓励。