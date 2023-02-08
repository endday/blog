---
title: el-overlay遮罩层组件设计思路
url: https://www.yuque.com/endday/blog/gr277y
---

判断点击位置不在遮罩层
判断点击位置是否有拖动

```javascript
let mousedownTarget = false
let mouseupTarget = false
const onMaskClick = (e) => {
  if (mousedownTarget && mouseupTarget) {
    visible.value = false
  }
  mousedownTarget = mouseupTarget = false
}
const onMouseDown = (e) => {
  mousedownTarget = e.target === e.currentTarget
}
const onMouseUp = (e) => {
  mouseupTarget = e.target === e.currentTarget
}
```
