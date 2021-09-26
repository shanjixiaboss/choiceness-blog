前段时间遇到一个问题，简化以后是这样的情况

```js
function fun1(){
console.log(arguments.length);  // 1
}
function fun2(){
fun1.apply(void  0, arguments);
}

fun2('first', 'second')

```

原代码经过webpack和babel处理，已不可读，但我在chrome中断点一步步调试多次，确实如此。抱歉我的js理论基础不够，这已经超过我的认知范围。
