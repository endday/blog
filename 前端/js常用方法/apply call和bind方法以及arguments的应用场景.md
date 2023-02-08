---
title: apply call和bind方法以及arguments的应用场景
url: https://www.yuque.com/endday/blog/zkrgyf
---

每个函数都包括两个非继承而来的方法：apply和call。

这两个方法的用途的都是在特定的作用域中调用函数，实际上等于设置函数体内的this对象的值。

apply接收两个参数，一个是其中运行函数的作用域，也就是this的指向，或者说该函数将在哪个对象上执行。另一个参数是参数的数组。

第二个参数可以是array对象，或者直接传入arguments对象。

call和apply的作用相同，区别只在于接受的参数不同，对于call方法而言，第一个参数是this的值，这个和apply是一样的，不同的是，call的其余参数全都直接直接传递给函数。

```javascript
Function.apply(obj, args)
```

obj：这个对象将代替Function类里this对象

args：这个是数组，它将作为参数传给Function（args-->arguments）

```javascript
Function.call(obj, [param1[,param2[,…[,paramN]]]])
```

obj：这个对象将代替Function类里this对象

params：这个是一个参数列表

常用的做法就是对类数组对象做循环，例如想在dom列表上用foreach。

```javascript
var domList= document.querySelector(".box");
Array.prototype.forEach.call(domList, () =>{
  //do something
})
```

这里顺便摘抄一个大神的文章，关于apply的考点。

```javascript
log('hello world')
//定义log，然后它可以代理console.log的方法
//这是简单的问题
function log(msg)　{
  console.log(msg)
}
```

```javascript
//我要传多个参数
log('hello', 'world')
//更加6的直接使用 apply
function log() {
  console.log.apply(this, arguments)
}
```

```javascript
log('hello', 'world')
// 输入
'(foo) hello world'
// 输出这样的结果
// arugments 是一个伪数组，这又要用到call或者apply了
// 这是大佬的答案
function log(){
  const args = Array.prototype.slice.call(arguments);
  args.unshift('(foo)');
  console.log.apply(console, args);
}
```

```javascript
//我觉得能不能干脆就这样，不用转换成数组对象，直接unshift
function log(){
  const args = arguments
  Array.prototype.unshift.call(args , '(foo)')
  console.log.apply(this, args)
}
//ok，这样是可以的
```

补充下bind的用法，bind的作用和call已经apply也类似，都能绑定指定this的指向。

> bind()方法创建一个新的函数, 当被调用时，将其this关键字设置为提供的值，在调用新函数时，在任何提供之前提供一个给定的参数序列。

意思就是bind函数会创建一个函数，这个函数的this指向会指向传入bind函数的参数

bind返回一个更改了this指向的函数，并不会像call和apply那样直接执行

上一个call的例子，如果用bind来写就是这样

```javascript
function log(){
  const args = arguments
  Array.prototype.unshift.bind(args , '(foo)')()
  console.log.apply(this, args)
}
```
