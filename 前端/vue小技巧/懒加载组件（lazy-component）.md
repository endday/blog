---
title: 懒加载组件（lazy-component）
url: https://www.yuque.com/endday/blog/qvg47b
---

```javascript
const map = new WeakMap()

export default {
  name: 'lazy-component',
  functional: true,
  props: {
    show: {
      type: Boolean,
      default: false
    }
  },
  render (h, context) {
    const children = context.children
    const hasTarget = map.has(children)
    if (!hasTarget && context.props.show) {
      map.set(children, true)
    }
    if (hasTarget) {
      return h('div', context.data, context.children)
    } else {
      return h('div', context.data, [])
    }
  }
}

```

```html
// 使用

<create-once :show="showOrNot">
  <custom-component/>
</create-once>
```
