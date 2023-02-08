---
title: new
url: https://www.yuque.com/endday/blog/rhb3md
---

> 1.创建了一个全新的对象。
> 2.这个对象会被执行`[[Prototype]]`（也就是`__proto__`）链接。

> 3.生成的新对象会绑定到函数调用的this。

> 4.通过`new`创建的每个对象将最终被`[[Prototype]]`链接到这个函数的`prototype`对象上。

> 5.如果函数没有返回对象类型`Object`(包含`Functoin`, `Array`, `Date`, `RegExg`, `Error`)，那么`new`表达式中的函数调用会自动返回这个新的对象。

```javascript
function mockNew () {
  // 创建一个全新的对象
  var obj = {}
  // 取第一个参数
  var Constructor = [].shift.call(arguments)
  // 关联对象的原型链到构造函数的原型上
  // 等同于 Object.setPrototypeOf(obj, Constructor.prototype)
  obj.__proto__ = Constructor.prototype
	// 通过apply执行构造函数，将this指向obj
  var ret = Constructor.apply(obj, arguments)
  // 判断构造函数的返回值是否对象，如果是，则返回这个返回值，否则返回这个新的对象
  return typeof ret === 'object' ? ret : obj
}
```
