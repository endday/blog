---
title: js面向对象基础
url: https://www.yuque.com/endday/blog/ruhspw
---

<a name="b98e9d7c"></a>

## 工厂模式

工厂模式是软件工程领域一种广为人知的设计模式，而由于在ECMAScript中无法创建类，因此用函数封装以特定接口创建对象。其实现方法非常简单，也就是在函数内创建一个对象，给对象赋予属性及方法再将对象返回即可。

    function person(name, address) {
      let obj = new Object()
      obj.name = name
      obj.address = address
      obj.sayName = function () {
        console.log(this.name)
      }
      return obj
    }

    const tom = person('tom', 'shenzhen')

可以看到工厂模式的实现方法非常简单，解决了创建多个相似对象的问题，但是工厂模式却无从识别对象的类型，因为全部都是Object，不像Date、Array等，因此出现了构造函数模式。

<a name="2263c1e9"></a>

## 构造函数模式

ECMAScript中构造函数可以创建特定类型的对象，类似于Array、Date等原生JS的对象。其实现方法如下：

    function Blog(name, url) {
        this.name = name;
        this.url = url;
        this.alertUrl = function() {
            alert(this.url);
        }
    }

    var blog = new Blog('wuyuchang', 'http://www.cnblogs.com/wuyuchang/');
    console.log(blog instanceof Blog);    // true， 判断blog是否是Blog的实例，即解决了工厂模式中不能识别是对象的类型的问题

这个例子与工厂模式中除了函数名不同以外，细心的童鞋应该发现许多不同之处：

函数名首写字母为大写（虽然标准没有严格规定首写字母为大写，但按照惯例，构造函数的首写字母用大写

没有显示的创建对象

直接将属性和方法赋值给了this对象

没有return语句

使用new创建对象

能够识别对象，这正是构造函数模式胜于工厂模式的地方

构造函数虽然好用，但也并非没有缺点，使用构造函数的最大的问题在于每次创建实例的时候都要重新创建一次方法（理论上每次创建对象的时候对象的属性均不同，而对象的方法是相同的），然而创建两次完全相同的方法是没有必要的，因此，我们可以将函数移到对象外面

这里我说下自己的理解

MDN对于instanceof 的定义这样的

> instanceof 运算符用来检测 constructor.prototype 是否存在于参数 object 的原型链上。
>
> 一般可以用来判断对象的类型

    function Animal(name) {
      this.name = name
    }

    const cat = new Animal('kitty')
    const isAnimal = cat instanceof Animal    // 返回 true
    const isObject = cat instanceof Object // 返回 true

cat既属于Animal类型，又属于Object类型。

工厂函数都是new object()，类型都是object。

构造函数的new 函数是在实例化对象时调用的，那就能判断对象类型了。

<a name="ec3c9c8f"></a>

## 原型模式

<a name="3e2a920c"></a>

## 混合模式（原型模式  + 构造函数模式）

<a name="c9f17664"></a>

## 动态原型模式

（全文基本都是摘抄，如有雷同，请勿见怪）
