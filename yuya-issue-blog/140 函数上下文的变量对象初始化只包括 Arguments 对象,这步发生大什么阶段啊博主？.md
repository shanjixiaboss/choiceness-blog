到这里变量对象的创建过程就介绍完了，让我们简洁的总结我们上述所说：

1.全局上下文的变量对象初始化是全局对象

2.函数上下文的变量对象初始化只包括 Arguments 对象

3.在进入执行上下文时会给变量对象添加形参、函数声明、变量声明等初始的属性值

4.在代码执行阶段，会再次修改变量对象的属性值

我理解的是：
或者理解成要执行到那行代码了，又分了三个阶段，1，初始化，2，进入执行阶段，最后执行代码，反正初始化是在进入执行之前，应该在读到代码要执行之后，否则没法根据实参个数初始化arguments对象。对吗
