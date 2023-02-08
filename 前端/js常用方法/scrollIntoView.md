---
title: scrollIntoView
url: https://www.yuque.com/endday/blog/xrs4u5
---

elementui中的写法

```javascript
scrollIntoView (container, selected) {
  if (!selected) {
    container.scrollTop = 0
    return
  }

  const offsetParents = []
  let pointer = selected.offsetParent
  // 获取目标节点的上层节点，
  // 层层遍历，依次将目标节点的外层节点push到数组中
  // Node.contains()返回的是一个布尔值，来表示传入的节点是否为该节点的后代节点
  while (pointer && container !== pointer && container.contains(pointer)) {
    // 当遍历到上层节点为容器container时终止
    offsetParents.push(pointer)
    pointer = pointer.offsetParent
  }
  // reduce出目标节点距离容器的top距离
  const top = offsetParents.reduce((prev, curr) => (prev + curr.offsetTop), selected.offsetTop)
  // bottom距离即为top距离+目标节点的高度
  const bottom = top + selected.offsetHeight
  // 容器的已经滚动了的高度，指已经隐藏起来的部分的高度
  const viewRectTop = container.scrollTop
  // 容器展示的高度 + 容器已经滚动的距离
  const viewRectBottom = viewRectTop + container.clientHeight
  if (top < viewRectTop) {
    container.scrollTop = top
  } else if (bottom > viewRectBottom) {
    container.scrollTop = bottom - container.clientHeight
  }
}
```

简单粗暴的版本，直接将目标节点滚动到容器中心

```javascript
if (!selected) {
    container.scrollTop = 0
    return
  }

  const offsetParents = []
  let pointer = selected.offsetParent
  // 获取目标节点的上层节点，
  // 层层遍历，依次将目标节点的外层节点push到数组中
  // Node.contains()返回的是一个布尔值，来表示传入的节点是否为该节点的后代节点
  while (pointer && container !== pointer && container.contains(pointer)) {
    // 当遍历到上层节点为容器container时终止
    offsetParents.push(pointer)
    pointer = pointer.offsetParent
  }
  // reduce出目标节点距离容器的top距离
  const top = offsetParents.reduce((prev, curr) => (prev + curr.offsetTop), selected.offsetTop)
  container.scrollTop = top - (container.clientHeight / 2) + selected.clientHeight
}
```
