---
title: Typescript笔记
url: https://www.yuque.com/endday/blog/wz87ic
---

<a name="b9FiU"></a>

#### typescript里 泛型中 & 是什么意思 ？

[Intersection Types](https://www.typescriptlang.org/docs/handbook/advanced-types.html#intersection-types)
An intersection type combines multiple types into one. This allows you to add together existing types to get a single type that has all the features you need. For example, > `Person & Serializable & Loggable` is a > `Person` and > `Serializable` and > `Loggable`. That means an object of this type will have all members of all three types.
表示这个变量同时拥有所有类型所需要的成员，可以作为其中任何一个类型使用。

<a name="3OwVr"></a>

#### interface中的extends

```javascript
// 生物体的接口
interface Creature {
 name: string;
}
// 动物接口 继承生物接口
interface Animal extends Creature {
 // 自己拥有的属性 action
 action(): void;
}
```

一个接口可以继承多个接口

```javascript
// 生物体的接口
interface Creature {
 name: string;
}
// 动物接口
interface Animal {
 // 自己拥有的属性 action
 action(): void;
}
// 狗Dog接口继承 生物Creature 和 Animal 多继承
interface Dog extends Creature, Animal{
 color: string;
}
```
