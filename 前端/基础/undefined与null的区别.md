---
title: undefined与null的区别
url: https://www.yuque.com/endday/blog/mospw9
---

<a name="Undefined"></a>

### Undefined

Undefined是一种数据类型，只有一个值， 即特殊的undefined。

undefined是全局对象的一个属性。也就是说，它是全局作用域的一个变量。

undefined的最初值就是原始数据类型undefined。

一个没有被赋值的变量的类型是undefined。

如果方法或者是语句中操作的变量没有被赋值，则会返回undefined。

```javascript
let x
x === undefined === true
typeof x === 'undefined' === true
```

- 变量被声明了，但没有赋值时，就等于undefined。
- 调用函数时，应该提供的参数没有提供，该参数等于undefined。
- 对象没有赋值的属性，该属性的值为undefined。
- 函数没有返回值时，默认返回undefined。

<a name="Null"></a>

### Null

Null类型，只有一个特殊的值null。

值 null 是一个字面量，它不像undefined 是全局对象的一个属性。null 是表示缺少的标识，指示变量未指向任何对象。

从逻辑角度来说，null值表示一个空对象指针，而这也正是typeof操作符检测null值是返回‘object’的原因

```javascript
typeof null        // object (因为一些以前的原因而不是'null')
typeof undefined   // "undefined"
null === undefined // false
null == undefined // true
null === null // true
null == null // true
!null //true
isNaN(1 + null) // false
isNaN(1 + undefined) // true
```
