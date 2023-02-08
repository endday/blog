---
title: 修改组件的prop的默认值和默认slot
url: https://www.yuque.com/endday/blog/wty6sx
---

在使用element-ui的table组件时，想全局配置一个空数据状态，重复写十分蛋疼。
试了下用函数式组件，算是假装实现了一波继承。

在后面开发的过程中，发现想改个prop的默认值，不想每次用组件的时候都写一次去改，于是弄出下面的改进版，针对props也可以做修改，这样继承就更完善了

<a name="iB937"></a>

#### 函数式组件，extends

```javascript
import { Table } from 'element-ui'

const emptyComp = (h, text) => {
    const emptyTip = text || '暂无数据'
    return h('div', {
        slot: 'empty'
    }, [
        h('div', null, emptyTip)
    ])
}

const myTable = {
    props: {
        stripe: {
            type: Boolean,
            default: true
        }
    },
    extends: Table 
    // 继承element-ui的table
    // 覆盖了table默认prop，根据extends的描述，配置将会合并
}

export default {
    functional: true,
    render: function (h, context) {
        const childNode = [...context.children, emptyComp(h, context.props.emptyTxt)]
        return h(myTable, context.data, childNode)
    }
}
```

<a name="cCJ5U"></a>

#### 函数式组件，修改attrs

```javascript
import { Dialog } from 'element-ui'

// 修改attrs，用assign来赋默认值
// 注意：在 2.3.0 之前的版本中，如果一个函数式组件想要接收 prop，则 props 选项是必须的。
// 在 2.3.0 或以上的版本中，你可以省略 props 选项，所有组件上的 attribute 都会被自动隐式解析为 prop。
// 这就是为什么我直接放在attrs里，也可以当props用的原因

export default {
  functional: true,
  render (h, context) {
    context.data.attrs = Object.assign({
      closeOnClickModal: false,
      closeOnPressEscape: false
    }, context.data.attrs)
    return h(
      Dialog,
      context.data,
      context.children
    )
  }
}

```

<a name="qr0MI"></a>

#### 普通组件，extends

普通的render模式仍然会产生一层template，要用ref的方式去调用组件内部方法就比较尴尬了

```javascript
import { Table } from 'element-ui'

const emptyComp = (h, text) => {
    const emptyTip = text || '暂无数据'
    return h('div', {
        slot: 'empty'
    }, [
        h('div', null, emptyTip)
    ])
}

const myTable = {
    props: {
        stripe: {
            type: Boolean,
            default: true
        }
    },
    extends: Table 
    // 继承element-ui的table
    // 覆盖了table默认prop，根据extends的描述，配置将会合并
}

export default {
    render: function (h) {
        const slots = Object.keys(this.$slots)
            .reduce((arr, key) => arr.concat(this.$slots[key]), [])
            .map(vnode => {
                vnode.context = this._self
                return vnode
            })
        // 展开$slots为数组，并插入自己需要的插槽
        slots.push(emptyComp(h, this.emptyTxt))
        return h(myTable, {
            listeners: this.$listeners,
            props: this.$props,
            scopedSlots: this.$scopedSlots,
            attrs: this.$attrs
        }, slots)
    }
}
```

函数式组件针对hooks的优雅写法，还没想到应用场景

```javascript
export default {
    functional: true,
    render (h, { parent, children }) {
        if (parent._isMounted) {
            return children
        } else {
            parent.$once('hooks:mounted', () => {
                parent.$forceUpdate()
            })
        }
    }
}
```
