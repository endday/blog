---
title: vue组件方法劫持的探索
url: https://www.yuque.com/endday/blog/azryig
---

<a name="jv6CZ"></a>

#### 待劫持的组件

```javascript
<template>
  <button @click="handleClick">
    <slot></slot>
  </button>
</template>

<script>
export default {
  name: 'XButton',
  methods: {
    handleClick (event) {
      this.$emit('click', event)
    }
  }
}
</script>

<style scoped>

</style>

```

<a name="AiURP"></a>

#### 劫持的逻辑

```javascript
import xButton from '../components/button'

// 防抖函数
function debounce (func, delay, context, args) {
  clearTimeout(func.timer)
  func.timer = setTimeout(function () {
    func.call(context, ...args)
  }, delay)
}

function proxy (func) {
  return function () {
    console.log('debounce')
    debounce(func, 300, this, arguments)
  }
}

// 导出新组件
export default {
  functional: true,
  render (h, context) {
    context.listeners.click = proxy(context.listeners.click)
    return h(xButton, context.data, context.children)
  }
}
```
