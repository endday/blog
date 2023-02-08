---
title: 证明js中的函数参数是值传递
url: https://www.yuque.com/endday/blog/rev8zx
---

```javascript
function setName1 (name) {
	name = 'david'
}

const name = 'donad'
setName1(name)
console.log(name)

function setName2 (obj) {
    obj.name = 'jarvis'
    obj = new Object()
    obj.name = 'jax'
}

const person = {name: 'donad'}
setName2(person)
console.log(person.name)
```
