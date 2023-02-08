---
title: Object.create
url: https://www.yuque.com/endday/blog/dgfpsn
---

```javascript
if (typeof Object.create !== 'function') {
  Object.create = function (proto, propertiesObject) {
    if (typeof proto !== 'object' && typeof proto !== 'function') {
      throw new TypeError('Object prototype may only be an Object: ' + proto)
    } else if (proto === null) {
      throw new Error('This browser\'s implementation of Object.create is a shim and doesn\'t support \'null\' as the first argument.')
    }

    function F () {
    }

    F.prototype = proto

    if (propertiesObject) {
      Object.defineProperties(F, propertiesObject)
    }

    return new F()
  }
}

```
