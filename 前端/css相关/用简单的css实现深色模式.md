---
title: 用简单的css实现深色模式
url: https://www.yuque.com/endday/blog/eh8g7k
---

[几行CSS让整站支持深色模式的探索与拓展](https://www.zhangxinxu.com/wordpress/2020/11/css-mix-blend-mode-filter-dark-theme/)

```css
@media (prefers-color-scheme: dark) {
    html, img { 
        filter: invert(1) hue-rotate(180deg);
    }
    .some-ele {
        box-shadow: none;
    }
}
```
