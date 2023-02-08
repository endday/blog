---
title: 如何用ts构建工具库
url: https://www.yuque.com/endday/blog/wo8ruy
---

<a name="TAxX8"></a>

# 依赖

<a name="sFvd3"></a>

#### typescript

<a name="AWkN2"></a>

#### tslib

这是一个TypeScript的运行库，包含所有TypeScript助手函数。
此库主要由TypeScript中的--importelpers标志使用。使用--importelpers时，在以下发出的文件中使用\_\_extends和\_assign等辅助函数的模块：

```javascript
var __assign = (this && this.__assign) || Object.assign || function(t) {
  for (var s, i = 1, n = arguments.length; i < n; i++) {
    s = arguments[i];
    for (var p in s) if (Object.prototype.hasOwnProperty.call(s, p))
      t[p] = s[p];
  }
  return t;
};
exports.x = {};
exports.y = __assign({}, exports.x);
```

<a name="sV2dv"></a>

#### ts-node

ts-node 也是基于 node 的，在 node 执行的 hook 里自动进行了 ts->js 的语言编译，使得 ts 可以被直接执行
