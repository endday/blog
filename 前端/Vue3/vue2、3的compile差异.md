---
title: vue2、3的compile差异
url: https://www.yuque.com/endday/blog/wg7hm3
---

参考及抄袭自下列文章
[compile——优化静态内容.md](#Wcjlz)
[Vue中优化器为什么需要标记静态根节点？](https://segmentfault.com/q/1010000019024226/a-1020000019998944)
[Vue3 模板编译原理](https://zhuanlan.zhihu.com/p/181505806) <a name="rPJII"></a>

## vue2 compiler![image.png](..\\..\assets\wg7hm3\1629125892814-684715b4-e938-4091-83aa-5b152f7cafa9.png)

模板编译分为三个阶段：生成ast、优化静态内容、生成render <a name="OAk3T"></a>

### ast

```javascript
// flow/compiler.js

declare type ASTNode = ASTElement | ASTText | ASTExpression;

declare type ASTElement = {
  type: 1;
  tag: string;
  attrsList: Array<ASTAttr>;
  attrsMap: { [key: string]: any };
  rawAttrsMap: { [key: string]: ASTAttr };
  parent: ASTElement | void;
  children: Array<ASTNode>;
  
  start?: number;
  end?: number;

  processed?: true;

  static?: boolean;
  staticRoot?: boolean;
  staticInFor?: boolean;
  staticProcessed?: boolean;
  hasBindings?: boolean;

  text?: string;
  attrs?: Array<ASTAttr>;
  dynamicAttrs?: Array<ASTAttr>;
  props?: Array<ASTAttr>;
  plain?: boolean;
  pre?: true;
  ns?: string;

  // ...省略
};

declare type ASTExpression = {
  type: 2;
  expression: string;
  text: string;
  tokens: Array<string | Object>;
  static?: boolean;
  // 2.4 ssr optimization
  ssrOptimizability?: number;
  start?: number;
  end?: number;
};

declare type ASTText = {
  type: 3;
  text: string;
  static?: boolean;
  isComment?: boolean;
  // 2.4 ssr optimization
  ssrOptimizability?: number;
  start?: number;
  end?: number;
};
```

<a name="Y2YLP"></a>

### optimize

```javascript
const genStaticKeysCached = cached(genStaticKeys)
export function optimize (root: ?ASTElement, options: CompilerOptions) {
  if (!root) return
  isStaticKey = genStaticKeysCached(options.staticKeys || '')
  isPlatformReservedTag = options.isReservedTag || no
  // first pass: mark all non-static nodes.
  markStatic(root)
  // second pass: mark static roots.
  markStaticRoots(root, false)
}
```

```javascript
// 标记所有的静态和非静态结点

function markStatic (node: ASTNode) {
  node.static = isStatic(node)
  if (node.type === 1) {
    // do not make component slot content static. this avoids
    // 1. components not able to mutate slot nodes
    // 2. static slot content fails for hot-reloading
    if (
      !isPlatformReservedTag(node.tag) && // 是指node.tag不是保留标签，即我们自定义的标签时返回true
      node.tag !== 'slot' &&
      node.attrsMap['inline-template'] == null // 是指node不是一个内联模板容器
    ) {
      return
    }
    for (let i = 0, l = node.children.length; i < l; i++) {
      const child = node.children[i]
      markStatic(child) // 递归标记子节点
      if (!child.static) {
        node.static = false // 如果子节点有不是静态的，当前结点node.static = false
      }
    }
  }
}
```

```javascript
function isStatic (node: ASTNode): boolean {
  if (node.type === 2) { // expression 表达式
    return false
  }
  if (node.type === 3) { // text 静态文本
    return true
  }
  return !!(node.pre || ( // 元素上有v-pre指令，结点的子内容是不做编译的，返回true
    !node.hasBindings && // no dynamic bindings  结点没有动态属性，即没有任何指令、数据绑定、事件绑定等
    !node.if && !node.for && // not v-if or v-for or v-else
    !isBuiltInTag(node.tag) && // not a built-in 不是内置的标签，内置的标签有slot和component
    isPlatformReservedTag(node.tag) && // not a component 是平台保留标签，即HTML或SVG标签
    !isDirectChildOfTemplateFor(node) && // 不是template标签的直接子元素且没有包含在for循环中
    Object.keys(node).every(isStaticKey)
  ))
}
```

- 无动态绑定
- 没有 v-if 和 v-for 指令
- 不是内置的标签
- 是平台保留标签(html和svg标签)
- 不是 template 标签的直接子元素并且没有包含在 for 循环中
- 结点包含的属性只能有isStaticKey中指定的几个

```javascript
function markStaticRoots (node: ASTNode, isInFor: boolean) { // 标示ast是否在for循环中
  if (node.type === 1) {
  	// 如果node.static为true，则会添加node.staticInFor
    if (node.static || node.once) {
      node.staticInFor = isInFor
    }
    // For a node to qualify as a static root, it should have children that
    // are not just static text. Otherwise the cost of hoisting out will
    // outweigh the benefits and it's better off to just always render it fresh.
    if (node.static && node.children.length && !(
      node.children.length === 1 &&
      node.children[0].type === 3
    )) {
      node.staticRoot = true
      return
    } else {
      node.staticRoot = false
    }
    if (node.children) {
      for (let i = 0, l = node.children.length; i < l; i++) {
        markStaticRoots(node.children[i], isInFor || !!node.for)
      }
    }
    if (node.ifConditions) {
      walkThroughConditionsBlocks(node.ifConditions, isInFor)
    }
  }
}
```

一个节点要成为静态根节点，需要满足以下条件：

- 自身为静态节点，并且有子节点
- 子节点不能仅为一个文本节点

为什么要markStaticRoots，markStatic不够吗

```javascript
export function renderStatic (
  index: number,
  isInFor: boolean
): VNode | Array<VNode> {
  const cached = this._staticTrees || (this._staticTrees = [])
  let tree = cached[index]
  // if has already-rendered static tree and not inside v-for,
  // we can reuse the same tree.
  if (tree && !isInFor) {
    return tree
  }
  // otherwise, render a fresh tree.
  tree = cached[index] = this.$options.staticRenderFns[index].call(
    this._renderProxy,
    null,
    this // for render fns generated for functional component templates
  )
  markStatic(tree, `__static__${index}`, false)
  return tree
}
```

关于这部分的优化是当一个节点staticRoots为true，并且不在v-for中，那么第一次render的时候会对以这个节点为根的子树进行缓存，等到下次再render的时候直接从缓存中拿，避免再次render。所以标记静态根节点是为了缓存一棵子树用的。 <a name="LtF14"></a>

## vue3 compiler

![image.png](..\\..\assets\wg7hm3\1629125964165-e7d55f9b-ff9e-426a-8130-46f28c53529d.png) <a name="IpZV7"></a>

### ast

baseParse处理后

```javascript
export interface RootNode extends Node {
  type: NodeTypes.ROOT
  children: TemplateChildNode[]
  helpers: symbol[]
  components: string[]
  directives: string[]
  hoists: (JSChildNode | null)[]
  imports: ImportItem[]
  cached: number
  temps: number
  ssrHelpers?: symbol[]
  codegenNode?: TemplateChildNode | JSChildNode | BlockStatement

  // v2 compat only
  filters?: string[]
}

{
  cached: 0,
  children: [{…}],
  codegenNode: undefined,
  components: [],
  directives: [],
  helpers: [],
  hoists: [],
  imports: [],
  loc: {start: {…}, end: {…}, source: "<div><div>hi vue3</div><div>{{msg}}</div></div>"},
  temps: 0,
  type: 0
}
```

- children 中存放的就是最外层 div 的后代。
- loc 则用来描述这个 AST Element 在整个字符串（template）中的位置信息。
- type 则是用于描述这个元素的类型（例如 5 为插值、2 为文本）等等。

可以看到的是不同于「Vue2.x」的 AST，这里我们多了诸如 helpers、codegenNode、hoists 等属性。而，这些属性会在 transform 阶段进行相应地赋值，进而帮助 generate 阶段生成更优的可执行代码。

相比之下，「Vue2.x」的编译阶段没有完整的 transform，只是 optimize 优化了一下 AST

<a name="VaC6B"></a>

### transform

```javascript
export function transform(root: RootNode, options: TransformOptions) {
  const context = createTransformContext(root, options)
  traverseNode(root, context)
  if (options.hoistStatic) {
    hoistStatic(root, context)
  }
  if (!options.ssr) {
    createRootCodegen(root, context)
  }
  // finalize meta information
  root.helpers = [...context.helpers.keys()]
  root.components = [...context.components]
  root.directives = [...context.directives]
  root.imports = context.imports
  root.hoists = context.hoists
  root.temps = context.temps
  root.cached = context.cached

  if (__COMPAT__) {
    root.filters = [...context.filters!]
  }
}
```

<a name="AJVTT"></a>

#### PatchFlags

```javascript
export const enum PatchFlags {
  // 动态文本节点
  TEXT = 1,

  // 动态 class
  CLASS = 1 << 1, // 2

  // 动态 style
  STYLE = 1 << 2, // 4

  // 动态属性，但不包含类名和样式
  // 如果是组件，则可以包含类名和样式
  PROPS = 1 << 3, // 8

  // 具有动态 key 属性，当 key 改变时，需要进行完整的 diff 比较。
  FULL_PROPS = 1 << 4, // 16

  // 带有监听事件的节点
  HYDRATE_EVENTS = 1 << 5, // 32

  // 一个不会改变子节点顺序的 fragment
  STABLE_FRAGMENT = 1 << 6, // 64

  // 带有 key 属性的 fragment 或部分子字节有 key
  KEYED_FRAGMENT = 1 << 7, // 128

  // 子节点没有 key 的 fragment
  UNKEYED_FRAGMENT = 1 << 8, // 256

  // 一个节点只会进行非 props 比较
  NEED_PATCH = 1 << 9, // 512

  // 动态 slot
  DYNAMIC_SLOTS = 1 << 10, // 1024

  // 静态节点
  HOISTED = -1,

  // 指示在 diff 过程应该要退出优化模式
  BAIL = -2
}
```

<a name="sjYb6"></a>

### generate

`generate`是 compile 阶段的最后一步，它的作用是将 `transform`转换后的 AST 生成对应的**可执行代码**，从而在之后 Runtime 的 `Render`阶段时，就可以通过可执行代码生成对应的 `VNode Tree`，然后最终映射为真实的 DOM Tree 在页面上。

```javascript
export function generate(
  ast: RootNode,
  options: CodegenOptions & {
    onContextCreated?: (context: CodegenContext) => void
  } = {}
): CodegenResult {
  const context = createCodegenContext(ast, options)
  if (options.onContextCreated) options.onContextCreated(context)
  const {
    mode,
    push,
    prefixIdentifiers,
    indent,
    deindent,
    newline,
    scopeId,
    ssr
  } = context

	// ........太长，删了

  if (isSetupInlined) {
    push(`(${signature}) => {`)
  } else {
    push(`function ${functionName}(${signature}) {`)
  }
  indent()

  // ........太长，删了

  return {
    ast,
    code: context.code,
    preamble: isSetupInlined ? preambleContext.code : ``,
    // SourceMapGenerator does have toJSON() method but it's not in the types
    map: context.map ? (context.map as any).toJSON() : undefined
  }
}
```

执行 genHoists(ast.hoists, context)，将 transform 生成的静态节点数组 hoists 作为第一个参数

```javascript
// genHoists(ast.hoists, context)

hoists.forEach((exp, i) => {
    if (exp) {
        push(`const _hoisted_${i + 1} = `);
        genNode(exp, context);
        newline();
    }
})
```

```javascript
const _hoisted_1 = { name: "test" }
const _hoisted_2 = /*#__PURE__*/_createTextVNode(" 一个文本节点 ")
const _hoisted_3 = /*#__PURE__*/_createVNode("div", null, "good job!", -1 /* HOISTED */)
```

<a name="hOEFB"></a>

### render function

<a name="NU3C0"></a>

#### 静态提升 hoistStatic

- Vue2中的虚拟dom是进行全量的对比，即数据更新后在虚拟DOM中每个标签内容都会对比有没有发生变化
- Vue3新增了静态标记(PatchFlag)
  - 在创建虚拟DOM的时候会根据DOM中的内容会不会发生变化添加静态标记
  - 数据更新后，只对比带有patch flag的节点

![image.png](..\\..\assets\wg7hm3\1628180027213-8f0979db-0afd-4689-ab63-dacf1fc5f7c8.png)
<https://vue-next-template-explorer.netlify.app/>

```html
<div id="foo" class="bar">
	{{ text }}
</div>
```

```javascript
// 没有静态提升
export function render(_ctx, _cache, $props, $setup, $data, $options) {
  return (_openBlock(), _createElementBlock("div", {
    id: "foo",
    class: "bar"
  }, _toDisplayString(_ctx.text), 1 /* TEXT */))
}

// props属性进行静态提升
const _hoisted_1 = {
  id: "foo",
  class: "bar"
}

export function render(_ctx, _cache, $props, $setup, $data, $options) {
  return (_openBlock(), _createElementBlock("div", _hoisted_1, _toDisplayString(_ctx.text), 1 /* TEXT */))
}
```

```html
<div>
  <div>
    <span class="foo"></span>
    <span class="foo"></span>
  </div>
</div>
```

```javascript
// 没有静态提升的情况
export function render(_ctx, _cache, $props, $setup, $data, $options) {
  return (_openBlock(), _createElementBlock("div", null, [
    _createElementVNode("div", null, [
      _createElementVNode("span", { class: "foo" }),
      _createElementVNode("span", { class: "foo" })
    ])
  ]))
}

// 静态提升
const _hoisted_1 = /*#__PURE__*/_createElementVNode("div", null, [
  /*#__PURE__*/_createElementVNode("span", { class: "foo" }),
  /*#__PURE__*/_createElementVNode("span", { class: "foo" })
], -1 /* HOISTED */)
const _hoisted_2 = [
  _hoisted_1
]

export function render(_ctx, _cache, $props, $setup, $data, $options) {
  return (_openBlock(), _createElementBlock("div", null, _hoisted_2))
}
```

```html
<div>
  <div>
    <span class="foo"></span>
    <span class="foo"></span>
    <span class="foo"></span>
    <span class="foo"></span>
    <span class="foo"></span>
  </div>
</div>
```

```javascript
// StaticVNode 试了下，四个节点不会触发
const _hoisted_1 = /*#__PURE__*/_createStaticVNode("<div><span class=\"foo\"></span><span class=\"foo\"></span><span class=\"foo\"></span><span class=\"foo\"></span><span class=\"foo\"></span></div>", 1)
const _hoisted_2 = [
  _hoisted_1
]

export function render(_ctx, _cache, $props, $setup, $data, $options) {
  return (_openBlock(), _createElementBlock("div", null, _hoisted_2))
}
```

<a name="RQFGw"></a>

#### 事件监听缓存 cacheHandlers

```html
<div>
  <button id="btn" @click="onClick">按钮</button>
</div>
```

```javascript
// 关闭事件监听缓存
export function render(_ctx, _cache, $props, $setup, $data, $options) {
  return (_openBlock(), _createElementBlock("div", null, [
    _createElementVNode("button", {
      id: "btn",
      onClick: _ctx.onClick
    }, "按钮", 8 /* PROPS */, ["onClick"])
  ]))
}

// 开启事件监听缓存

export function render(_ctx, _cache, $props, $setup, $data, $options) {
  return (_openBlock(), _createElementBlock("div", null, [
    _createElementVNode("button", {
      id: "btn",
      onClick: _cache[0] || (_cache[0] = (...args) => (_ctx.onClick && _ctx.onClick(...args)))
    }, "按钮")
  ]))
}
```

vue3通过语法分析，可得知
1、id是个静态的prop，可以跳过，onClick是动态props，所以后面有标记`["onClick"]`
2、不开启事件监听缓存的情况，onClick会被视为props，所以会有标记，`8 /* PROPS */, ["onClick"]`
开启事件监听缓存，onClick会被缓存到\_cache中
