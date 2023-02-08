---
title: observer/index.js
url: https://www.yuque.com/endday/blog/rc69xo
---

![](..\\..\\..\assets\rc69xo\2609b17c85757a3cb0bdff58b9f9f51d.svg)

```javascript
/* @flow */

import Dep from './dep'
import VNode from '../vdom/vnode'
import { arrayMethods } from './array'
import {
  def,
  hasOwn,
  hasProto,
  isObject,
  isPlainObject,
  isPrimitive,
  isServerRendering,
  isUndef,
  isValidArrayIndex,
  warn
} from '../util/index'

const arrayKeys = Object.getOwnPropertyNames(arrayMethods)

/**
 * In some cases we may want to disable observation inside a component's
 * update computation.
 */
export let shouldObserve: boolean = true

export function toggleObserving (value: boolean) {
  shouldObserve = value
}

/*
* 针对Object使用defineReactive，
* 数组使用的是劫持原型方法
* */

/**
 * Observer class that is attached to each observed
 * object. Once attached, the observer converts the target
 * object's property keys into getter/setters that
 * collect dependencies and dispatch updates.
 */

/*
 * 每个被观察到对象被附加上观察者实例，
 * 一旦被添加，观察者将为目标对象加上getter\setter属性，
 * 进行依赖收集以及调度更新。
*/
export class Observer {
  value: any
  dep: Dep
  vmCount: number // number of vms that have this object as root $data

  constructor (value: any) {
    this.value = value
    this.dep = new Dep()
    this.vmCount = 0
    /*
    * 将Observer实例绑定到data的__ob__属性上面去，
    * 之前说过observe的时候会先检测是否已经有__ob__对象存放Observer实例了，
    * def方法定义可以参考
    * https://github.com/vuejs/vue/blob/dev/src/core/util/lang.js#L16
    * def方法第四个参数为是否可以枚举
    * 这里的__ob__这样声明，是不可枚举的
    * 这里写法导致的结果： this.a.__ob__.value === this.a
    */
    // def (obj: Object, key: string, val: any, enumerable?: boolean)
    def(value, '__ob__', this)
    if (Array.isArray(value)) {
      // '__proto__' in {}
      // ie9、ie10 和 opera 没有 __proto__ 属性
      if (hasProto) {
        /*直接覆盖原型的方法来修改目标对象*/
        protoAugment(value, arrayMethods)
      } else {
        /*定义（覆盖）目标对象或数组的某一个方法*/
        copyAugment(value, arrayMethods, arrayKeys)
      }
      /*如果是数组则需要遍历数组的每一个成员进行observe*/
      this.observeArray(value)
    } else {
      /*如果是对象则直接walk进行绑定*/
      this.walk(value)
    }
  }

  /**
   * Walk through all properties and convert them into
   * getter/setters. This method should only be called when
   * value type is Object.
   */

  /*
   * 遍历每一个对象并且在它们上面绑定getter与setter。
   * 这个方法只有在value的类型是对象的时候才能被调用
   */
  walk (obj: Object) {
    const keys = Object.keys(obj)
    for (let i = 0; i < keys.length; i++) {
      defineReactive(obj, keys[i])
    }
  }

  /**
   * Observe a list of Array items.
   */

  /*
   * 对一个数组的每一个成员进行observe
   */
  observeArray (items: Array<any>) {
    for (let i = 0, l = items.length; i < l; i++) {
      observe(items[i])
    }
  }
}

// helpers

/**
 * Augment a target Object or Array by intercepting
 * the prototype chain using __proto__
 */

/**
 * 直接覆盖原型的方法来修改目标对象
 */
function protoAugment (target, src: Object) {
  /* eslint-disable no-proto */
  target.__proto__ = src
  /* eslint-enable no-proto */
}

/**
 * Augment a target Object or Array by defining
 * hidden properties.
 */

/**
 * 定义（覆盖）目标对象或数组的某一个方法
 */

/* istanbul ignore next */
function copyAugment (target: Object, src: Object, keys: Array<string>) {
  for (let i = 0, l = keys.length; i < l; i++) {
    const key = keys[i]
    def(target, key, src[key])
  }
}

/**
 * Attempt to create an observer instance for a value,
 * returns the new observer if successfully observed,
 * or the existing observer if the value already has one.
 */

/*
 * 尝试创建一个Observer实例（__ob__），
 * 如果成功创建Observer实例则返回新的Observer实例，
 * 如果已有Observer实例则返回现有的Observer实例。
 */
export function observe (value: any, asRootData: ?boolean): Observer | void {
  if (!isObject(value) || value instanceof VNode) {
    return
  }
  let ob: Observer | void
  /*
  * 这里用__ob__这个属性来判断是否已经有Observer实例，
  * 如果没有Observer实例则会新建一个Observer实例并赋值给__ob__这个属性，
  * 如果已有Observer实例则直接返回该Observer实例
  */
  if (hasOwn(value, '__ob__') && value.__ob__ instanceof Observer) {
    ob = value.__ob__
  } else if (
    /*
     * 这里的判断是为了确保value是单纯的对象，而不是函数或者是Regexp等情况。
     * 而且该对象在shouldConvert的时候才会进行Observer。这是一个标识位，避免重复对value进行Observer
     */
    shouldObserve &&
    !isServerRendering() &&
    (Array.isArray(value) || isPlainObject(value)) &&
    Object.isExtensible(value) &&
    !value._isVue
    /*
      这里的判断是为了确保value是单纯的对象，而不是函数或者是Regexp等情况。
      而且该对象在shouldConvert的时候才会进行Observer。
      这是一个标识位，避免重复对value进行Observer
    */
  ) {
    ob = new Observer(value)
  }
  if (asRootData && ob) {
    /*如果是根数据则计数，后面Observer中的observe的asRootData非true*/
    ob.vmCount++
  }
  return ob
}

/**
 * Define a reactive property on an Object.
 */

/*为对象defineProperty上在变化时通知的属性*/
export function defineReactive (
  obj: Object,  //对象
  key: string,  //对象的key
  val: any,     //监听的数据 返回的数据
  customSetter?: ?Function, //  日志函数
  shallow?: boolean //是否要添加__ob__ 属性
) {
  /*在闭包中定义一个dep对象*/
  const dep = new Dep()

  const property = Object.getOwnPropertyDescriptor(obj, key)
  // configurable属性为true，对象的属性描述可以被改变或者属性可被删除
  if (property && property.configurable === false) {
    return
  }
  /*
   * 如果之前该对象已经预设了getter以及setter函数则将其取出来，
   * 新定义的getter/setter中会将其执行，
   * 保证不会覆盖之前已经定义的getter/setter。
   */
  // cater for pre-defined getter/setters
  const getter = property && property.get
  const setter = property && property.set
  // 只有在walk函数中才是args长度为2，args长度为2，即没有传入val
  // 当没有设置getter或者已有setter
  // TODO: 这个条件存疑
  if ((!getter || setter) && arguments.length === 2) {
    val = obj[key]
  }
  /*对象的子对象递归进行observe并返回子节点的Observer对象*/
  let childOb = !shallow && observe(val)
  Object.defineProperty(obj, key, {
    enumerable: true,
    configurable: true,
    get: function reactiveGetter () {
      /*如果原本对象拥有getter方法则执行*/
      const value = getter ? getter.call(obj) : val
      if (Dep.target) {
        /*进行依赖收集*/
        dep.depend()
        if (childOb) {
          /**
           * 子级dep也要收集依赖，这个就是为array准备的
           * 最开始分析，array数据的dep本身是不会收集依赖的
           * 但是可以让他强行进行一次收集，就是在这儿
           * */
          childOb.dep.depend()
          if (Array.isArray(value)) {
            /**
            是数组则需要对每一个成员都进行依赖收集，
            如果数组的成员还是数组，则递归。
            */
            dependArray(value)
          }
        }
      }
      return value
    },
    set: function reactiveSetter (newVal) {
      /*通过getter方法获取当前值，与新值进行比较，一致则不需要执行下面的操作*/
      const value = getter ? getter.call(obj) : val
      /* eslint-disable no-self-compare */
      if (newVal === value || (newVal !== newVal && value !== value)) {
        return
      }
      /* eslint-enable no-self-compare */
      if (process.env.NODE_ENV !== 'production' && customSetter) {
        customSetter()
      }
      // #7981: for accessor properties without setter
      if (getter && !setter) return
      if (setter) {
        /*如果原本对象拥有setter方法则执行setter*/
        setter.call(obj, newVal)
      } else {
        val = newVal
      }
      /*新的值需要重新进行observe，保证数据响应式*/
      childOb = !shallow && observe(newVal)
      /*dep对象通知所有的观察者*/
      dep.notify()
    }
  })
}

/**
 * Set a property on an object. Adds the new property and
 * triggers change notification if the property doesn't
 * already exist.
 */
// Vue.set = set
export function set (target: Array<any> | Object, key: any, val: any): any {
  if (process.env.NODE_ENV !== 'production' &&
    (isUndef(target) || isPrimitive(target))
  ) {
    warn(`Cannot set reactive property on undefined, null, or primitive value: ${(target: any)}`)
  }
  /*如果传入数组则在指定位置插入val*/
  if (Array.isArray(target) && isValidArrayIndex(key)) {
    target.length = Math.max(target.length, key)
    target.splice(key, 1, val)
    /*数组已经被改写了Array原型上的方法来进行响应式处理，这里直接调用就行*/
    return val
  }
  /*如果是一个对象，并且已经存在了这个key则直接返回*/
  if (key in target && !(key in Object.prototype)) {
    target[key] = val
    return val
  }
  /*获得target的Oberver实例*/
  const ob = (target: any).__ob__
  /*
    _isVue 一个防止vm实例自身被观察的标志位 ，_isVue为true则代表vm实例，也就是this
    vmCount判断是否为根节点，
    存在则代表是data的根节点，
    Vue 不允许在已经创建的实例上动态添加新的根级响应式属性(root-level reactive property)
  */
  if (target._isVue || (ob && ob.vmCount)) {
    process.env.NODE_ENV !== 'production' && warn(
      'Avoid adding reactive properties to a Vue instance or its root $data ' +
      'at runtime - declare it upfront in the data option.'
    )
    return val
  }
  if (!ob) {
    /*
     * 用__ob__这个属性来判断是否已经有Observer实例
     * 如果连需要赋值的对象都不是响应式的，那就直接赋值
     */
    target[key] = val
    return val
  }
  /*
   *注意：set方法是有返回值的
   */
  defineReactive(ob.value, key, val)
  ob.dep.notify()
  return val
}

/**
 * Delete a property and trigger change if necessary.
 */
// Vue.delete = del
export function del (target: Array<any> | Object, key: any) {
  if (process.env.NODE_ENV !== 'production' &&
    (isUndef(target) || isPrimitive(target))
  ) {
    warn(`Cannot delete reactive property on undefined, null, or primitive value: ${(target: any)}`)
  }
  if (Array.isArray(target) && isValidArrayIndex(key)) {
    target.splice(key, 1)
    return
  }
  const ob = (target: any).__ob__
  if (target._isVue || (ob && ob.vmCount)) {
    /*通过vmCount判断，提示你删掉的属性可能是根对象*/
    process.env.NODE_ENV !== 'production' && warn(
      'Avoid deleting properties on a Vue instance or its root $data ' +
      '- just set it to null.'
    )
    return
  }
  if (!hasOwn(target, key)) {
    return
  }
  delete target[key]
  if (!ob) {
    return
  }
  ob.dep.notify()
}

/**
 * Collect dependencies on array elements when the array is touched, since
 * we cannot intercept array element access like property getters.
 */

/*
  递归数组，进行依赖收集
*/
function dependArray (value: Array<any>) {
  for (let e, i = 0, l = value.length; i < l; i++) {
    e = value[i]
    e && e.__ob__ && e.__ob__.dep.depend()
    if (Array.isArray(e)) {
      dependArray(e)
    }
  }
}

```
