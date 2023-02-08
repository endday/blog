---
title: vue2迁移到vue3的一些修改点
url: https://www.yuque.com/endday/blog/is92z2
---

<a name="h9uqZ"></a>

# 过渡的 class 名更改

过渡类名v-enter修改为v-enter-from、过渡类名v-leave修改为v-leave-from <a name="LqME3"></a>

# v-model

- **非兼容**：用于自定义组件时，v-modelprop 和事件默认名称已更改：
  - prop：value->modelValue；
  - event：input->update:modelValue；
- **非兼容**：v-bind的.sync修饰符和组件的model选项已移除，可用v-model作为代替；
- **新增**：现在可以在同一个组件上使用多个v-model进行双向绑定；
- **新增**：现在可以自定义v-model修饰符。
