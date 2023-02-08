---
title: 用css重置table的空隙
url: https://www.yuque.com/endday/blog/sw77su
---

```html
// 页面里table标签声明边框粗细为0、单元格边距为0、单元格间距为0：
<table border="0" cellspacing="0" cellpadding="0">
// cellspacing=0 cellpadding=0是属性，不是CSS样式，不能写在CSS里边的。
// 你要写在CSS的话，这样写
<style>
  table { border-collapse: collapse; padding: 0;}
</style>
// cellspacing是表格里单元格之间的距离;
// cellpadding是表格里单元格内的空白部分;
// 俗称就是外补丁和内补丁,类似应用在div和span上的margin和padding
```
