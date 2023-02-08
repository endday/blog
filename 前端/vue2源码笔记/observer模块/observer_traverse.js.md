---
title: observer/traverse.js
url: https://www.yuque.com/endday/blog/gx46yv
---

```javascript
/* @flow */

import { _Set as Set, isObject } from '../util/index'
import type { SimpleSet } from '../util/index'
import VNode from '../vdom/vnode'

const seenObjects = new Set()

/**
 * Recursively traverse an object to evoke all converted
 * getters, so that every nested property inside the object
 * is collected as a "deep" dependency.
 *
 * 深度监听，就是watch的deep选项
 * 递归每一个对象或者数组，
 * 触发它们的getter，
 * 使得对象或数组的每一个成员都被依赖收集，
 * 形成一个“深（deep）”依赖关系
 */
export function traverse (val: any) {
  _traverse(val, seenObjects)
  //清除对象 给对象置空
  seenObjects.clear()
}

function _traverse (val: any, seen: SimpleSet) {
  let i, keys
  const isA = Array.isArray(val)
  //isFrozen 方法判断一个对象是否被冻结。
  //val 是否是被VNode 实例化
  // 如果对象已经被冻结，或者是vNode，就直接返回
  if ((!isA && !isObject(val)) || Object.isFrozen(val) || val instanceof VNode) {
    return
  }
  if (val.__ob__) {
    const depId = val.__ob__.dep.id
    if (seen.has(depId)) {
      return
    }
    // seen 是 seenObjects = new _Set(); add 就是set对象中的add方法，添加为一的值得key
    //如果没有则添加进去
    seen.add(depId)
  }
  //如果是数组
  if (isA) {
    i = val.length
    //则循环检查 回调递归
    while (i--) _traverse(val[i], seen)
  } else {
    keys = Object.keys(val)
    i = keys.length
    //如果是对象也循环递归检查
    while (i--) _traverse(val[keys[i]], seen)
  }
}

```
