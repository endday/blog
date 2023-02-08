---
title: observer/scheduler.js
url: https://www.yuque.com/endday/blog/rgonty
---

<a name="zFifp"></a>

### 重点关注

<a name="0RdiS"></a>

#### flushSchedulerQueue

<a name="6jlCu"></a>

#### queueWatcher

```javascript
/* @flow */

import type Watcher from './watcher'
import config from '../config'
import { callHook, activateChildComponent } from '../instance/lifecycle'

import {
  warn,
  nextTick,
  devtools,
  inBrowser,
  isIE
} from '../util/index'

export const MAX_UPDATE_COUNT = 100

const queue: Array<Watcher> = [] //记录观察者队列的数组
const activatedChildren: Array<Component> = []  //记录活跃的子组件
/*一个哈希表，用来存放watcher对象的id，防止重复的watcher对象多次加入*/
let has: { [key: number]: ?true } = {}
let circular: { [key: number]: number } = {} //持续循环更新的次数，如果超过100次 则判断已经进入了死循环，则会报错
let waiting = false //观察者在更新数据时候 等待的标志
let flushing = false //进入flushSchedulerQueue 函数等待标志
let index = 0 //queue 观察者队列的索引

/**
 * Reset the scheduler's state.
 */
/*重置调度者的状态*/
function resetSchedulerState () {
  index = queue.length = activatedChildren.length = 0
  has = {}
  if (process.env.NODE_ENV !== 'production') {
    circular = {}
  }
  waiting = flushing = false
}

// Async edge case #6566 requires saving the timestamp when event listeners are
// attached. However, calling performance.now() has a perf overhead especially
// if the page has thousands of event listeners. Instead, we take a timestamp
// every time the scheduler flushes and use that for all event listeners
// attached during that flush.
/*
* 异步边缘事件＃6566需要在连接事件侦听器时保存时间戳。
* 但是，调用performance.now（）会产生性能开销，特别是在页面具有数千个事件侦听器的情况下。
* 取而代之的是，每次调度程序刷新时，我们都会使用一个时间戳，
* 并将其用于该刷新期间附加的所有事件侦听器。
* */
export let currentFlushTimestamp = 0

// Async edge case fix requires storing an event listener's attach timestamp.
// 获取时间戳
let getNow: () => number = Date.now

// Determine what event timestamp the browser is using. Annoyingly, the
// timestamp can either be hi-res (relative to page load) or low-res
// (relative to UNIX epoch), so in order to compare time we have to use the
// same timestamp type when saving the flush timestamp.
// All IE versions use low-res event timestamps, and have problematic clock
// implementations (#9632)

//确定浏览器使用的是什么事件时间戳。
// 恼人的是，时间戳可以是hi-res（相对于页面加载）或low-res（相对于UNIX epoch），
// 所以为了比较时间，我们在保存刷新时间戳时必须使用相同的时间戳类型。
// 所有IE版本都使用low-res的事件时间戳，并且有问题的时钟实现(#9632)

if (inBrowser && !isIE) {
  const performance = window.performance
  if (
    performance &&
    typeof performance.now === 'function' &&
    getNow() > document.createEvent('Event').timeStamp
  ) {
    // if the event timestamp, although evaluated AFTER the Date.now(), is
    // smaller than it, it means the event is using a hi-res timestamp,
    // and we need to use the hi-res version for event listener timestamps as
    // well.
    /*
      如果事件的时间戳虽然是在Date.now()之后评估的，
      但比它小，这意味着事件使用的是hi-res时间戳，
      我们需要对事件监听器的时间戳也使用hi-res版本。
    * */
    getNow = () => performance.now()
  }
}

/**
 * Flush both queues and run the watchers.
 * 清空队列和运行watchers
 */
function flushSchedulerQueue () {
  currentFlushTimestamp = getNow()
  flushing = true
  let watcher, id

  // Sort queue before flush.
  // This ensures that:
  // 1. Components are updated from parent to child. (because parent is always
  //    created before the child)
  // 2. A component's user watchers are run before its render watcher (because
  //    user watchers are created before the render watcher)
  // 3. If a component is destroyed during a parent component's watcher run,
  //    its watchers can be skipped.
  /*
    在清空前对队列进行排序，这样做可以保证：
    1.组件更新的顺序是从父组件到子组件的顺序，因为父组件总是比子组件先创建。
    2.一个组件的user watchers比render watcher先运行，
    因为user watchers往往比render watcher更早创建
    3.如果一个组件在父组件watcher运行期间被销毁，它的watcher执行将被跳过。
  */
  queue.sort((a, b) => a.id - b.id)

  // do not cache length because more watchers might be pushed
  // as we run existing watchers
  // 这里不用index = queue.length;index > 0; index--的方式写
  // 是因为不要将length进行缓存，
  // 因为在执行处理现有watcher对象期间，
  // 更多的watcher对象可能会被push进queue
  for (index = 0; index < queue.length; index++) {
    watcher = queue[index]
    if (watcher.before) {
      watcher.before()
    }
    id = watcher.id
    /*将has的标记删除*/
    has[id] = null
    /*执行watcher*/
    watcher.run()
    // in dev build, check and stop circular updates.
    /*
      在测试环境中，检测watch是否在死循环中
      比如这样一种情况
      watch: {
        test () {
          this.test++;
        }
      }
      持续执行了一百次watch代表可能存在死循环
    */
    if (process.env.NODE_ENV !== 'production' && has[id] != null) {
      circular[id] = (circular[id] || 0) + 1
      if (circular[id] > MAX_UPDATE_COUNT) {
        warn(
          'You may have an infinite update loop ' + (
            watcher.user
              ? `in watcher with expression "${watcher.expression}"`
              : `in a component render function.`
          ),
          watcher.vm
        )
        break
      }
    }
  }

  // keep copies of post queues before resetting state
  /*在重置状态之前保留post队列的副本*/
  const activatedQueue = activatedChildren.slice()
  const updatedQueue = queue.slice()

  resetSchedulerState()

  // call component updated and activated hooks
  /*使子组件状态都改编成active同时调用activated钩子*/
  callActivatedHooks(activatedQueue)
  callUpdatedHooks(updatedQueue)

  // devtool hook
  /* istanbul ignore if */
  if (devtools && config.devtools) {
    devtools.emit('flush')
  }
}
/*调用updated钩子*/
function callUpdatedHooks (queue) {
  let i = queue.length
  while (i--) {
    const watcher = queue[i]
    const vm = watcher.vm //获取到虚拟dom
    //判断watcher与vm._watcher 相等 _isMounted已经更新触发了 mounted 钩子函数
    if (vm._watcher === watcher && vm._isMounted && !vm._isDestroyed) {
      callHook(vm, 'updated')
    }
  }
}

/**
 * Queue a kept-alive component that was activated during patch.
 * The queue will be processed after the entire tree has been patched.
 */
/*
 在patch期间被激活（activated）的keep-alive组件保存在队列中，
 队列将在整个树被patch之后处理
*/
export function queueActivatedComponent (vm: Component) {
  // setting _inactive to false here so that a render function can
  // rely on checking whether it's in an inactive tree (e.g. router-view)
  vm._inactive = false
  activatedChildren.push(vm)
}
/*使子组件状态都改编成active同时调用activated钩子*/
function callActivatedHooks (queue) {
  for (let i = 0; i < queue.length; i++) {
    queue[i]._inactive = true
    //判断是否有不活跃的组件 禁用他 如果有活跃组件则触发钩子函数activated
    activateChildComponent(queue[i], true /* true */)
  }
}

/**
 * Push a watcher into the watcher queue.
 * Jobs with duplicate IDs will be skipped unless it's
 * pushed when the queue is being flushed.
 */
/*
  将一个观察者对象push进观察者队列，
  在队列中已经存在相同的id则该观察者对象将被跳过，
  除非它是在队列被刷新时推送
*/
export function queueWatcher (watcher: Watcher) {
  /*获取watcher的id*/
  const id = watcher.id
  /*检验id是否存在，已经存在则直接跳过，不存在则标记哈希表has，用于下次检验*/
  if (has[id] == null) {
    has[id] = true
    if (!flushing) {
      /*如果队列没有处于清空状态，直接push到队列中即可*/
      queue.push(watcher)
    } else {
      // if already flushing, splice the watcher based on its id
      // if already past its id, it will be run next immediately.
      // 如果队列正在清空，根据watcher的id大小插入到队列
      // 如果watcher的id是最小的，将会排在队列的第一个，将会立即执行
      /*
      * 在遍历的时候每次都会对 queue.length 求值，
      * 因为在 watcher.run() 的时候，很可能用户会再次添加新的watcher，
      * 也就是会再次执行queueWatcher方法
      * */
      let i = queue.length - 1
      while (i > index && queue[i].id > watcher.id) {
        i--
      }
      queue.splice(i + 1, 0, watcher)
    }
    // queue the flush
    if (!waiting) {
      // 如果没有重置队列，就不执行
      waiting = true
      /*
        异步执行更新。目的是由Vue Test Utils使用。
        如果设置为 "false"，将大大降低性能。
      */
      if (process.env.NODE_ENV !== 'production' && !config.async) {
        flushSchedulerQueue()
        return
      }
      // 异步执行watcher队列
      nextTick(flushSchedulerQueue)
    }
  }
}

/*
  将watcher插入队列的方法
  id重复的会跳过，id较小的会被按照顺序排列到最前
  保证id最小的watcher最先执行更新

  每次插入watcher就会去清空队列，为了避免反复执行清空
  使用waiting作为是否在清空队列的标记

  flushing也是表示正在清空队列
  当清空队列时，也可以插入watcher，但是要排序
  不在清空状态时，就直接push
* */

```
