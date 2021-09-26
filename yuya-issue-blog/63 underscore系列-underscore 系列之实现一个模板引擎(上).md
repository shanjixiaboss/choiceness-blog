## 前言

underscore 提供了模板引擎的功能，举个例子：

```js
var tpl = "hello: <%= name %>";

var compiled = _.template(tpl);
compiled({name: 'Kevin'}); // "hello: Kevin"
```

感觉好像没有什么强大的地方，再来举个例子：

在 HTML 文件中：

```html
<ul id="name_list"></ul>

<script type="text/html" id="user_tmpl">
    <%for ( var i = 0; i < users.length; i++ ) { %>
        <li>
            <a href="<%=users[i].url%>">
                <%=users[i].name%>
            </a>
        </li>
    <% } %>
</script>
```

JavaScript 文件中：

```js
var container = document.getElementById("name_list");

var data = {
    users: [
        { "name": "Kevin", "url": "http://localhost" },
        { "name": "Daisy", "url": "http://localhost" },
        { "name": "Kelly", "url": "http://localhost" }
    ]
}
var precompile = _.template(document.getElementById("user_tmpl").innerHTML);
var html = precompile(data);

container.innerHTML = html;
```

效果为：

![模板引擎效果](https://raw.githubusercontent.com/mqyqingfeng/Blog/master/Images/template/template.png)

那么该如何实现这样一个 _.template 函数呢？

## 实现思路

underscore 的 template 函数参考了 jQuery 的作者 John Resig 在 2008 年发表的一篇文章 [JavaScript Micro-Templating](https://johnresig.com/blog/javascript-micro-templating/#postcomment)，我们先从这篇文章的思路出发，思考一下如何写一个简单的模板引擎。

依然是以这段模板字符串为例：

```js
<%for ( var i = 0; i < users.length; i++ ) { %>
    <li>
        <a href="<%=users[i].url%>">
            <%=users[i].name%>
        </a>
    </li>
<% } %>
```

John Resig 的思路是将这段代码转换为这样一段程序：

```js
// 模拟数据
var users = [{"name": "Kevin", "url": "http://localhost"}];

var p = [];
for (var i = 0; i < users.length; i++) {
    p.push('<li><a href="');
    p.push(users[i].url);
    p.push('">');
    p.push(users[i].name);
    p.push('</a></li>');
}

// 最后 join 一下就可以得到最终拼接好的模板字符串
console.log(p.join('')) // <li><a href="http://localhost">Kevin</a></li>
```

我们注意，模板其实是一段字符串，我们怎么根据一段字符串生成一段代码呢？很容易就想到用 eval，那我们就先用 eval 吧。

然后我们会发现，为了转换成这样一段代码，我们需要将`<%xxx%>`转换为 `xxx`，其实就是去掉包裹的符号，还要将 `<%=xxx%>`转化成 `p.push(xxx)`，这些都可以用正则实现，但是我们还需要写 `p.push('<li><a href="');` 、`p.push('">');`呐，这些该如何实现呢？

那我们换个思路，依然是用正则，但是我们

1. 将 `%>` 替换成 `p.push('`
2. 将 `<%` 替换成 `');`
3. 将 `<%=xxx%>` 替换成 `');p.push(xxx);p.push('`

我们来举个例子：

```
<%for ( var i = 0; i < users.length; i++ ) { %>
    <li>
        <a href="<%=users[i].url%>">
            <%=users[i].name%>
        </a>
    </li>
<% } %>
```

按照这个替换规则会被替换为：

```
');for ( var i = 0; i < users.length; i++ ) { p.push('
    <li>
        <a href="');p.push(users[i].url);p.push('">
            ');p.push(users[i].name);p.push('
        </a>
    </li>
'); } p.push('
```

这样肯定会报错，毕竟代码都没有写全，我们在首和尾加上部分代码，变成：

```
// 添加的首部代码
var p = []; p.push('

');for ( var i = 0; i < users.length; i++ ) { p.push('
    <li>
        <a href="');p.push(users[i].url);p.push('">
            ');p.push(users[i].name);p.push('
        </a>
    </li>
'); } p.push('

// 添加的尾部代码
');
```

我们整理下这段代码：

```js
var p = []; p.push('');
for ( var i = 0; i < users.length; i++ ) { 
    p.push('<li><a href="');
    p.push(users[i].url);
    p.push('">');
    p.push(users[i].name);
    p.push('</a></li>'); 
}
    p.push('');
```

恰好可以实现这个功能，不过还要注意一点，要将换行符替换成空格，防止解析成代码的时候报错，不过在这里为了方便理解原理，就只在代码里实现。

## 第一版

我们来尝试实现第一版：

```js
// 第一版
function tmpl(str, data) {
    var str = document.getElementById(str).innerHTML;

    var string = "var p = []; p.push('" +
    str
    .replace(/[\r\t\n]/g, "")
    .replace(/<%=(.*?)%>/g, "');p.push($1);p.push('")
    .replace(/<%/g, "');")
    .replace(/%>/g,"p.push('")
    + "');"

    eval(string)

    return p.join('');
};
```

为了验证是否有用：

HTML 文件：

```html
<script type="text/html" id="user_tmpl">
    <%for ( var i = 0; i < users.length; i++ ) { %>
        <li>
            <a href="<%=users[i].url%>">
                <%=users[i].name%>
            </a>
        </li>
    <% } %>
</script>
```

JavaScript 文件：

```js
var users = [
    { "name": "Byron", "url": "http://localhost" },
    { "name": "Casper", "url": "http://localhost" },
    { "name": "Frank", "url": "http://localhost" }
]
tmpl("user_tmpl", users)
```

完整的 Demo 可以查看 [template 示例一](https://github.com/mqyqingfeng/Blog/tree/master/demos/template/template1)

## Function

在这里我们使用了 eval ，实际上 John Resig 在文章中使用的是 Function 构造函数。

Function 构造函数创建一个新的 Function 对象。 在 JavaScript 中, 每个函数实际上都是一个 Function 对象。

使用方法为：

```js
new Function ([arg1[, arg2[, ...argN]],] functionBody)
```

arg1, arg2, ... argN 表示函数用到的参数，functionBody 表示一个含有包括函数定义的 JavaScript 语句的字符串。 

举个例子：

```js
var adder = new Function("a", "b", "return a + b");

adder(2, 6); // 8
```

那么 John Resig 到底是如何实现的呢？

## 第二版

使用 Function 构造函数：

```js
// 第二版
function tmpl(str, data) {
    var str = document.getElementById(str).innerHTML;

    var fn = new Function("obj",

    "var p = []; p.push('" +

    str
    .replace(/[\r\t\n]/g, "")
    .replace(/<%=(.*?)%>/g, "');p.push($1);p.push('")
    .replace(/<%/g, "');")
    .replace(/%>/g,"p.push('")
    + "');return p.join('');");

    return fn(data);
};
```

使用方法依然跟第一版相同，具体 Demo 可以查看 [template 示例二](https://github.com/mqyqingfeng/Blog/tree/master/demos/template/template2)

不过值得注意的是：其实 tmpl 函数没有必要传入 data 参数，也没有必要在最后 return 的时候，传入 data 参数，即使你把这两个参数都去掉，代码还是可以正常执行的。

这是因为:

> 使用Function构造器生成的函数，并不会在创建它们的上下文中创建闭包；它们一般在全局作用域中被创建。当运行这些函数的时候，它们只能访问自己的本地变量和全局变量，不能访问Function构造器被调用生成的上下文的作用域。这和使用带有函数表达式代码的 eval 不同。

这里之所以依然传入了 data 参数，是为了下一版做准备。

## with

现在有一个小问题，就是实际上我们传入的数据结构可能比较复杂，比如：

```js
var data = {
    status: 200,
    name: 'kevin',
    friends: [...]
}
```

如果我们将这个数据结构传入 tmpl 函数中，在模板字符串中，如果要用到某个数据，总是需要使用 `data.name`、`data.friends` 的形式来获取，麻烦就麻烦在我想直接使用 name、friends 等变量，而不是繁琐的使用 `data.` 来获取。

这又该如何实现的呢？答案是 with。

with 语句可以扩展一个语句的作用域链(scope chain)。当需要多次访问一个对象的时候，可以使用 with 做简化。比如：

```js
var hostName = location.hostname;
var url = location.href;

// 使用 with
with(location){
    var hostname = hostname;
    var url = href;
}
```

```js
function Person(){
    this.name = 'Kevin';
    this.age = '18';
}

var person = new Person();

with(person) {
    console.log('my name is ' + name + ', age is ' + age + '.')
}
// my name is Kevin, age is 18.
```

最后：不建议使用 with 语句，因为它可能是混淆错误和兼容性问题的根源，除此之外，也会造成性能低下

## 第三版

使用 with ，我们再写一版代码：

```js
// 第三版
function tmpl(str, data) {
    var str = document.getElementById(str).innerHTML;

    var fn = new Function("obj",

    // 其实就是这里多添加了一句 with(obj){...}
    "var p = []; with(obj){p.push('" +

    str
    .replace(/[\r\t\n]/g, "")
    .replace(/<%=(.*?)%>/g, "');p.push($1);p.push('")
    .replace(/<%/g, "');")
    .replace(/%>/g,"p.push('")
    + "');}return p.join('');");

    return fn(data);
};
```

具体 Demo 可以查看 [template 示例三](https://github.com/mqyqingfeng/Blog/tree/master/demos/template/template3)

## 第四版

如果我们的模板不变，数据却发生了变化，如果使用我们的之前写的 tmpl 函数，每次都会 new Function，这其实是没有必要的，如果我们能在使用 tmpl 的时候，返回一个函数，然后使用该函数，传入不同的数据，只根据数据不同渲染不同的 html 字符串，就可以避免这种无谓的损失。

```js
// 第四版
function tmpl(str, data) {
    var str = document.getElementById(str).innerHTML;

    var fn = new Function("obj",

    "var p = []; with(obj){p.push('" +

    str
    .replace(/[\r\t\n]/g, "")
    .replace(/<%=(.*?)%>/g, "');p.push($1);p.push('")
    .replace(/<%/g, "');")
    .replace(/%>/g,"p.push('")
    + "');}return p.join('');");

    var template = function(data) {
        return fn.call(this, data)
    }
    return template;
};

// 使用时
var compiled = tmpl("user_tmpl");
results.innerHTML = compiled(data);
```

具体 Demo 可以查看 [template 示例四](https://github.com/mqyqingfeng/Blog/tree/master/demos/template/template4)

## 下期预告

至此，我们已经跟着 jQuery 的作者 John Resig 实现了一个简单的模板引擎，虽然 underscore 基于这个思路实现，但是功能强大，相对的，代码也更加复杂一下，下一篇，我们一起去分析 underscore 的 template 函数实现。

## underscore 系列

underscore 系列目录地址：[https://github.com/mqyqingfeng/Blog](https://github.com/mqyqingfeng/Blog)。

underscore 系列预计写八篇左右，重点介绍 underscore 中的代码架构、链式调用、内部函数、模板引擎等内容，旨在帮助大家阅读源码，以及写出自己的 undercore。

如果有错误或者不严谨的地方，请务必给予指正，十分感谢。如果喜欢或者有所启发，欢迎 star，对作者也是一种鼓励。
