---
title: TamperMonkey 脚本开发笔记
url: https://www.yuque.com/endday/blog/zytais
---

[Greasemonkey文档](https://wiki.greasespot.net/Greasemonkey_Manual:API)
[violentmonkey文档](https://violentmonkey.github.io/api/gm/)

TamperMonkey中有`GM_addStyle`方法，Greasemonkey没有这个方法，所以要做兼容

为了兼容没有addStyle

```javascript
export function addStyle (styleContent) {
  if (!styleContent) {
    return
  }
  // eslint-disable-next-line
  if (window.GM_addStyle) {
    // eslint-disable-next-line
    window.GM_addStyle(styleContent)
  } else {
    const style = document.createElement('style')
    style.innerHTML = styleContent
    const head = document.getElementsByTagName('head')[0]
    head.appendChild(style)
  }
}
```
