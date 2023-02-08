---
title: js原型与原型链
url: https://www.yuque.com/endday/blog/coq1qp
---

<a name="MPoM2"></a>

#### ![39025547-51be867a-447a-11e8-86c1-d2bfb8380c57.png](..\\..\assets\coq1qp\1558277279338-10ff9d70-6069-4fcc-9b0c-f8b5e124e89c.png)

```javascript
var a = {};
console.log(a.prototype);  //undefined
console.log(a.__proto__);  //Object {}
var b = function(){}
console.log(b.prototype);  //b {}
console.log(b.__proto__);  //function() {}
```

![39025631-e8f911cc-447a-11e8-9b79-b08ba556771b.png](..\\..\assets\coq1qp\1558277441453-5f0274ad-e7ca-4b29-a75f-00377305936e.png)

```javascript
/*1、字面量方式*/
var a = {};
console.log(a.__proto__);  //Object {}
console.log(a.__proto__ === a.constructor.prototype); //true
/*2、构造器方式*/
var A = function(){};
var a = new A();
console.log(a.__proto__); //A {}
console.log(a.__proto__ === a.constructor.prototype); //true
/*3、Object.create()方式*/
var a1 = {a:1}
var a2 = Object.create(a1);
console.log(a2.__proto__); //Object {a: 1}
console.log(a.__proto__ === a.constructor.prototype); //false（此处即为图1中的例外情况）
```

![39025547-51be867a-447a-11e8-86c1-d2bfb8380c57.png](..\\..\assets\coq1qp\1558277449440-22aab100-097a-4164-bc29-a93b99be09fd.png)

```javascript
var A = function(){};
var a = new A();
console.log(a.__proto__); //A {}（即构造器function A 的原型对象）
console.log(a.__proto__.__proto__); //Object {}（即构造器function Object 的原型对象）
console.log(a.__proto__.__proto__.__proto__); //null
```
