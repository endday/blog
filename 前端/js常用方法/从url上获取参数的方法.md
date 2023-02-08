---
title: 从url上获取参数的方法
url: https://www.yuque.com/endday/blog/elv32g
---

```javascript
getQueryString (name, url) {
  url = url || window.location.href
  const r = new RegExp('(\\?|#|&)' + name + '=([^&#]*)(&|#|$)')
  const m = url.match(r)
  return decodeURIComponent(!m ? '' : m[2])
}
```
