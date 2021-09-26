> 这是每一个JavaScript对象(除了 null )都具有的一个属性，叫__proto__，这个属性会指向该对象的原型。

按照冴羽大大文中这句话的描述，__proto__是对象被创造出来就有的，可下面的代码运行情况却否定了这一点

```js
  function Foo() {

  }
  let oldProto = Foo.prototype
  Foo.prototype = Object.create(null)
  let foo = new Foo()
  console.log('Foo.prototype', Foo.prototype)       // {}
  console.log('__proto__', foo.__proto__)           // undefined
  console.log('getPrototypeOf', Object.getPrototypeOf(foo)) // {}
  console.log('__proto__ === oldProto',foo.__proto__ === oldProto) // false
```

这里在实例上的`__proto__`指向丢失，证明了其并不是真的属于实例上的一个隐藏属性。

我们观察对象的原型链末端，也就是`Object.prototype`会发现，其含有`get  __proto__`和`set  __proto__`的getter和setter函数，所以这里猜测`__proto__`也是通过原型链一步一步往上寻找到`Object.prototype`后，调用了 getter 函数得到的结果。上述代码使用
`Object.create(null)`切断了这一联系，所以返回`undefined`。

