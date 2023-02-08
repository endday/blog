---
title: 滚动至指定dom的位置
url: https://www.yuque.com/endday/blog/daqpge
---

```javascript
const el = document.getElementById('example')
window.scrollBy(0, el.getBoundingClientRect().top)
```
