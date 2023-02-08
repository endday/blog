---
title: 十分经典的js题目解析
url: https://www.yuque.com/endday/blog/mdxykf
---

```javascript
function Foo() {
  getName = function () {
    alert(1)
  }
  return this
}

Foo.getName = function () {
  alert(2)
}

Foo.prototype.getName = function () {
  alert(3)
}

var getName = function () {
  alert(4)
}

function getName() {
  alert(5)
}
 
//请写出以下输出结果：
Foo.getName();
getName();
Foo().getName();
getName();
new Foo.getName();
new Foo().getName();
new new Foo().getName();
```

<a name="87f2fd53"></a>

## 第一问

`Foo.getName();`

这个不用多说，返回2.

执行的是Foo函数的静态属性getName方法

这里复习下一些定义

```javascript
function User(name) {
  var name = name //私有属性
  this.name = name //公有属性
  function getName() { //私有方法
    return name
  }
}

User.prototype.getName = function () { //公有方法
  return this.name
}
User.name = 'Wscats' //静态属性
User.getName = function () { //静态方法
  return this.name
}
var Wscat = new User('Wscats') //实例化
```

- 调用公有方法，公有属性，我们必需先实例化对象，也就是用new操作符实化对象，就可构造函数实例化对象的方法和属性，并且公有方法是不能调用私有方法和静态方法的
- 静态方法和静态属性就是我们无需实例化就可以调用
- 而对象的私有方法和属性,外部是不可以访问的

个人理解：

公有方法必须实例化才能访问，是因为查找对象的属性时，如果该对象没有这个属性，会在原型链上查找，也就是会在对象的**proto**上继续查找，但是不会查找自身的prototype，而**proto**指向的是这个对象的构造函数的prototype。

也就是说，通过构造函数实例化一个对象后，这个对象就可以通过**proto**访问到构造函数的prototype上的方法。

<a name="5a22dc26"></a>

## 第二问

直接调用getName函数。

这个明显是只和4和5有关了

4是函数表达式，5是函数声明

这里有个陷阱

> JavaScript 解释器中存在一种变量声明被提升的机制，也就是说函数声明会被提升到作用域的最前面，即使写代码的时候是写在最后面，也还是会被提升至最前面。
>
> 而用函数表达式创建的函数是在运行时进行赋值，且要等到表达式赋值完成后才能调用

```javascript
getName() // Uncaught TypeError: getName is not a function
var getName = function () {
  alert(4)
}
getName() // 4
```

函数表达式的值是在JS运行时确定，并且在表达式赋值完成后，该函数才能调用

```javascript
getName() // 5 函数提升了
function getName() {
  alert(5)
}
```

函数声明在JS解析时进行函数提升，因此在同一个作用域内，不管函数声明在哪里定义，该函数都可以进行调用

```javascript
getName() // 5 函数提升了
var getName = function () {
  alert(4)
}
getName() // 4 被覆盖了
function getName() {
  alert(5)
}
getName() // 4
```

这题的考核就是在这个地方，变量提升

<a name="06ed3d44"></a>

## 第三问

`Foo().getName()`先执行Foo()，再调用Foo()的返回值调用getName()

```javascript
function Foo() {
  getName = function () {
    alert(1)
  }
  return this
}
```

调用Foo()，先从函数作用域内寻找getName这个变量，没有找到，再向当前函数作用域上层，找到了，第四个函数定义的getName会被覆盖。

> 上面说的这个重复向上层作用域寻找函数变量的过程，如果到window对象都没有找到，就会直接创建一个全局的getName变量。

接下来返回this，这里直接调用Foo()，this直接指向window。

> 全局作用域或者普通函数中this指向全局对象window
>
> 方法调用中谁调用this指向谁

那么执行`Foo().getName()`就相当于`window.getName()`，直接执行第四个函数，但是第四个函数刚刚已经被覆盖了，所以返回的是1。

这道题考察的一个是变量作用域，一个是this指向。

<a name="29233ab6"></a>

## 第四问

直接调用getName()，也就是window.getName()， 第一问的Foo函数已经把第五个函数给覆盖了，所以返回的结果是1。

<a name="9d47105c"></a>

## 第五问

new Foo.getName()

这题考察的是JS的运算符优先级问题，这是我做这道题之前都没有去了解过的知识点。

下面是JS运算符的优先级表格，从高到低排列。可参考[MDN运算符优先级](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/Operator_Precedence)

