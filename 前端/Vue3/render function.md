---
title: render function
url: https://www.yuque.com/endday/blog/hhgw44
---

<a name="Uf0sa"></a>

### Vue 2 API

```javascript
function render(h) {
  return h('div', {
    attrs: {
      id: foo
    },
    on: {
      click: this.onClick
    }
  }, 'hello')
}
```

- 第一个参数是类型，所以我们在这里创建一个 div。
- 第二个参数是一个对象包含 vnode 上的所有数据或属性，API有点冗长从某种意义上说，你必须指明传递给节点的绑定类型。例如，如果要绑定属性你必须把它嵌套在attrs对象下如果要绑定事件侦听器你得把它列在 on 下面。
- 第三个参数是这个 vnode 的子节点。所以直接传递一个字符串是一个方便的 API，表明此节点只包含文本子节点，但它也可以是包含更多子节点的数组。所以你可以在这里有一个数组并且嵌套了更多的嵌套 h 调用。 <a name="hzbDZ"></a>

### Vue3 API

```javascript
function render(h) {
  return h('div', {
    id: 'foo',
    onClick: this.onClick
  }, 'hello')
}
```

第一个显著的变化是我们现在有了一个扁平的 props 结构。当你调用 h 时，第二个参数现在总是一个扁平的对象。你可以直接给它传递一个属性，这里我们只是给它一个 ID。按惯例监听器以 on 开头，所以任何带 on 的都会自动绑定为一个监听器所以你不必考虑太多嵌套的问题。
在大多数情况下，你也不需要思考是应将其作为 attribute 绑定还是DOM属性绑定，因为 Vue 将智能地找出为你做这件事的最好方法。我们检查这个 key 是否作为属性存在在原生 DOM 中。如果存在，我们会将其设置为 property，如果它不存在，我们将它设置为一个attribute。

render API 的另一项改动是 h helper 现在是直接从 Vue 本身全局导入的。一些用户在 Vue 2 中因为 h 在这里传递而在这里面 h 又很特别，因为它绑定到当前组件实例。当你想拆分一个大的渲染函数时，你必须把这个 h 函数一路传递给这些分割函数。所以，这有点困难，但有了全局引入的 h 你导入一次就可以分割你的渲染函数，在同一个文件里分割多少个都行。

渲染函数不再有 h 参数了，在内部它确实接收参数，但这只是编译器使用的用来生成代码。当用户直接使用时，他们不需要这个参数。所以，如果你用 TypeScript 使用定义的组件 API 你也会得到 this 的完整类型推断。
