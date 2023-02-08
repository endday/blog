---
title: js类型判断
url: https://www.yuque.com/endday/blog/cx7e55
---

<a name="WOiUd"></a>

## vue的类型判断

```javascript
/*
	判断数据类型
	@param s {Object Number Function Symbol}
*/
export const type = s => Object.prototype.toString.call(s).slice(8, -1).toLowerCase();

[
  'String',
	'Array',
	'Undefined',
	'Boolean',
	'Number',
	'Function',
	'Symbol',
	'Object'
].forEach(v=>type['is'+v] = s => type(s) === v.toLowerCase())

eg: type.isNumber(123) // true
    type.isString(123) // false
```

<a name="6MUbh"></a>

## es5代码

```javascript
var type = function (s) {
    return Object.prototype.toString.call(s).slice(8, -1).toLowerCase();
};

var types = [
    'String',
    'Array',
    'Undefined',
    'Boolean',
    'Number',
    'Function',
    'Symbol',
    'Object'
];

types.forEach(function (str) {
    type['is' + str] = function (val) {
        return type(val) === str.toLowerCase();
    };
});

module.exports = type;
```