| 优先级 | 运算类型 | 关联性 | 运算符 |
| --- | --- | --- | --- |
| 19 | 圆括号 | n/a | ( … ) |
| 18 | 成员访问 | 从左到右 | … . … |
| 需计算的成员访问 | 从左到右 | … \[ … ] |  |
| new (带参数列表) | n/a	new | … ( … ) |  |
| 17 | 函数调用 | 从左到右 | … ( … ) |
| new (无参数列表) | 从右到左 | new … |  |
| 16 | 后置递增(运算符在后) | n/a | … ++ |
| 后置递减(运算符在后) | n/a | … -- |  |
| 15 | 逻辑非 | 从右到左 | ! … |
| 按位非 | 从右到左 | ~ … |  |
| 一元加法 | 从右到左 | + … |  |
| 一元减法 | 从右到左 | - … |  |
| 前置递增 | 从右到左 | ++ … |  |
| 前置递减 | 从右到左 | -- … |  |
| typeof | 从右到左 | typeof … |  |
| void | 从右到左 | void … |  |
| delete | 从右到左 | delete … |  |
| 14 | 乘法 | 从左到右 | … * … |
| 除法 | 从左到右 | … / … |  |
| 取模 | 从左到右 | … % … |  |
| 13 | 加法 | 从左到右 | … + … |
| 减法 | 从左到右 | … - … |  |
| 12 | 按位左移 | 从左到右 | … << … |
| 按位右移 | 从左到右 | … >> … |  |
| 无符号右移 | 从左到右 | … >>> … |  |
| 11 | 小于 | 从左到右 | … < … |
| 小于等于 | 从左到右 | … <= … |  |
| 大于 | 从左到右 | … > … |  |
| 大于等于 | 从左到右 | … >= … |  |
| in | 从左到右 | … in … |  |
| instanceof | 从左到右 | … instanceof … |  |
| 10 | 等号 | 从左到右 | … == … |
| 非等号 | 从左到右 | … != … |  |
| 全等号 | 从左到右 | … === … |  |
| 非全等号 | 从左到右 | … !== … |  |
| 9 | 按位与 | 从左到右 | … & … |
| 8 | 按位异或 | 从左到右 | … ^ … |
| 7 | 按位或 | 从左到右 | … 按位或 … |
| 6 | 逻辑与 | 从左到右 | … && … |
| 5 | 逻辑或 | 从左到右 | … 逻辑或 … |
| 4 | 条件运算符 | 从右到左 | … ? … : … |
| 3 | 赋值 | 从右到左 | … = … |
|  |  | … += … |  |
|  |  | … -= … |  |
|  |  | … *= … |  |
|  |  | … /= … |  |
|  |  | … %= … |  |
|  |  | … <<= … |  |
|  |  | … >>= … |  |
|  |  | … >>>= … |  |
|  |  | … &= … |  |
|  |  | … ^= … |  |
|  |  | … 或= … |  |
| 2 | yield | 从右到左 | yield … |
| yield* | 从右到左 | yield* … |  |
| 1 | 展开运算符 | n/a | ... … |
| 0 | 逗号 | 从左到右 | … , … |

new (带参数列表)比new (无参数列表)高比函数调用高，跟成员访问同级

```javascript
new Foo.getName()
(new (Foo.getName))() //相当于
```

1. 执行成员访问，找到Foo对象的getName属性
2. new 构造函数实例化
3. () 调用函数 （这里的()是个陷阱，new Foo.getName 并不是返回一个函数，而是对象，所以这里的调用函数会报错）

点的优先级(18)比new无参数列表(17)优先级高

当点运算完后又因为有个括号()，此时就是变成new有参数列表(18)，所以直接执行new，当然也可能有朋友会有疑问为什么遇到()不函数调用再new呢，那是因为函数调用(17)比new有参数列表(18)优先级低

最后结果就是2

<a name="d9525693"></a>

## 第六问

```javascript
new Foo().getName()
```

new有参数列表(18)跟点的优先级(18)是同级，同级的话按照从左向右的执行顺序

```javascript
(new Foo()).getName()
```

构造函数的返回值

在传统语言中，构造函数不应该有返回值，实际执行的返回值就是此构造函数的实例化对象。

而在JS中构造函数可以有返回值也可以没有。

没有返回值则按照其他语言一样返回实例化对象。

```javascript
function Foo (name) {
 this.name = name
}

console.log(new Foo('wscats'))
```

若有返回值则检查其返回值是否为引用类型。如果是非引用类型，如基本类型（String,Number,Boolean,Null,Undefined）则与无返回值相同，实际返回其实例化对象。

```javascript
function Foo (name) {
  this.name = name
  return 520
}

console.log(new Foo('wscats'))
```

若返回值是引用类型，则实际返回值为这个引用类型。

```javascript
function Foo (name) {
  this.name = name
  return {
    age: 16
  }
}

console.log(new Foo('wscats'))
```

原题中，由于返回的是this，而this在构造函数中本来就代表当前实例化对象，最终Foo函数返回实例化对象。

之后调用实例化对象的getName函数，因为在Foo构造函数中没有为实例化对象添加任何属性，当前对象的原型对象(prototype)中寻找getName函数。

答案是3

[參考地址](https://github.com/Wscats/Good-text-Share/issues/85)
