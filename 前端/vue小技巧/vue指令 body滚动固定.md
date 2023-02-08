---
title: vue指令 body滚动固定
url: https://www.yuque.com/endday/blog/zo8en9
---

```javascript
export default {
  componentUpdated (el, binding) {
    if (binding.value === undefined) {
      return
    }
    let body = document.body
    if (binding.value) {
      let scrollTop = body.scrollTop || document.documentElement.scrollTop
      document.body.style.cssText += 'position:fixed;width:100%;top:-' + scrollTop + 'px;'
    } else {
      body.style.position = ''
      let top
      if (body.style.top) {
        top = body.style.top
      } else {
        top = document.body.scrollTop || document.documentElement.scrollTop
      }
      document.body.scrollTop = document.documentElement.scrollTop = Math.abs(parseInt(top))
      body.style.top = ''
    }
  }
}

```
