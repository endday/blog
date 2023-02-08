---
title: Vue3笔记-Vue 3 Deep Dive with Evan You
url: https://www.yuque.com/endday/blog/kuyy6u
---

<a name="Jpr3g"></a>

### Vue 的三个核心模块

<a name="bl0vk"></a>

#### Reactivity Module 响应式模块

响应式模块允许我们创建 JavaScript 响应对象并可以观察其变化。当使用这些对象的代码运行时，它们会被跟踪，因此，它们可以在响应对象发生变化后运行。 <a name="ExKhN"></a>

#### Compiler Module 编译器模块

编译器模块获取 HTML 模板并将它们编译成渲染函数。这可能在运行时在浏览器中发生，但在构建 Vue 项目时更常见。这样浏览器就可以只接收渲染函数。 <a name="p33eM"></a>

#### Renderer Module 渲染模块

渲染模块的代码包含在网页上渲染组件的三个不同阶段：

- 渲染阶段
- 挂载阶段
- 补丁阶段

在渲染阶段，将调用 render 函数，它返回一个虚拟 DOM 节点。
在挂载阶段，使用虚拟DOM节点并调用 DOM API 来创建网页。
在补丁阶段，渲染器将旧的虚拟节点和新的虚拟节点进行比较并只更新网页变化的部分。
