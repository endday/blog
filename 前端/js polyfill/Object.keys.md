---
title: Object.keys
url: https://www.yuque.com/endday/blog/oc72b1
---

```typescript
if (!Object.keys) {
  Object.keys = function (o) {
    if (o !== Object(o)) { throw TypeError('Object.keys called on non-object'); }
    var ret = [], p;
    for (p in o) {
      // 检测一个对象是否含有特定的自身属性
      // 忽略掉那些从原型链上继承到的属性
      if (Object.prototype.hasOwnProperty.call(o, p)) {
        ret.push(p);
      }
    }
    return ret;
  };
}
```
