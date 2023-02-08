---
title: observer/watcher.js
url: https://www.yuque.com/endday/blog/dvck2s
---

<a name="FwEi6"></a>

# watcher

Watcher类的实现比较复杂，因为他的实例分为渲染 watcher（render-watcher）、计算属性 watcher（computed-watcher）、侦听器 watcher（normal-watcher）三种， 
这三个实例分别是在三个函数中构建的：mountComponent 、initComputed和Vue.prototype.$watch。 <a name="XYM4m"></a>

#### normal-watcher

我们在组件钩子函数watch 中定义的，都属于这种类型，即只要监听的属性改变了，都会触发定义好的回调函数，这类watch的expression是我们写的回调函数的字符串形式。 <a name="vi8E3"></a>

#### computed-watcher

我们在组件钩子函数computed中定义的，都属于这种类型，每一个 computed 属性，最后都会生成一个对应的 watcher 对象，但是这类 watcher 有个特点：当计算属性依赖于其他数据时，属性并不会立即重新计算，只有之后其他地方需要读取属性的时候，它才会真正计算，即具备 lazy（懒计算）特性。这类watch的expression是计算属性中的属性名。 <a name="5U8BO"></a>

#### render-watcher

每一个组件都会有一个 render-watcher, 当 data/computed 中的属性改变的时候，会调用该 render-watcher 来更新组件的视图。这类watch的expression是` function () {vm._update(vm._render(), hydrating);}`。

除了功能上的区别，这三种 watcher 也有固定的**执行顺序**，分别是：`computed-render -> normal-watcher -> render-watcher`。
这样安排是有原因的，这样就能尽可能的保证，在更新组件视图的时候，computed 属性已经是最新值了，如果 render-watcher 排在 computed-render 前面，就会导致页面更新的时候 computed 值为旧数据。

![](..\\..\\..\assets\dvck2s\4efdc14a9b6b2bd800c313b1146d4420.svg)

```javascript
//lifecycle.js
new Watcher(vm, updateComponent, noop, {
  before () {
    if (vm._isMounted && !vm._isDestroyed) {
      callHook(vm, 'beforeUpdate')
    }
  }
}, true /* isRenderWatcher */)
```

