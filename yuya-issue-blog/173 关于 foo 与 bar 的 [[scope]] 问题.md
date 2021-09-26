如果说在 foo 函数进入上下文栈创建变量对象，此时 bar 函数开始创建，bar.[[scope]] 保存了存所有父变量对象到其中。那么如下代码怎么解释？
function foo() {
    var a = 10
    function bar() {
       var b = 20
    }
    console.dir(bar)
}
foo()
function foo() {
    var a = 10
    function bar() {
       var b = 20
       console.dir(arguments.callee)
    }
    bar()
}
foo()
此时 bar 输出的 [[scopes]] 属性里只有 Global 对象。没有包含 VO(foo)