---
title: observer/dep.js
url: https://www.yuque.com/endday/blog/wsx01g
---

![](..\\..\\..\assets\wsx01g\1599012455204-68ada1c6-56dc-4068-a935-ecb5657049fb.png)

```javascript
/* @flow */

import type Watcher from './watcher'
import {
  remove
} from '../util/index'
import config from '../config'

let uid = 0

/**
 * A dep is an observable that can have multiple
 * directives subscribing to it.
 */
export default class Dep {
  static target: ? Watcher; // 静态属性，其实就是全局的一个变量
  id: number;
  subs: Array < Watcher > ;

  constructor() {
    this.id = uid++
    this.subs = []
  }
  /*添加一个观察者对象*/
  addSub(sub: Watcher) {
    this.subs.push(sub)
  }
  /*移除一个观察者对象*/
  removeSub(sub: Watcher) {
    remove(this.subs, sub)
  }
  /*
   * 依赖收集，当存在Dep.target的时候添加观察者对象
   * Dep.target是一个全局的对象,是watcher的一个实例
   * 所以会有addDep方法，把watcher本身push到dep.subs
   * */
  depend() {
    if (Dep.target) {
      Dep.target.addDep(this)
    }
  }
  /*通知所有订阅者*/
  notify() {
    // stabilize the subscriber list first
    const subs = this.subs.slice()
    if (process.env.NODE_ENV !== 'production' && !config.async) {
      // subs aren't sorted in scheduler if not running async
      // we need to sort them now to make sure they fire in correct
      // order
      // 如果不是异步运行，subs在调度器中没有排序，
      // 我们现在需要对它们进行排序，以确保它们以正确的顺序启动。
      subs.sort((a, b) => a.id - b.id)
    }
    for (let i = 0, l = subs.length; i < l; i++) {
      subs[i].update()
    }
  }
}

// The current target watcher being evaluated.
// This is globally unique because only one watcher
// can be evaluated at a time.

/*
  当前target的watcher依赖正在被收集，
  一次只会收集一个watcher，因为Dep.target是全局的唯一变量
 */
Dep.target = null
const targetStack = []

/*将watcher观察者实例设置给Dep.target，用以依赖收集。同时将该实例存入target栈中*/
export function pushTarget(target: ? Watcher) {
  targetStack.push(target)
  Dep.target = target
}
/*存疑
* 出栈的一个watcher
* 和赋值Dep.target的明显不是一个watcher
* 在生命周期方法和data的初始化中，都会禁用依赖收集
* 使用的方式就是在Dep.target中放入一个undefined
* 具体的原因，在子组件进行data初始化时，触发到getter会调用addDep
* 会收集到父组件的render watcher，
* 导致子组件更改 会令父组件触发更新，即使父组件没有更改
* https://github.com/vuejs/vue/commit/318f29fcdf3372ff57a09be6d1dc595d14c92e70#diff-7060e3d9c5c616dbceee1f0d33ca5ff0
* */
export function popTarget() {
  targetStack.pop()
  Dep.target = targetStack[targetStack.length - 1]
}

```

```javascript
// https://github.com/vuejs/vue/commit/318f29fcdf3372ff57a09be6d1dc595d14c92e70#diff-7060e3d9c5c616dbceee1f0d33ca5ff0

it('props should not be reactive', done => {
    let calls = 0
    const vm = new Vue({
      template: `<child :msg="msg"></child>`,
      data: {
        msg: 'hello'
      },
      beforeUpdate () { calls++ },
      components: {
        child: {
          template: `<span>{{ localMsg }}</span>`,
          props: ['msg'],
          data () {
            return { localMsg: this.msg }
          },
          computed: {
            computedMsg () {
              return this.msg + ' world'
            }
          }
        }
      }
    }).$mount()
    const child = vm.$children[0]
    vm.msg = 'hi'
    waitForUpdate(() => {
      expect(child.localMsg).toBe('hello')
      expect(child.computedMsg).toBe('hi world')
      expect(calls).toBe(1)
    }).then(done)
  })
```
