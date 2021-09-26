@Tan90Qian 厉害了，我也才发现，原来：

```js
Object.prototype.toString.call(location)
// "[object Location]"
Object.prototype.toString.call(history)
// "[object History]"
```

而且他们跟自身相等是因为引用是相同的吧，就像：

```js
var obj = {};
console.log(obj === obj) // true
```

_Originally posted by @mqyqingfeng in https://github.com/mqyqingfeng/Blog/issues/28#issuecomment-381464362_