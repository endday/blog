---
title: 判断是否json
url: https://www.yuque.com/endday/blog/mupa4g
---

```javascript
function isJson (str) {
    if (typeof str !== 'string') {
        return false
    }
    const char = str.charAt(0)
    if (char !== '[' && char !== '{') {
        return false
    }
    try {
        return typeof JSON.parse(str) === 'object'
    } catch (e) {
        return false
    }
}
```
