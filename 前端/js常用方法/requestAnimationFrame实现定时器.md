---
title: requestAnimationFrame实现定时器
url: https://www.yuque.com/endday/blog/xb6d1x
---

```javascript
const rafInterval = (fn, delay = 16, count) => {
  let lastRunTime
  let timerId = null
  let runCount = 0
  const loop = (timestamp) => {
    if (lastRunTime === undefined) {
      lastRunTime = timestamp
    }
    timerId = requestAnimationFrame(loop)
    if (timestamp - lastRunTime >= delay) {
      if (count && runCount >= count) {
        cancelAnimationFrame(timerId)
      } else {
        runCount++
        fn(timestamp)
        lastRunTime = timestamp
      }
    }
  }
  timerId = requestAnimationFrame(loop)
  return timerId
}

let id
let count = 0
let b = (t) => {
  console.log(count, t)
  count++
  id = rafInterval(b, 2000, 1)
  if (count > 8) {
    window.cancelAnimationFrame(id)
  }
}
b()
```
