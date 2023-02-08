---
title: share/util.js
url: https://www.yuque.com/endday/blog/ukh7fm
---

<a name="LhiVr"></a>

### 重点关注的函数

<a name="cGYwD"></a>

#### cached

缓存函数 <a name="v1kaC"></a>

#### looseEqual

判断两个值是否全等 <a name="7M1Tk"></a>

#### once

确保该函数只调用一次

<a name="Ms4Kh"></a>

### 补充说明

<a name="yl9nG"></a>

#### makeMap

这个函数是用来将字符串查找变成对象取值，
虽然增加一个步骤，但是缩减了代码
name1,name2
\[name1,name2]
{name1: true, name2: true}
字符代码少，数组性能差，对象代码多，性能好
这个函数就是平衡性能和代码量的产物

```javascript
/* @flow */

// Object.freeze()阻止修改现有属性的特性和值，并阻止添加新属性。
export const emptyObject = Object.freeze({})

// These helpers produce better VM code in JS engines due to their
// explicitness and function inlining.
//判断数据 是否是undefined或者null
export function isUndef (v: any): boolean %checks {
  return v === undefined || v === null
}
//判断数据 是否不等于 undefined或者null
export function isDef (v: any): boolean %checks {
  return v !== undefined && v !== null
}
//判断是否真的等于true
export function isTrue (v: any): boolean %checks {
  return v === true
}
//  判断是否是false
export function isFalse (v: any): boolean %checks {
  return v === false
}

/**
 * Check if value is primitive.
 * 判断数据类型是否是string，number，symbol，boolean
 */
export function isPrimitive (value: any): boolean %checks {
  return (
    typeof value === 'string' ||
    typeof value === 'number' ||
    // $flow-disable-line
    typeof value === 'symbol' ||
    typeof value === 'boolean'
  )
}

/**
 * Quick object check - this is primarily used to tell
 * Objects from primitive values when we know the value
 * is a JSON-compliant type.
 * 快速对象检查-当我们知道值是符合JSON的类型时，
 * 它主要用于区分对象和原始值
 */
//判断是否是对象
export function isObject (obj: mixed): boolean %checks {
  return obj !== null && typeof obj === 'object'
}

/**
 * Get the raw type string of a value, e.g., [object Object].
 */
//获取toString 简写
const _toString = Object.prototype.toString
//类型判断 返会Array ，Function，String,Object,Re 等
export function toRawType (value: any): string {
  return _toString.call(value).slice(8, -1)
}

/**
 * Strict object type check. Only returns true
 * for plain JavaScript objects.
 */
//判断是否是对象
export function isPlainObject (obj: any): boolean {
  return _toString.call(obj) === '[object Object]'
}
//判断是否是正则对象
export function isRegExp (v: any): boolean {
  return _toString.call(v) === '[object RegExp]'
}

/**
 * Check if val is a valid array index.
 * 检查数值是否是有效的数组索引
 */
export function isValidArrayIndex (val: any): boolean {
  // Math.floor 向下取整
  // isFinite 如果 number 是有限数字（或可转换为有限数字），那么返回 true。
  // 否则，如果 number 是 NaN（非数字），或者是正、负无穷大的数，则返回 false。
  // 也就是说n要是大于0的整数，不为正负无穷大
  const n = parseFloat(String(val))
  return n >= 0 && Math.floor(n) === n && isFinite(val)
}
// 判断是否为promise对象
export function isPromise (val: any): boolean {
  return (
    isDef(val) &&
    typeof val.then === 'function' &&
    typeof val.catch === 'function'
  )
}

/**
 * Convert a value to a string that is actually rendered.
 * 将对象或者其他基本数据 变成一个 字符串
 */
/*
value
将要序列化成 一个 JSON 字符串的值。
replacer 可选
如果该参数是一个函数，则在序列化过程中，被序列化的值的每个属性都会经过该函数的转换和处理；
如果该参数是一个数组，则只有包含在这个数组中的属性名才会被序列化到最终的 JSON 字符串中；
如果该参数为 null 或者未提供，则对象所有的属性都会被序列化；
space 可选
指定缩进用的空白字符串，用于美化输出（pretty-print）；
如果参数是个数字，它代表有多少的空格；上限为10。该值若小于1，则意味着没有空格；
如果该参数为字符串（当字符串长度超过10个字母，取其前10个字母），该字符串将被作为空格；
如果该参数没有提供（或者为 null），将没有空格。
*/
export function toString (val: any): string {
  return val == null
    ? ''
    : Array.isArray(val) || (isPlainObject(val) && val.toString === _toString)
      ? JSON.stringify(val, null, 2)
      : String(val)
}

/**
 * Convert an input value to a number for persistence.
 * If the conversion fails, return original string.
 * 字符串转数字，如果失败则返回字符串
 */
export function toNumber (val: string): number | string {
  const n = parseFloat(val)
  return isNaN(n) ? val : n
}

/**
 * Make a map and return a function for checking if a key
 * is in that map.
 * map 对象中的[name1,name2]
 * 变成这样的map{name1:true,name2:true}，
 * 传入一个key值就能判断是否该key值是否存在
 */
export function makeMap (
  str: string,
  expectsLowerCase?: boolean
): (key: string) => true | void {
  const map = Object.create(null)
  const list: Array<string> = str.split(',')
  for (let i = 0; i < list.length; i++) {
    map[list[i]] = true
  }
  return expectsLowerCase
    ? val => map[val.toLowerCase()]
    : val => map[val]
}

/**
 * Check if a tag is a built-in tag.
 * 检查标记是否为内置标记。
 */
export const isBuiltInTag = makeMap('slot,component', true)

/**
 * Check if an attribute is a reserved attribute.
 * 检查属性是否为保留属性。
 */
export const isReservedAttribute = makeMap('key,ref,slot,slot-scope,is')

/**
 * Remove an item from an array.
 * 删除数组
 */
export function remove (arr: Array<any>, item: any): Array<any> | void {
  if (arr.length) {
    const index = arr.indexOf(item)
    if (index > -1) {
      return arr.splice(index, 1)
    }
  }
}

/**
 * Check whether an object has the property.
 * 检查对象属性是否是实例化还是原型上面的
 */
const hasOwnProperty = Object.prototype.hasOwnProperty
export function hasOwn (obj: Object | Array<*>, key: string): boolean {
  return hasOwnProperty.call(obj, key)
}

/**
 * Create a cached version of a pure function.
 * 创建纯函数的缓存版本。
 * 创建一个函数，缓存到闭包里，再返回柯里化函数
 * 这个函数的作用就是当传入第二次传入同一个值的时候，
 * 直接返回已经缓存的函数结果
 */
export function cached<F: Function> (fn: F): F {
  const cache = Object.create(null)
  return (function cachedFn (str: string) {
    const hit = cache[str]
    return hit || (cache[str] = fn(str))
  }: any)
}

/**
 * Camelize a hyphen-delimited string.
 * 用连字符分隔的字符串变为驼峰式
 */
const camelizeRE = /-(\w)/g
export const camelize = cached((str: string): string => {
  return str.replace(camelizeRE, (_, c) => c ? c.toUpperCase() : '')
})

/**
 * Capitalize a string.
 * 将首字母变成大写
 */
export const capitalize = cached((str: string): string => {
  return str.charAt(0).toUpperCase() + str.slice(1)
})

/**
 * Hyphenate a camelCase string.
 * \B是非单词分界符，即可以查出是否包含某个字，
 * 如“ABCDEFGHIJK”中是否包含“BCDEFGHIJK”这个字。
 */
const hyphenateRE = /\B([A-Z])/g
export const hyphenate = cached((str: string): string => {
  //大写字母，加完减号又转成小写了 比如把驼峰 aBc 变成了 a-bc
  //匹配大写字母并且两面不是空白的 替换成 '-' + '字母' 再全部转换成小写
  return str.replace(hyphenateRE, '-$1').toLowerCase()
})

/**
 * Simple bind polyfill for environments that do not support it,
 * e.g., PhantomJS 1.x. Technically, we don't need this anymore
 * since native bind is now performant enough in most browsers.
 * But removing it would mean breaking code that was able to run in
 * PhantomJS 1.x, so this must be kept for backward compatibility.
 * 简单绑定polyfill适用于不支持它的环境，例如PhantomJS 1.x。
 * 从技术上讲，我们不再需要它了，因为本机绑定现在在大多数浏览器中已经有足够的性能了。
 * 但是删除它将意味着破坏能够在PhantomJS 1.x中运行的代码，
 * 因此必须保留这些代码以实现向后兼容。
 */

/* istanbul ignore next */
function polyfillBind (fn: Function, ctx: Object): Function {
  function boundFn (a) {
    const l = arguments.length
    return l
      ? l > 1
        ? fn.apply(ctx, arguments)
        : fn.call(ctx, a)
      : fn.call(ctx)
  }

  boundFn._length = fn.length
  return boundFn
}

function nativeBind (fn: Function, ctx: Object): Function {
  return fn.bind(ctx)
}

export const bind = Function.prototype.bind
  ? nativeBind
  : polyfillBind

/**
 * Convert an Array-like object to a real Array.
 * 将类数组转换成真的数组
 */
export function toArray (list: any, start?: number): Array<any> {
  start = start || 0
  let i = list.length - start
  const ret: Array<any> = new Array(i)
  while (i--) {
    ret[i] = list[i + start]
  }
  return ret
}

/**
 * Mix properties into target object.
 * 浅拷贝
 * 对象浅拷贝，参数（to, _from）循环_from的值，会覆盖掉to的值
 */
export function extend (to: Object, _from: ?Object): Object {
  for (const key in _from) {
    to[key] = _from[key]
  }
  return to
}

/**
 * Merge an Array of Objects into a single Object.
 * 合并对象数组合并成一个对象
 */
export function toObject (arr: Array<any>): Object {
  const res = {}
  for (let i = 0; i < arr.length; i++) {
    if (arr[i]) {
      extend(res, arr[i])
    }
  }
  return res
}

/* eslint-disable no-unused-vars */

/**
 * Perform no operation.
 * Stubbing args to make Flow happy without leaving useless transpiled code
 * with ...rest (https://flow.org/blog/2017/05/07/Strict-Function-Call-Arity/).
 * 不执行任何操作。在不留下无用的透明代码和…rest的情况下，存根参数使flow愉快
 */
export function noop (a?: any, b?: any, c?: any) {}

/**
 * Always return false.
 * 返回假
 */
export const no = (a?: any, b?: any, c?: any) => false

/* eslint-enable no-unused-vars */

/**
 * Return the same value.
 * 返回相同值
 */
export const identity = (_: any) => _

/**
 * Generate a string containing static keys from compiler modules.
 * [{ staticKeys:1},{staticKeys:2},{staticKeys:3}]
 * 连接数组对象中的 staticKeys key值，连接成一个字符串 str=‘1,2,3’
 */
export function genStaticKeys (modules: Array<ModuleOptions>): string {
  return modules.reduce((keys, m) => {
    //累加staticKeys的值变成数组
    return keys.concat(m.staticKeys || [])
  }, []).join(',')//转换成字符串
}

/**
 * Check if two values are loosely equal - that is,
 * if they are plain objects, do they have the same shape?
 * 检测a和b的数据类型，是否是不是数组或者对象，
 * 对象的key长度一样即可，数组长度一样即可
 */
export function looseEqual (a: any, b: any): boolean {
  //如果a和b是完全相等 则true
  if (a === b) return true
  const isObjectA = isObject(a)
  const isObjectB = isObject(b)
  if (isObjectA && isObjectB) { //如果a和都是对象则让下走
    try {
      const isArrayA = Array.isArray(a)
      const isArrayB = Array.isArray(b)
      if (isArrayA && isArrayB) { //如果a和b都是数组
        return a.length === b.length && a.every((e, i) => { //如果a长度和b长度一样的时候
          return looseEqual(e, b[i]) //递归
        })
      } else if (a instanceof Date && b instanceof Date) { //或者a和b是日期类型
        return a.getTime() === b.getTime() //获取毫秒数来判断是否相等
      } else if (!isArrayA && !isArrayB) { //或者a和b都不是数组
        const keysA = Object.keys(a) // 获取到a的key值 变成一个数组
        const keysB = Object.keys(b) // 获取到b的key值 变成一个数组
        return keysA.length === keysB.length && keysA.every(key => { //他们的对象key值长度是一样的时候  则加载every 条件函数
          return looseEqual(a[key], b[key])  //递归
        })
      } else {
        /* istanbul ignore next */
        return false
      }
    } catch (e) {
      /* istanbul ignore next */
      return false
    }
  } else if (!isObjectA && !isObjectB) { //b和a 都不是对象的时候
    return String(a) === String(b) //把a和b变成字符串，判断他们是否相同
  } else {
    return false
  }
}

/**
 * Return the first index at which a loosely equal value can be
 * found in the array (if value is a plain object, the array must
 * contain an object of the same shape), or -1 if it is not present.
 * 返回第一个索引，在该索引处可以在数组中找到大致相等的值
 * 返回第一个索引，在该索引处可以在数组中找到大致相等的值
 * （如果值是普通对象，则数组必须包含全等的对象），
 * 如果不存在则返回-1。
 */
export function looseIndexOf (arr: Array<mixed>, val: mixed): number {
  for (let i = 0; i < arr.length; i++) {
    if (looseEqual(arr[i], val)) return i
  }
  return -1
}

/**
 * Ensure a function is called only once.
 * 确保该函数只调用一次 闭包函数
 */
export function once (fn: Function): Function {
  let called = false
  return function () {
    if (!called) {
      called = true
      fn.apply(this, arguments)
    }
  }
}

```