```javascript
/* @flow */

import {
  warn,
  remove,
  isObject,
  parsePath,
  _Set as Set,
  handleError,
  noop
} from '../util/index'

import { traverse } from './traverse'
import { queueWatcher } from './scheduler'
import Dep, { pushTarget, popTarget } from './dep'

import type { SimpleSet } from '../util/index'

let uid = 0

/**
 * A watcher parses an expression, collects dependencies,
 * and fires callback when the expression value changes.
 * This is used for both the $watch() api and directives.
 */

/*
 *  一个解析表达式，进行依赖收集的观察者，
 *  同时在表达式数据变更时触发回调函数。
 *  它被用于$watch() api以及指令
 */

// 当是渲染watcher时，expOrFn是updateComponent，即重新渲染执行render（_update）
// 当是计算watcher时，expOrFn是计算属性的计算方法
// 当是侦听器watcher时，expOrFn是watch属性的名字，this.cb就是watch的handler属性

// 对于渲染watcher和计算watcher来说，expOrFn的值是一个函数，可以直接设置getter
// 对于侦听器watcher来说，expOrFn是watch属性的名字，会使用parsePath函数解析路径，获取组件上该属性的值（运行getter）

/*
* 需要特别注意的是，在一个组件中，也就是一个vue实例内，
* 只有一个render watcher
* 也就是说一个组件内的变量改变导致dom更新，触发的都是同一个render watcher
* computed的watcher，则会在this._computedWatchers中
* */
export default class Watcher {
  vm: Component;
  expression: string;
  cb: Function;
  id: number;
  deep: boolean;
  user: boolean;
  lazy: boolean;
  sync: boolean;
  dirty: boolean;
  active: boolean;
  deps: Array<Dep>;
  newDeps: Array<Dep>;
  depIds: SimpleSet;
  newDepIds: SimpleSet;
  before: ?Function;
  getter: Function;
  value: any;

  constructor (
    vm: Component,
    expOrFn: string | Function,
    cb: Function,
    options?: ?Object,
    isRenderWatcher?: boolean
  ) {
    this.vm = vm
    if (isRenderWatcher) {
      vm._watcher = this
    }
    /*_watchers存放订阅者实例*/
    vm._watchers.push(this)
    // options
    if (options) {
      this.deep = !!options.deep //是否启用深度监听
      this.user = !!options.user //主要用于错误处理，侦听器 watcher的 user为true，其他基本为false
      this.lazy = !!options.lazy //惰性求值，当属于计算属性watcher时为true
      this.sync = !!options.sync //标记为同步计算，三大类型暂无
      this.before = options.before
    } else {
      this.deep = this.user = this.lazy = this.sync = false
    }
    this.cb = cb
    this.id = ++uid // uid for batching
    this.active = true
    this.dirty = this.lazy // for lazy watchers
    this.deps = []
    this.newDeps = []
    this.depIds = new Set()
    this.newDepIds = new Set()
    this.expression = process.env.NODE_ENV !== 'production'
      ? expOrFn.toString()
      : ''
    // parse expression for getter
    // 解析expOrFn，赋值给this.getter
    if (typeof expOrFn === 'function') {
      this.getter = expOrFn
    } else {
      /* 把表达式expOrFn解析成getter */
      this.getter = parsePath(expOrFn)
      if (!this.getter) {
        this.getter = noop
        process.env.NODE_ENV !== 'production' && warn(
          `Failed watching path: "${expOrFn}" ` +
          'Watcher only accepts simple dot-delimited paths. ' +
          'For full control, use a function instead.',
          vm
        )
      }
    }
    this.value = this.lazy ?
      undefined :
      this.get()
    // 当为computed属性时，lazy为true，
    // 这时不会在new Watcher()执行get函数，即为惰性求值
    // 其他的时候都会在实例化时执行get，进行依赖收集

  }

  /**
   * Evaluate the getter, and re-collect dependencies.
   */
  get() {
    pushTarget(this) // 将当前watcher实例放入Dep.target中
    let value
    const vm = this.vm
    try {
      value = this.getter.call(vm, vm)
      // 执行getter函数即是获取了一次observer中的值
      // 触发了observer的get中的dep.depend()
      // 将Dep.target的watcher实例push到dep.subs中
    } catch (e) {
      if (this.user) {
        // user为true，那即是侦听器watch
        // 如果是watch的值不存在或有问题
        handleError(e, vm, `getter for watcher "${this.expression}"`)
      } else {
        throw e
      }
    } finally {
      // "touch" every property so they are all tracked as
      // dependencies for deep watching
      /*如果存在deep，则触发每个深层对象的依赖，追踪其变化*/
      if (this.deep) {
        // 深度监听，就是watch的deep选项
        /*递归每一个对象或者数组，
         *触发它们的getter，
         *使得对象或数组的每一个成员都被依赖收集，
         *形成一个“深（deep）”依赖关系
        */
        traverse(value)
      }
      popTarget()
      this.cleanupDeps()
    }
    return value
  }

  /**
   * Add a dependency to this directive.
   */
  /*添加一个依赖关系到Deps集合中*/
  addDep(dep: Dep) {
    const id = dep.id
    if (!this.newDepIds.has(id)) {
      // 同时实例中也会保存添加过当前实例的dep和dep的id
      this.newDepIds.add(id)
      this.newDeps.push(dep)
      if (!this.depIds.has(id)) {
        dep.addSub(this)
      }
    }
  }

  /**
   * Clean up for dependency collection.
   */
  /*清理依赖收集*/
  cleanupDeps() {
    let i = this.deps.length
    while (i--) {
      const dep = this.deps[i]
      if (!this.newDepIds.has(dep.id)) {
        dep.removeSub(this)
      }
    }
    let tmp = this.depIds
    this.depIds = this.newDepIds
    this.newDepIds = tmp
    this.newDepIds.clear()
    tmp = this.deps
    this.deps = this.newDeps
    this.newDeps = tmp
    this.newDeps.length = 0
  }

  /**
   * Subscriber interface.
   * Will be called when a dependency changes.
   */
  /*
      调度者接口，
      当依赖发生改变的时候进行回调。
   */
  update() {
    /* istanbul ignore else */
    if (this.lazy) {
      // 脏值标记
      this.dirty = true
    } else if (this.sync) {
      /*同步则执行run直接渲染视图*/
      this.run()
    } else {
      /*异步推送到观察者队列中，由调度者调用。*/
      queueWatcher(this)
    }
  }

  /**
   * Scheduler job interface.
   * Will be called by the scheduler.
   */
  /*
      调度者工作接口，将被调度者回调。
  */
  run() {
    if (this.active) {
      const value = this.get()
      if (
        value !== this.value ||
        // Deep watchers and watchers on Object/Arrays should fire even
        // when the value is the same, because the value may
        // have mutated.
        // 即便值相同，拥有Deep属性的观察者以及在对象／数组上的观察者应该被触发更新，
        // 因为它们的值可能发生改变。
        isObject(value) ||
        this.deep
      ) {
        // set new value
        const oldValue = this.value
        /*设置新的值*/
        this.value = value
        if (this.user) {
          // user 为真时，使用了侦听者watch
          try {
            this.cb.call(this.vm, value, oldValue)
          } catch (e) {
            handleError(e, this.vm, `callback for watcher "${this.expression}"`)
          }
        } else {
          /*触发回调渲染视图*/
          this.cb.call(this.vm, value, oldValue)
        }
      }
    }
  }

  /**
   * Evaluate the value of the watcher.
   * This only gets called for lazy watchers.
   */
  /*获取观察者的值
   * 只会被用于惰性求值的观察者
  * */
  evaluate() {
    this.value = this.get()
    // 用于标记脏数据的变量
    this.dirty = false
  }

  /**
   * Depend on all deps collected by this watcher.
   */
  /*收集该watcher的所有deps依赖*/
  depend() {
    let i = this.deps.length
    while (i--) {
      this.deps[i].depend()
    }
  }

  /**
   * Remove self from all dependencies' subscriber list.
   */
  /* 将自身从所有依赖收集订阅列表删除 */
  teardown() {
    if (this.active) {
      // remove self from vm's watcher list
      // this is a somewhat expensive operation so we skip it
      // if the vm is being destroyed.
      // 将self从vm的watcher列表中移除，
      // 这是一个有点耗时的操作，
      // 所以如果vm正在被销毁，我们就跳过它。
      // 从watcher列表移除自身
      if (!this.vm._isBeingDestroyed) {
        remove(this.vm._watchers, this)
      }
      let i = this.deps.length
      while (i--) {
        this.deps[i].removeSub(this)
      }
      this.active = false
    }
  }
}

```
